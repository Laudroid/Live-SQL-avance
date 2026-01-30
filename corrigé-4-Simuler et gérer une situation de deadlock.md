Voici deux solutions pour le TP sur la gestion des deadlocks en SQL, présentées sous forme de chapitres distincts pour une meilleure clarté et un approfondissement progressif.

---

### **Chapitre 1 : Simulation et Résolution Fondamentale du Deadlock**

Ce chapitre vous guide à travers la simulation d'un deadlock, son analyse et sa résolution en appliquant la stratégie essentielle de l'ordre de verrouillage cohérent.

#### **Phase 1 : Préparation de l'environnement**


```sql
-- Création de la table Comptes
CREATE TABLE Comptes (
    id_compte INT PRIMARY KEY,
    nom_titulaire VARCHAR(50) NOT NULL,
    solde DECIMAL(10, 2) NOT NULL
);

-- Insertion des données initiales
INSERT INTO Comptes (id_compte, nom_titulaire, solde) VALUES
(1, 'Alice', 1000.00),
(2, 'Bob', 500.00);

-- Vérification de l'état initial
SELECT * FROM Comptes;
```


#### **Phase 2 : Simulation du Deadlock**

Pour cette simulation, nous utiliserons la syntaxe PostgreSQL pour la fonction de pause (`pg_sleep`). Adaptez-la si vous utilisez un autre SGBD :
*   **PostgreSQL :** `SELECT pg_sleep(5);`
*   **SQL Server :** `WAITFOR DELAY '00:00:05';`
*   **MySQL :** `SELECT SLEEP(5);`

**Procédure d'exécution :**
1.  **Session 1 (Transaction 1) :** Exécutez le bloc de code suivant jusqu'à la ligne `SELECT pg_sleep(5);` incluse.
    
```sql
    -- Session 1 (Transaction 1)
    BEGIN TRANSACTION;

    -- Alice envoie 100 à Bob
    -- Étape 1 : Débiter Alice (verrouille la ligne du compte 1)
    UPDATE Comptes
    SET solde = solde - 100.00
    WHERE id_compte = 1;

    -- Simule un délai
    SELECT pg_sleep(5); -- Adaptez cette ligne à votre SGBD
    ```

2.  **Session 2 (Transaction 2) :** Immédiatement après, dans une seconde session, exécutez le bloc de code suivant jusqu'à la ligne `SELECT pg_sleep(5);` incluse.
    
```sql
    -- Session 2 (Transaction 2)
    BEGIN TRANSACTION;

    -- Bob envoie 50 à Alice
    -- Étape 1 : Débiter Bob (verrouille la ligne du compte 2)
    UPDATE Comptes
    SET solde = solde - 50.00
    WHERE id_compte = 2;

    -- Simule un délai
    SELECT pg_sleep(5); -- Adaptez cette ligne à votre SGBD
    ```

3.  **Session 1 (Transaction 1) :** Revenez à la première session et exécutez la suite de T1.
    
```sql
    -- Suite de la Session 1
    -- Étape 2 : Créditer Bob (T1 tente de verrouiller la ligne du compte 2, déjà verrouillée par T2)
    UPDATE Comptes
    SET solde = solde + 100.00
    WHERE id_compte = 2;

    COMMIT;
    ```

4.  **Session 2 (Transaction 2) :** Revenez à la seconde session et exécutez la suite de T2.
    
```sql
    -- Suite de la Session 2
    -- Étape 2 : Créditer Alice (T2 tente de verrouiller la ligne du compte 1, déjà verrouillée par T1)
    UPDATE Comptes
    SET solde = solde + 50.00
    WHERE id_compte = 1;

    COMMIT;
    ```


**Observation :**
L'une des deux transactions (généralement la dernière à tenter d'acquérir le verrou manquant) échouera.
*   **Exemple de message d'erreur (PostgreSQL) :** `ERROR: deadlock detected` ou `SQLSTATE[40P01]: Deadlock detected: 7 ERROR: deadlock detected`
*   Le SGBD détecte le deadlock et choisit une transaction "victime" à annuler (`ROLLBACK`) pour permettre à l'autre de se terminer.

#### **Phase 3 : Analyse et Compréhension**

1.  **Explication du deadlock :**
    Un deadlock s'est produit en raison d'une situation de "circular wait" (attente circulaire).
    *   **Transaction 1 (T1)** a verrouillé le `Compte 1` (pour débiter Alice) et attend de verrouiller le `Compte 2` (pour créditer Bob).
    *   Simultanément, **Transaction 2 (T2)** a verrouillé le `Compte 2` (pour débiter Bob) et attend de verrouiller le `Compte 1` (pour créditer Alice).
    Chaque transaction détient une ressource que l'autre attend, et attend une ressource que l'autre détient. Aucune ne peut avancer, créant un blocage mutuel.

2.  **Réaction du SGBD :**
    Les SGBD modernes intègrent un **détecteur de deadlock**. Lorsqu'un deadlock est identifié, le SGBD doit le résoudre pour éviter un blocage permanent. Il choisit une des transactions impliquées comme "victime" (souvent celle qui a le moins de travail déjà effectué ou qui est la plus facile à annuler) et effectue un `ROLLBACK` sur cette transaction. Cela libère les verrous détenus par la victime, permettant à l'autre transaction de se terminer avec succès.

#### **Phase 4 : Résolution du Deadlock**

La solution la plus efficace est d'établir un **ordre de verrouillage cohérent**.

1.  **Réinitialisation des soldes :**
    
```sql
    UPDATE Comptes SET solde = 1000.00 WHERE id_compte = 1;
    UPDATE Comptes SET solde = 500.00 WHERE id_compte = 2;
    SELECT * FROM Comptes; -- Vérifiez
    ```


2.  **Transactions modifiées avec ordre de verrouillage cohérent :**
    Nous allons toujours mettre à jour le compte avec le plus petit `id_compte` en premier, puis le plus grand.

    **Session 1 (Transaction 1 - Modifiée) :**
    
```sql
    -- Session 1 (Transaction 1) - Ordre cohérent
    BEGIN TRANSACTION;

    -- Alice envoie 100 à Bob
    -- Étape 1 : Débiter Alice (id_compte = 1) - plus petit ID
    UPDATE Comptes
    SET solde = solde - 100.00
    WHERE id_compte = 1;

    -- Simule un délai
    SELECT pg_sleep(5); -- Adaptez cette ligne à votre SGBD

    -- Étape 2 : Créditer Bob (id_compte = 2) - plus grand ID
    UPDATE Comptes
    SET solde = solde + 100.00
    WHERE id_compte = 2;

    COMMIT;
    ```


    **Session 2 (Transaction 2 - Modifiée) :**
    
```sql
    -- Session 2 (Transaction 2) - Ordre cohérent
    BEGIN TRANSACTION;

    -- Bob envoie 50 à Alice
    -- Étape 1 : Créditer Alice (id_compte = 1) - plus petit ID
    UPDATE Comptes
    SET solde = solde + 50.00
    WHERE id_compte = 1;

    -- Simule un délai
    SELECT pg_sleep(5); -- Adaptez cette ligne à votre SGBD

    -- Étape 2 : Débiter Bob (id_compte = 2) - plus grand ID
    UPDATE Comptes
    SET solde = solde - 50.00
    WHERE id_compte = 2;

    COMMIT;
    ```


3.  **Répétez la procédure d'exécution :**
    *   Lancez la première partie de T1 (jusqu'au `pg_sleep`).
    *   Lancez la première partie de T2 (jusqu'au `pg_sleep`).
    *   Lancez la suite de T1.
    *   Lancez la suite de T2.

**Observation :**
*   Le deadlock ne s'est pas reproduit.
*   Les transactions se sont terminées avec succès, l'une après l'autre (T1 attendra T2 pour le `Compte 1` si T2 le verrouille en premier, ou vice-versa, mais il n'y aura pas d'attente circulaire).
*   **Soldes finaux :**
    *   Alice : 1000 (initial) - 100 (T1) + 50 (T2) = 950.00
    *   Bob : 500 (initial) + 100 (T1) - 50 (T2) = 550.00
    *   Vérifiez : `SELECT * FROM Comptes;`

---

### **Chapitre 2 : Approfondissement des Stratégies et Bonnes Pratiques**

Ce chapitre explore des stratégies plus avancées pour la gestion des deadlocks et des considérations pour leur implémentation en production.

#### **Phase 5 : Discussion et Approfondissement**

1.  **Autres stratégies pour minimiser ou gérer les deadlocks :**
    *   **Niveaux d'isolation des transactions :** Des niveaux d'isolation plus bas (comme `READ COMMITTED` par défaut dans PostgreSQL) peuvent réduire les verrous, mais augmentent les risques d'autres anomalies (lectures non répétables, lectures fantômes). `SERIALIZABLE` offre la plus forte garantie mais peut augmenter les deadlocks ou les échecs de sérialisation. Le choix dépend du compromis entre intégrité des données et concurrence.
    *   **Transactions courtes et rapides :** Plus une transaction est courte, moins elle détient de verrous longtemps, réduisant ainsi la fenêtre de temps pendant laquelle un deadlock peut se produire.
    *   **Utilisation de `SELECT ... FOR UPDATE` (ou `FOR SHARE`) :** Verrouiller explicitement les lignes nécessaires au début de la transaction, dans un ordre cohérent, peut prévenir les deadlocks en acquérant tous les verrous requis avant de commencer les modifications.
        
```sql
        -- Exemple avec SELECT FOR UPDATE pour T1
        BEGIN TRANSACTION;
        SELECT solde FROM Comptes WHERE id_compte = 1 FOR UPDATE; -- Verrouille 1
        SELECT solde FROM Comptes WHERE id_compte = 2 FOR UPDATE; -- Verrouille 2 (si disponible)
        -- Puis les UPDATE
        UPDATE Comptes SET solde = solde - 100.00 WHERE id_compte = 1;
        UPDATE Comptes SET solde = solde + 100.00 WHERE id_compte = 2;
        COMMIT;
        ```

    *   **Gestion des erreurs et re-tentatives (Retry Logic) :** Puisque les deadlocks sont parfois inévitables même avec les meilleures pratiques, l'application doit être capable de détecter un échec de deadlock, d'annuler la transaction et de la re-tenter après un court délai.
    *   **Verrouillage au niveau de la table :** Dans des cas très spécifiques et avec une compréhension claire des implications sur la concurrence, un verrouillage au niveau de la table (`LOCK TABLE`) peut être utilisé pour des opérations massives, mais cela réduit drastiquement la concurrence.

2.  **Assurer un ordre de verrouillage cohérent dans des systèmes complexes :**
    *   **Conventions et documentation :** Établir des conventions claires pour l'ordre de verrouillage (ex: toujours par clé primaire croissante, ou par ordre alphabétique des noms de table, puis par clé primaire) et les documenter rigoureusement.
    *   **Frameworks et ORM :** Utiliser des Object-Relational Mappers (ORM) ou des frameworks de persistance qui peuvent aider à gérer l'ordre des opérations ou à fournir des outils pour le verrouillage. Cependant, une mauvaise utilisation peut aussi introduire des deadlocks.
    *   **Code Reviews :** Intégrer la vérification de l'ordre de verrouillage dans les revues de code pour s'assurer que les développeurs respectent les conventions.
    *   **Tests automatisés :** Mettre en place des tests de concurrence qui simulent des scénarios de deadlocks pour détecter les problèmes tôt dans le cycle de développement.
    *   **Microservices :** Dans une architecture de microservices, les transactions distribuées peuvent introduire des défis supplémentaires. Des patterns comme le "Saga" ou l'utilisation de messages asynchrones peuvent aider à gérer la cohérence sans transactions monolithiques.

3.  **Implémentation d'un mécanisme de re-tentative (retry logic) pour un deadlock :**
    Si une transaction est victime d'un deadlock, une logique de re-tentative est essentielle.
    *   **Détection de l'erreur :** L'application doit intercepter le code d'erreur spécifique au deadlock de son SGBD (ex: `SQLSTATE 40P01` pour PostgreSQL).
    *   **Délai exponentiel (Exponential Backoff) :** Au lieu de re-tenter immédiatement, introduire un délai croissant entre chaque tentative (ex: 1s, 2s, 4s, 8s...). Cela réduit la probabilité que la transaction re-tente et entre en collision avec la même transaction ou d'autres transactions concurrentes.
    *   **Nombre maximal de tentatives :** Définir un nombre maximum de re-tentatives. Si après N tentatives la transaction échoue toujours, elle doit être signalée comme une erreur irrécupérable.
    *   **Idempotence :** S'assurer que la transaction est idempotente, c'est-à-dire que la re-exécution de la transaction plusieurs fois produit le même résultat final que si elle n'avait été exécutée qu'une seule fois. C'est crucial car la transaction peut avoir effectué des opérations partielles avant d'être annulée.
    *   **Journalisation :** Enregistrer les deadlocks et les re-tentatives pour faciliter le débogage et l'analyse des performances.
    *   **Contexte de la transaction :** La logique de re-tentative doit re-démarrer la transaction depuis le début, y compris la re-lecture des données si nécessaire, pour s'assurer qu'elle opère sur l'état le plus récent de la base de données.