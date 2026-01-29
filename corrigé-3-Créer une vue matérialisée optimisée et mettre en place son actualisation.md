Voici deux solutions détaillées pour le TP sur les vues matérialisées et leur actualisation concurrente, structurées en chapitres pour une meilleure compréhension.

---

### **Chapitre 1 : Mise en Place, Création et Actualisation Concurrente de la Vue Matérialisée**

Ce chapitre couvre les étapes fondamentales de la création de l'environnement, de la définition de la vue matérialisée, de son optimisation par indexation, et de la démonstration de l'actualisation concurrente.

#### **1. Mise en Place de l'Environnement et des Données**


```sql
-- 1.1. Création de la base de données (à exécuter depuis une base comme 'postgres')
-- CREATE DATABASE ecom_reporting;

-- 1.2. Connexion à la nouvelle base de données (ex: \c ecom_reporting; ou via votre client SQL)

-- 1.3. Création des tables
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    category VARCHAR(50) NOT NULL
);

CREATE TABLE regions (
    region_id SERIAL PRIMARY KEY,
    region_name VARCHAR(50) NOT NULL
);

CREATE TABLE sales (
    sale_id BIGSERIAL PRIMARY KEY,
    product_id INT NOT NULL REFERENCES products(product_id),
    region_id INT NOT NULL REFERENCES regions(region_id),
    sale_date DATE NOT NULL,
    quantity INT NOT NULL CHECK (quantity > 0),
    price_per_unit DECIMAL(10, 2) NOT NULL CHECK (price_per_unit > 0),
    total_amount DECIMAL(10, 2) GENERATED ALWAYS AS (quantity * price_per_unit) STORED
);

-- 1.4. Index pour accélérer les jointures et les filtres sur les tables sources
CREATE INDEX idx_sales_product_id ON sales (product_id);
CREATE INDEX idx_sales_region_id ON sales (region_id);
CREATE INDEX idx_sales_sale_date ON sales (sale_date);

-- 1.5. Peuplement des données d'exemple (pour les tables de référence)
INSERT INTO products (product_name, category) VALUES
('Laptop Pro', 'Électronique'),
('T-Shirt Coton', 'Vêtements'),
('SQL Guide', 'Livres'),
('Café Bio', 'Alimentaire'),
('Smartphone X', 'Électronique'),
('Jeans Slim', 'Vêtements');

INSERT INTO regions (region_name) VALUES
('Nord'),
('Sud'),
('Est'),
('Ouest'),
('Centre');

-- Pour la table 'sales', il est recommandé d'utiliser un script de génération de données
-- pour insérer des dizaines ou centaines de milliers de lignes afin de simuler un volume réaliste.
-- Exemple pour quelques lignes (à étendre massivement pour le TP) :
INSERT INTO sales (product_id, region_id, sale_date, quantity, price_per_unit) VALUES
(1, 1, '2023-01-15', 2, 1200.00),
(3, 2, '2023-01-20', 1, 45.00),
(2, 1, '2023-02-01', 3, 25.00),
(4, 3, '2023-02-10', 5, 8.00),
(1, 4, '2023-03-05', 1, 1300.00),
(5, 5, '2023-03-12', 1, 800.00),
(6, 2, '2023-04-01', 2, 30.00),
(1, 1, '2024-01-01', 1, 1250.00), -- Vente récente
(3, 3, '2024-01-05', 2, 48.00);  -- Vente récente
```


#### **2. Création de la Vue Matérialisée pour le Reporting**


```sql
-- 2.1. Requête SELECT de base pour le rapport mensuel
SELECT
    TO_CHAR(s.sale_date, 'YYYY-MM') AS reporting_month,
    p.category,
    r.region_name,
    COUNT(s.sale_id) AS total_sales_count,
    SUM(s.quantity) AS total_quantity_sold,
    SUM(s.total_amount) AS total_revenue
FROM
    sales s
JOIN
    products p ON s.product_id = p.product_id
JOIN
    regions r ON s.region_id = r.region_id
GROUP BY
    reporting_month, p.category, r.region_name
ORDER BY
    reporting_month, p.category, r.region_name;

-- 2.2. Création de la vue matérialisée
CREATE MATERIALIZED VIEW mv_sales_summary_monthly AS
SELECT
    TO_CHAR(s.sale_date, 'YYYY-MM') AS reporting_month,
    p.category,
    r.region_name,
    COUNT(s.sale_id) AS total_sales_count,
    SUM(s.quantity) AS total_quantity_sold,
    SUM(s.total_amount) AS total_revenue
FROM
    sales s
JOIN
    products p ON s.product_id = p.product_id
JOIN
    regions r ON s.region_id = r.region_id
GROUP BY
    reporting_month, p.category, r.region_name
ORDER BY
    reporting_month, p.category, r.region_name;

-- 2.3. Vérification de la vue
SELECT * FROM mv_sales_summary_monthly LIMIT 10;
```


**Réflexion :**
*   **Vue matérialisée vs. vue simple :** Une vue matérialisée stocke physiquement les résultats de la requête sous-jacente, ce qui la rend beaucoup plus rapide à interroger qu'une vue simple qui exécute la requête à chaque appel. Pour le reporting agrégé sur de grands volumes de données, où les données ne changent pas en temps réel mais périodiquement, la vue matérialisée est essentielle pour la performance.
*   **Impact :** La création d'une vue matérialisée consomme de l'espace disque pour stocker les données agrégées et prend un certain temps à être calculée initialement, proportionnellement à la complexité de la requête et au volume de données sources.

#### **3. Optimisation de la Vue Matérialisée**


```sql
-- Création d'un index unique sur les colonnes de groupement pour accélérer les requêtes
-- et permettre l'actualisation CONCURRENTLY.
CREATE UNIQUE INDEX idx_mv_sales_summary_monthly_pk
ON mv_sales_summary_monthly (reporting_month, category, region_name);
```


#### **4. Actualisation Concurrente de la Vue Matérialisée**


```sql
-- 4.1. Simulation de nouvelles ventes
INSERT INTO sales (product_id, region_id, sale_date, quantity, price_per_unit) VALUES
(1, 1, CURRENT_DATE, 5, 12.50), -- Nouvelle vente pour le mois en cours
(2, 3, CURRENT_DATE, 2, 50.00),
(1, 1, CURRENT_DATE - INTERVAL '1 day', 10, 10.00),
(5, 4, CURRENT_DATE, 3, 850.00); -- Une vente d'électronique significative
```


**Test de l'actualisation concurrente :**

*   **Session 1 (Client A) :**
    1.  `BEGIN;`
    2.  `SELECT * FROM mv_sales_summary_monthly WHERE reporting_month = TO_CHAR(CURRENT_DATE, 'YYYY-MM');` (Laissez cette session ouverte, la transaction non validée).
    3.  (Dans une autre fenêtre ou après avoir basculé) `INSERT INTO sales (product_id, region_id, sale_date, quantity, price_per_unit) VALUES (1, 1, CURRENT_DATE, 100, 1.00);`
    4.  `COMMIT;`

*   **Session 2 (Client B) :**
    1.  `REFRESH MATERIALIZED VIEW CONCURRENTLY mv_sales_summary_monthly;`
        *   **Observation :** Cette commande s'exécute sans bloquer la `Session 1`. Les utilisateurs de la `Session 1` peuvent toujours interroger l'ancienne version de la vue.

*   **Retour à la Session 1 (Client A) :**
    1.  Exécutez à nouveau : `SELECT * FROM mv_sales_summary_monthly WHERE reporting_month = TO_CHAR(CURRENT_DATE, 'YYYY-MM');`
        *   **Observation :** Les résultats affichés sont toujours ceux *avant* l'actualisation, car la transaction a commencé avant la fin de l'actualisation.

*   **Une fois la `REFRESH` terminée dans la Session 2 :**
    1.  Dans la Session 1, démarrez une nouvelle transaction ou exécutez la requête hors transaction : `SELECT * FROM mv_sales_summary_monthly WHERE reporting_month = TO_CHAR(CURRENT_DATE, 'YYYY-MM');`
        *   **Observation :** Les résultats reflètent maintenant les nouvelles données, car la vue a été actualisée et la nouvelle version est disponible.

**Réflexion :**
*   **Avantage de `CONCURRENTLY` :** L'avantage principal est la disponibilité continue de la vue matérialisée pendant son actualisation. Les requêtes en cours ou nouvelles peuvent toujours accéder à l'ancienne version des données, évitant ainsi les blocages et les interruptions de service, ce qui est crucial dans un environnement de production.
*   **Conditions préalables pour `CONCURRENTLY` :** La vue matérialisée doit impérativement posséder au moins un index `UNIQUE` sur une ou plusieurs colonnes qui identifient de manière unique chaque ligne. Sans cet index, PostgreSQL ne peut pas effectuer l'actualisation concurrente.
*   **Quand utiliser `REFRESH MATERIALIZED VIEW` (sans `CONCURRENTLY`) :** Cette option est utilisée lorsque la disponibilité de la vue n'est pas critique pendant l'actualisation, par exemple, lors de fenêtres de maintenance, pour des vues très petites, ou si l'application peut tolérer une brève indisponibilité. Elle est généralement plus rapide car elle ne gère pas la complexité de la concurrence.

---

### **Chapitre 2 : Approfondissement - Automatisation, Surveillance et Cas d'Usage Avancés**

Ce chapitre explore des aspects plus avancés de la gestion des vues matérialisées, essentiels pour leur déploiement et leur maintenance en production.

#### **1. Automatisation de l'Actualisation**

L'automatisation de l'actualisation des vues matérialisées est cruciale pour maintenir la fraîcheur des données sans intervention manuelle.

*   **Méthode 1 : `cron` (Linux/Unix)**
    Un job `cron` est la méthode la plus courante. Il exécute une commande à des intervalles définis.
    1.  Créez un script shell (ex: `refresh_mv.sh`) :
        
```bash
        #!/bin/bash
        # Script pour actualiser la vue matérialisée
        # Assurez-vous que les variables d'environnement PGPASSWORD, PGUSER, PGDATABASE, PGHOST sont définies
        # ou utilisez l'option -d, -U, etc. directement avec psql.

        LOG_FILE="/var/log/ecom_reporting/refresh_mv.log"
        DATE_TIME=$(date +"%Y-%m-%d %H:%M:%S")

        echo "[$DATE_TIME] Starting refresh of mv_sales_summary_monthly..." >> "$LOG_FILE"

        psql -h localhost -U votre_utilisateur -d ecom_reporting -c "REFRESH MATERIALIZED VIEW CONCURRENTLY mv_sales_summary_monthly;" >> "$LOG_FILE" 2>&1

        if [ $? -eq 0 ]; then
            echo "[$DATE_TIME] Refresh completed successfully." >> "$LOG_FILE"
        else
            echo "[$DATE_TIME] Refresh failed!" >> "$LOG_FILE"
        fi
        ```

    2.  Rendez le script exécutable : `chmod +x refresh_mv.sh`
    3.  Ajoutez une entrée `cron` (ex: `crontab -e`) pour l'exécuter chaque nuit à 3h du matin :
        
```
        0 3 * * * /chemin/vers/votre/script/refresh_mv.sh
        ```


*   **Méthode 2 : Planificateur de tâches du SGBD (si disponible)**
    Certains SGBD (comme SQL Server Agent) ont des planificateurs de tâches intégrés. PostgreSQL n'en a pas directement, mais des extensions comme `pg_cron` peuvent être installées pour planifier des tâches SQL directement depuis la base de données.
    
```sql
    -- Exemple avec pg_cron (nécessite l'installation de l'extension)
    SELECT cron.schedule(
        'daily-mv-refresh', -- Nom du job
        '0 3 * * *',        -- Chaque jour à 3h du matin
        'REFRESH MATERIALIZED VIEW CONCURRENTLY mv_sales_summary_monthly;'
    );
    ```


#### **2. Surveillance de la Performance**

La surveillance est essentielle pour s'assurer que l'actualisation se déroule comme prévu et pour identifier les goulots d'étranglement.

*   **Temps d'exécution :**
    *   **Logs du script `cron` :** Le script shell ci-dessus redirige la sortie vers un fichier de log, permettant de voir les heures de début et de fin.
    *   **Vues système PostgreSQL :**
        *   `pg_stat_statements` (si activée) peut enregistrer les statistiques d'exécution de toutes les requêtes, y compris `REFRESH MATERIALIZED VIEW`.
        *   `EXPLAIN ANALYZE REFRESH MATERIALIZED VIEW CONCURRENTLY mv_sales_summary_monthly;` peut être exécuté manuellement pour analyser le plan d'exécution et les coûts de l'actualisation.

*   **Requêtes en cours d'exécution :**
    *   La vue système `pg_stat_activity` est l'outil principal pour voir les requêtes actives, leur durée, l'utilisateur, et l'état de la transaction.
    
```sql
    SELECT
        pid,
        application_name,
        datname,
        usename,
        client_addr,
        backend_start,
        query_start,
        state,
        wait_event_type,
        wait_event,
        query
    FROM
        pg_stat_activity
    WHERE
        state = 'active'
        AND query ILIKE '%REFRESH MATERIALIZED VIEW%';
    ```

    Cela permet de vérifier si l'actualisation est en cours, si elle est bloquée, et depuis combien de temps.

#### **3. Cas d'Usage `WITH NO DATA`**

L'option `WITH NO DATA` lors de la création d'une vue matérialisée signifie que la vue est créée mais n'est pas peuplée immédiatement.

*   **Quand l'utiliser :**
    *   **Vues très volumineuses ou complexes :** Si la requête de définition de la vue est extrêmement longue à exécuter, vous pourriez vouloir créer la structure sans données pour des raisons de rapidité de déploiement, puis la peupler ultérieurement pendant une période de faible activité.
    *   **Dépendances circulaires :** Dans des scénarios complexes où des vues matérialisées dépendent les unes des autres de manière circulaire, `WITH NO DATA` permet de briser le cycle en créant d'abord la structure, puis en la peuplant.
    *   **Peuplement par étapes ou personnalisé :** Si vous souhaitez peupler la vue avec des données historiques d'une manière spécifique (par exemple, par lots, ou en filtrant certaines périodes initialement) plutôt que d'exécuter la requête complète d'un coup.

*   **Exemple concret :**
    Imaginez une vue matérialisée qui agrège des données sur plusieurs années, mais vous souhaitez d'abord la créer rapidement pour ensuite la peupler avec un script qui insère les données année par année, ou qui effectue des transformations supplémentaires avant l'insertion finale.

    
```sql
    -- Création de la vue matérialisée sans la peupler immédiatement
    CREATE MATERIALIZED VIEW mv_sales_summary_yearly_nodata AS
    SELECT
        EXTRACT(YEAR FROM s.sale_date) AS reporting_year,
        p.category,
        SUM(s.total_amount) AS total_revenue_yearly
    FROM
        sales s
    JOIN
        products p ON s.product_id = p.product_id
    GROUP BY
        reporting_year, p.category
    ORDER BY
        reporting_year, p.category
    WITH NO DATA;

    -- Plus tard, quand vous êtes prêt, vous l'actualisez pour la peupler
    REFRESH MATERIALIZED VIEW mv_sales_summary_yearly_nodata;
    ```