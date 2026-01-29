# Différences entre Views et Materialized Views : définition et usages  

Les **Views** et **Materialized Views** sont deux concepts clés en bases de données relationnelles permettant de simplifier l’accès aux données et d’optimiser les performances. Comprendre leurs différences fondamentales et leurs usages spécifiques permet de mieux choisir la solution adaptée à chaque besoin.

---

## 1. Définition des Views  

Une **View** est une requête SQL sauvegardée sous forme d’une table virtuelle. Elle ne contient pas de données en propre, mais fournit un résultat dynamique issu des tables sous-jacentes à chaque exécution.

- **Fonctionnement** : À chaque accès, la base exécute la requête définissant la View en temps réel.  
- **Mise à jour** : Les changements dans les tables sources sont immédiatement visibles.  
- **Utilisation** : Simplifie la lecture, masque la complexité, applique des filtres/agrégations récurrentes.  

### Exemple simple de création d’une View

```sql
CREATE VIEW recent_orders AS
SELECT order_id, customer_id, order_date, amount
FROM orders
WHERE order_date >= CURRENT_DATE - INTERVAL '30 days';
```

---

## 2. Définition des Materialized Views  

Une **Materialized View** (vue matérialisée) est une vue dont les résultats sont **stockés physiquement** dans la base de données. C’est un snapshot pré-calculé et matérialisé des données.

- **Fonctionnement** : La requête est exécutée une fois, puis le résultat est enregistré.  
- **Mise à jour** : Nécessite une actualisation explicite (`REFRESH MATERIALIZED VIEW`) pour refléter les changements.  
- **Utilisation** : Améliore la performance des requêtes lourdes ou sur des données volumineuses, utiles en BI ou reporting.  

### Exemple simple de création d’une Materialized View

```sql
CREATE MATERIALIZED VIEW monthly_sales_summary AS
SELECT date_trunc('month', order_date) AS month, SUM(amount) AS total_sales
FROM orders
GROUP BY month;
```

---

## 3. Comparaison synthétique  

| Caractéristique         | View                            | Materialized View                  |
|------------------------|--------------------------------|----------------------------------|
| Données                | Résultats calculés à la volée  | Données stockées physiquement    |
| Mise à jour            | Instantanée avec les tables sources | Manuelle (refresh explicite)     |
| Performances           | Dépend de la complexité de la requête | Optimisée grâce au stockage      |
| Usage                  | Simplification, abstraction    | Optimisation de requêtes lourdes |
| Stockage               | Pas de stockage dédié           | Occupe de l’espace disque        |
| Fraîcheur des données  | Toujours à jour                 | Synchronisation à programmer     |

---

## 4. Cas d’usage typiques  

- **Views**  
  - Masquer la complexité de jointures complexes  
  - Sécurité : limiter l’accès par colonnes ou lignes (Views filtrées)  
  - Requêtes légères avec besoin de données toujours à jour  

- **Materialized Views**  
  - Rapports ou tableaux de bord avec agrégations lourdes  
  - Réduction du temps de réponse sur des données volumineuses  
  - Données issues de sources externes ou peu fréquemment mises à jour  

---

## 5. Diagramme Mermaid : schéma simplifié  

```mermaid
graph TD
    A[Tables Sources] -->|Lecture en temps réel| B[View (table virtuelle)]
    A -->|Calcul & stockage| C[Materialized View (données stockées)]
    C -->|REFRESH MATERIALIZED VIEW| A
```

---

## 6. Notions spécifiques dans PostgreSQL  

- Les Materialized Views supportent l’actualisation partielle et concurrente à partir de PostgreSQL 9.4+.  
- L’actualisation peut se faire en `CONCURRENTLY` afin que la vue reste accessible pendant la mise à jour.  
- Jusqu’à PostgreSQL 12, les Views ne peuvent pas contenir d’index, les Materialized Views oui, ce qui améliore encore leurs performances.  

---

## 7. Sources et documentation  

- [PostgreSQL Documentation - Views](https://www.postgresql.org/docs/current/sql-createview.html)  
- [PostgreSQL Documentation - Materialized Views](https://www.postgresql.org/docs/current/sql-creatematerializedview.html)  
- [Oracle Docs - Views and Materialized Views](https://docs.oracle.com/en/database/oracle/oracle-database/19/dwhsg/materialized-views-and-snapshots.html)  
- [SQLShack - Understanding SQL Views and Materialized Views](https://www.sqlshack.com/understanding-sql-views-and-materialized-views/)  

---

## Conclusion  

Les Views et Materialized Views répondent à des objectifs différents. Tandis que les Views offrent une abstraction dynamique synthétisant des requêtes complexes, les Materialized Views privilégient la performance pour des analyses lourdes en stockant les données pré-calculées. Choisir entre ces deux solutions dépend du compromis entre fraîcheur des données et rapidité d’accès.