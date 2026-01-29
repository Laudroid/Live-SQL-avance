# Exemples pratiques d'utilisation des vues et vues matérialisées en contexte ERP et finance

Les systèmes ERP (Enterprise Resource Planning) et les applications financières manipulent de vastes volumes de données et requièrent des traitements complexes et rapides. L’usage des **vues** et **vues matérialisées** dans PostgreSQL offre des solutions efficaces pour structurer les données, améliorer les performances, et garantir la sécurité. Cet article illustre ces cas d’usage avec des exemples directement applicables.

---

## 1. Vues dans un contexte ERP

### 1.1. Agrégation des données de stock

Dans un ERP, il est fréquent de consulter l’état des stocks par produit et entrepôt. Une vue simplifie l’accès aux données agrégées :

```sql
CREATE VIEW stock_summary AS
SELECT product_id, warehouse_id, SUM(quantity) AS total_quantity
FROM stock_movements
GROUP BY product_id, warehouse_id;
```

Cette vue facilite les rapports d’inventaire sans avoir à réécrire cette jointure et agrégation complexe.

### 1.2. Restriction d’accès sécurisée

Les données sensibles (ex : coûts d’achats) peuvent être masquées via des vues. Par exemple, une vue pour le service commercial retire les coûts :

```sql
CREATE VIEW sales_data_public AS
SELECT order_id, customer_id, product_id, quantity, sale_price
FROM sales_orders;
```

Les droits d’accès sur cette vue sont attribués uniquement au personnel commercial, alors que les accès à la table complète sont restreints.

---

## 2. Vues matérialisées pour les rapports financiers

### 2.1. Performance sur l’agrégation des mouvements comptables

Les rapports financiers nécessitent souvent des calculs lourds sur des millions d’enregistrements : balances, cumuls par compte, périodes fiscales, etc.

Une vue matérialisée permet de stocker ces résultats pour des requêtes rapides :

```sql
CREATE MATERIALIZED VIEW financial_balances AS
SELECT account_id, fiscal_period, SUM(debit) AS total_debit, SUM(credit) AS total_credit
FROM accounting_entries
GROUP BY account_id, fiscal_period;
```

Le traitement différé par `REFRESH MATERIALIZED VIEW` optimise les ressources, tout en assurant un accès rapide pour les audits ou analyses.

### 2.2. Indexation pour accélérer les filtres

Après création de la vue matérialisée, créer un index améliore encore la vitesse des requêtes filtrant par compte ou période :

```sql
CREATE INDEX idx_fin_bal_account_period ON financial_balances(account_id, fiscal_period);
```

---

## 3. Exemple avancé combiné (ERP + Finance)  

### Scénario : tableau de bord consolidé

Un tableau de bord requiert des indicateurs combinant ventes, stocks et données financières pour piloter l’activité.

Une vue matérialisée peut agréger ces multiples données :

```sql
CREATE MATERIALIZED VIEW dashboard_data AS
SELECT
  s.product_id,
  s.total_quantity,
  sf.total_sales,
  fb.total_debit,
  fb.total_credit
FROM
  stock_summary s
JOIN
  sales_summary sf ON s.product_id = sf.product_id
LEFT JOIN
  financial_balances fb ON s.product_id = fb.account_id -- Hypothèse de correspondance produit-compte
WHERE
  fb.fiscal_period = '2024Q1';
```

Cette structure permet des lectures rapides du tableau de bord, tout en déléguant la charge de calcul au rafraîchissement planifié.

---

## 4. Diagramme Mermaid : architecture vue / vue matérialisée

```mermaid
graph LR
    A[Tables sources ERP & Finance] --> B[Vues simples]
    B --> C[Vues matérialisées agrégées]
    C --> D[Indexation]
    D --> E[Requêtes rapides Tableau de bord]
    E --> F[Utilisateurs métier]
```

---

## 5. Bonnes pratiques

- **Choisir judicieusement entre vues simples et matérialisées** selon fréquence de mise à jour des données et exigence de rapidité.  
- **Indexer les vues matérialisées** sur les colonnes utilisées dans les filtres et jointures.  
- **Planifier les refreshs** des vues matérialisées en heures creuses.  
- **Restreindre l’accès** via des vues dédiées pour la sécurité des données.  

---

## 6. Sources et références

- [PostgreSQL Documentation - Materialized Views](https://www.postgresql.org/docs/current/sql-creatematerializedview.html)  
- [EnterpriseDB Blog - Practical uses of materialized views in ERP](https://www.enterprisedb.com/blog/practical-uses-materialized-views-postgresql)  
- [Cybertec PostgreSQL - Performance tips for Materialized Views](https://www.cybertec-postgresql.com/en/postgresql-materialized-views-explained/)  
- [PG Day Europe - Postgres in Financial Services](https://2019.pgday.eu/postgres-in-financial-services)  

---

## Conclusion

Dans les environnements ERP et finance, les vues classiques favorisent la simplicité et la sécurité, tandis que les vues matérialisées apportent un gain significatif de performance sur les calculs lourds et répétés. Bien intégrés, ces outils aident à construire des systèmes robustes, réactifs, et adaptés aux contraintes métiers complexes.