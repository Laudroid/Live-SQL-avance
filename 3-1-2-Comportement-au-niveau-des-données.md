# Comportement au niveau des données : Views vs Materialized Views  

Les Views et les Materialized Views diffèrent fondamentalement dans la manière dont elles traitent, stockent et actualisent les données. Cette distinction impacte directement leur usage, leur performance et la fraîcheur des données consultées. Cet article explique précisément ces différences en s’appuyant sur des exemples pratiques et illustre les mécanismes sous-jacents.

---

## 1. Vue dynamique vs vue matérialisée  

### View (vue classique) : données toujours à jour  

- La View est une **table virtuelle** qui ne contient aucune donnée physique.  
- Les données sont **recalculées à chaque requête** à partir des tables sources.  
- Toute modification dans les tables sous-jacentes est immédiatement visible via la View.  
- Exemple :

```sql
CREATE VIEW active_customers AS
SELECT customer_id, name
FROM customers
WHERE status = 'active';
```

Chaque fois qu’on interroge `active_customers`, le système exécute la requête pour retourner les clients actifs à jour.

---

### Materialized View : données statiques jusqu’à rafraîchissement  

- La Materialized View stocke **physiquement le résultat** d’une requête, comme un snapshot.  
- Les données sont **figées** au moment de la création ou du dernier rafraîchissement.  
- Pour mettre à jour ces données, une opération de **REFRESH** est nécessaire :  

```sql
REFRESH MATERIALIZED VIEW monthly_sales_summary;
```

- Entre deux rafraîchissements, les données peuvent être obsolètes par rapport aux tables sources.  

---

## 2. Impact des mises à jour sur les données  

| Aspect                         | View                         | Materialized View                     |
|-------------------------------|------------------------------|-------------------------------------|
| Données visibles instantanément | Oui                          | Non, jusqu’au prochain rafraîchissement |
| Impact des modifications dans les tables source | Immédiat                   | Aucun, si pas rafraîchie             |
| Consommation de ressources     | Exécution complète à chaque requête | Coût lors du rafraîchissement, rapide en lecture |
| Possibilité d’index             | Non (PostgreSQL)             | Oui, permet accélération des requêtes |

---

## 3. Exemple concret : analyse des ventes

### Vue classique

```sql
CREATE VIEW sales_summary AS
SELECT product_id, SUM(quantity) AS total_qty
FROM sales
GROUP BY product_id;
```

Si on ajoute une vente dans `sales`, la View renverra immédiatement le total mis à jour.

---

### Materialized View correspondante

```sql
CREATE MATERIALIZED VIEW sales_summary_mat AS
SELECT product_id, SUM(quantity) AS total_qty
FROM sales
GROUP BY product_id;
```

Les totaux resteront figés jusqu’à l’exécution de

```sql
REFRESH MATERIALIZED VIEW sales_summary_mat;
```

---

## 4. Options spécifiques à PostgreSQL

- Depuis PostgreSQL 9.4, on peut actualiser une Materialized View **concurremment** pour la rendre disponible pendant le rafraîchissement :  

```sql
REFRESH MATERIALIZED VIEW CONCURRENTLY sales_summary_mat;
```

- Cependant, cela requiert une **clé unique** définie dans la Materialized View.  
- Les Views classiques n’ont pas d’index et dépendent entièrement des indexes des tables sous-jacentes.

---

## 5. Diagramme Mermaid : comparaison des flux de données

```mermaid
flowchart LR
    subgraph Tables Sources
        A[Tables Sources] 
    end
    subgraph View
        B[Vue (virtual) - Calcul à la volée]
        A --> B
        B --> C[Requête - données à jour]
    end
    subgraph Materialized View
        D[Materialized View - Données stockées]
        A -- Modifications -->|Ne changent pas immédiatement| D
        D --> E[Lecture rapide]
        F[REFRESH MATERIALIZED VIEW]
        F --> D
    end
```

---

## 6. Sources

- [PostgreSQL Documentation - Views](https://www.postgresql.org/docs/current/sql-createview.html)  
- [PostgreSQL Documentation - Materialized Views](https://www.postgresql.org/docs/current/sql-creatematerializedview.html)  
- [Oracle Documentation - Materialized Views Behavior](https://docs.oracle.com/en/database/oracle/oracle-database/19/dwhsg/materialized-views-and-snapshots.html)  
- [SQLShack - Difference Between Views and Materialized Views](https://www.sqlshack.com/difference-between-views-and-materialized-views/)  

---

## Résumé  

Le choix entre View et Materialized View dépend du besoin en fraîcheur des données versus performance de lecture. Les Views garantissent toujours des données à jour mais au prix d’un calcul à chaque requête. À l’inverse, les Materialized Views offrent des lectures rapides grâce au stockage physique mais nécessitent une actualisation explicite pour refléter les changements des tables sous-jacentes. Connaître ces comportements permet d’optimiser ses architectures de données selon les contraintes métier.