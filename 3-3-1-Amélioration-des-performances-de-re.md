# Amélioration des performances de requêtes lourdes avec les Views et Materialized Views

Les requêtes complexes ou volumineuses peuvent entraîner un temps d'exécution important et solliciter lourdement les ressources d'une base de données. Pour optimiser leur exécution, PostgreSQL propose plusieurs mécanismes, notamment l'utilisation des **Views** (vues classiques) et **Materialized Views** (vues matérialisées).  

Cet article détaille comment ces outils peuvent améliorer les performances des requêtes lourdes avec des exemples d’application et des bonnes pratiques.

---

## 1. Comprendre le contexte des requêtes lourdes

Requêtes lourdes impliquent souvent :  
- Des **jointures multiples** sur de larges tables.  
- Des **agrégations** complexes ou des calculs sur de grandes volumétries.  
- Des filtres dynamiques changeant fréquemment.  

Ces éléments augmentent le temps de calcul à chaque exécution.

---

## 2. Utilisation des Views (vues classiques)  

### Points clés

- Les vues sont des **requêtes stockées**, mais elles sont **calculées à chaque exécution**.  
- Elles simplifient la lecture et la maintenance du code SQL.  

### Limites pour l’optimisation

- Ne réduisent pas le coût de calcul car la requête sous-jacente s’exécute intégralement à chaque appel.  

### Exemple

```sql
CREATE VIEW sales_summary AS
SELECT product_id, SUM(quantity) AS total_qty, SUM(amount) AS total_sales
FROM sales
GROUP BY product_id;
```

Toute requête sur `sales_summary` exécutera ce calcul à la volée.

---

## 3. Utilisation des Materialized Views (vues matérialisées)

### Principes

- Stockent physiquement le résultat de la requête au moment du **rafraîchissement** (`REFRESH MATERIALIZED VIEW`).  
- Lecture très rapide, car les données sont pré-calculées.  
- Les données peuvent devenir **obsolètes** entre deux rafraîchissements.

### Exemple

```sql
CREATE MATERIALIZED VIEW sales_summary_mat AS
SELECT product_id, SUM(quantity) AS total_qty, SUM(amount) AS total_sales
FROM sales
GROUP BY product_id;
```

Pour actualiser :

```sql
REFRESH MATERIALIZED VIEW sales_summary_mat;
```

---

## 4. Scénarios d’amélioration des performances

### A. Requêtes interactives vs traitement batch

| Cas                 | Vue classique       | Vue matérialisée      |
| --------------------|--------------------|----------------------|
| Requêtes adhoc fréquentes | Moins efficace, recalcul à chaque fois | Très efficace, lecture rapide |
| Données très volatiles | Adapté            | Risque d’obsolescence |
| Données stables ou rafraîchies périodiquement | Moins pertinent | Idéal pour analyses batch |

### B. Large volume de données avec agrégation lourde  

- Pré-calculer via materialized views évite de répéter des calculs coûteux.  
- Indexer la vue matérialisée pour accélérer les filtres.

---

## 5. Optimisations complémentaires

- **Actualisation concurrente** (`REFRESH MATERIALIZED VIEW CONCURRENTLY`) pour éviter le blocage lors de la mise à jour de la vue. Nécessite un index unique.  
- Création d’**index** sur les colonnes fréquemment utilisées dans les filtres.  

```sql
CREATE UNIQUE INDEX idx_product_id ON sales_summary_mat(product_id);
```

- Utilisation de partitions pour réduire les volumes à traiter (partitionnement de tables derrière les vues).  

---

## 6. Répartition des charges avec vues et views matérialisées

```mermaid
flowchart TD
    A[Requête lourde utilisateur] -->|Sans optimisation| B[Calcul complet en temps réel]
    A -->|Avec View| C[Calcul complet en temps réel (vue simplifiée)]
    A -->|Avec Materialized View| D[Lecture rapide des résultats pré-calculés]
    D --> E[Possibilité de rafraîchissement périodique]
```

---

## 7. Sources et références

- [PostgreSQL Documentation - Materialized Views](https://www.postgresql.org/docs/current/sql-creatematerializedview.html)  
- [Cybertec PostgreSQL Blog - Materialized Views](https://www.cybertec-postgresql.com/en/postgresql-materialized-views-explained/)  
- [Use The Index, Luke! - Materialized Views](https://use-the-index-luke.com/sql/materialized-views/overview)  
- [AWS Database Blog - Optimize Query Performance with Materialized Views in PostgreSQL](https://aws.amazon.com/fr/blogs/database/optimize-query-performance-with-materialized-views-in-postgresql/)  

---

## Conclusion

Les vues matérialisées constituent une solution efficace pour améliorer la vitesse des requêtes lourdes en évitant le recalcul systématique. Elles sont particulièrement adaptées aux situations où la rapidité de lecture prime sur la fraîcheur absolue des données. En les combinant avec des index bien choisis et des actualisations planifiées, elles fournissent une approche pragmatique pour accélérer les traitements analytiques et opérationnels dans PostgreSQL.