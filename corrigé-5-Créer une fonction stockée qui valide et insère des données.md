Voici deux solutions pour le TP sur la fonction PL/pgSQL de validation et d'insertion sécurisée, présentées en chapitres distincts pour offrir des perspectives légèrement différentes.

---

### **Chapitre 1 : Solution Basique avec Messages d'Erreur Textuels**

Cette première solution implémente la fonction `fn_inserer_tache_securisee` en suivant scrupuleusement les exigences du TP, retournant un message textuel en cas de succès ou d'échec de validation.

#### **1. Préparation de l'Environnement**

(Le code de création des tables et d'insertion des données est identique à celui fourni dans le TP et n'est pas répété ici pour concision.)

#### **2. Création de la Fonction `fn_inserer_tache_securisee`**



```sql
CREATE OR REPLACE FUNCTION fn_inserer_tache_securisee(
    p_projet_id INT,
    p_nom_tache VARCHAR,
    p_description TEXT,
    p_date_debut DATE,
    p_date_fin DATE,
    p_statut VARCHAR,
    p_priorite VARCHAR,
    p_assigne_a INT
)
RETURNS TEXT
LANGUAGE plpgsql
AS $$
DECLARE
    v_projet_existe BOOLEAN;
    v_utilisateur_existe BOOLEAN;
BEGIN
    -- 1. Validation de l'existence du Projet
    SELECT EXISTS (SELECT 1 FROM Projets WHERE projet_id = p_projet_id) INTO v_projet_existe;
    IF NOT v_projet_existe THEN
        RETURN 'Erreur: Le projet spécifié (ID: ' || p_projet_id || ') n''existe pas.';
    END IF;

    -- 2. Validation du Nom de la Tâche
    IF p_nom_tache IS NULL OR TRIM(p_nom_tache) = '' THEN
        RETURN 'Erreur: Le nom de la tâche ne peut pas être vide.';
    END IF;

    -- 3. Validation des Dates de la Tâche
    IF p_date_debut IS NULL THEN
        RETURN 'Erreur: La date de début de la tâche ne peut pas être NULL.';
    END IF;

    IF p_date_fin IS NOT NULL AND p_date_fin < p_date_debut THEN
        RETURN 'Erreur: La date de fin de la tâche ne peut pas être antérieure à la date de début.';
    END IF;

    -- 4. Validation du Statut de la Tâche
    IF p_statut IS NULL OR p_statut NOT IN ('À faire', 'En cours', 'Terminée', 'Annulée') THEN
        RETURN 'Erreur: Le statut de la tâche est invalide. Valeurs acceptées : ''À faire'', ''En cours'', ''Terminée'', ''Annulée''.';
    END IF;

    -- 5. Validation de la Priorité de la Tâche
    IF p_priorite IS NULL OR p_priorite NOT IN ('Basse', 'Moyenne', 'Haute', 'Urgent') THEN
        RETURN 'Erreur: La priorité de la tâche est invalide. Valeurs acceptées : ''Basse'', ''Moyenne'', ''Haute'', ''Urgent''.';
    END IF;

    -- 6. Validation de l'existence de l'Utilisateur assigné
    IF p_assigne_a IS NOT NULL THEN
        SELECT EXISTS (SELECT 1 FROM Utilisateurs WHERE utilisateur_id = p_assigne_a) INTO v_utilisateur_existe;
        IF NOT v_utilisateur_existe THEN
            RETURN 'Erreur: L''utilisateur assigné (ID: ' || p_assigne_a || ') n''existe pas.';
        END IF;
    END IF;

    -- Si toutes les validations passent, insérer la tâche
    INSERT INTO Taches (
        projet_id,
        nom_tache,
        description,
        date_debut,
        date_fin,
        statut,
        priorite,
        assigne_a
    ) VALUES (
        p_projet_id,
        p_nom_tache,
        p_description,
        p_date_debut,
        p_date_fin,
        p_statut,
        p_priorite,
        p_assigne_a
    );

    RETURN 'OK';

END;
$$;
```



#### **3. Exemples d'Appels et Vérification**

(Les exemples d'appels et la commande de vérification sont identiques à ceux fournis dans le TP et ne sont pas répétés ici pour concision.)

**Explication de la solution :**

*   La fonction est définie avec `RETURNS TEXT` pour renvoyer des messages d'état.
*   Chaque règle de validation est implémentée avec un bloc `IF ... THEN ... END IF;`.
*   Pour les validations d'existence (`p_projet_id`, `p_assigne_a`), une sous-requête avec `EXISTS` est utilisée et son résultat est stocké dans une variable `BOOLEAN` pour une meilleure lisibilité.
*   En cas d'échec d'une validation, la fonction retourne immédiatement un message d'erreur descriptif.
*   Si toutes les validations sont réussies, l'instruction `INSERT` est exécutée, puis la fonction retourne `'OK'`.
*   L'utilisation de `TRIM(p_nom_tache) = ''` pour le nom de la tâche permet de gérer les chaînes composées uniquement d'espaces.

---

### **Chapitre 2 : Solution Avancée avec Retour d'ID et Gestion des Exceptions**

Cette solution étend la première en retournant l'ID de la tâche insérée en cas de succès, et en utilisant un bloc `EXCEPTION` pour une gestion plus robuste des erreurs inattendues (bien que les validations métier soient toujours gérées explicitement). Elle aborde également l'unicité du nom de tâche par projet.

#### **1. Préparation de l'Environnement**

(Le code de création des tables et d'insertion des données est identique à celui fourni dans le TP et n'est pas répété ici pour concision.)

#### **2. Création de la Fonction `fn_inserer_tache_securisee_v2`**

Pour cette version, nous allons modifier le type de retour pour inclure l'ID de la tâche et le message. Une approche courante est de retourner un `RECORD` ou un type composite, mais pour rester simple et proche du `TEXT` demandé, nous pouvons retourner un format JSON ou un texte structuré. Ici, nous allons retourner un `TEXT` mais avec un formatage spécifique.

Nous allons également ajouter une contrainte d'unicité sur la table `Taches` pour le défi optionnel.


```sql
-- Ajout d'une contrainte d'unicité pour le défi optionnel
ALTER TABLE Taches
ADD CONSTRAINT uq_tache_projet_nom UNIQUE (projet_id, nom_tache);
```




```sql
CREATE OR REPLACE FUNCTION fn_inserer_tache_securisee_v2(
    p_projet_id INT,
    p_nom_tache VARCHAR,
    p_description TEXT,
    p_date_debut DATE,
    p_date_fin DATE,
    p_statut VARCHAR,
    p_priorite VARCHAR,
    p_assigne_a INT
)
RETURNS TEXT -- Retourne 'OK:ID_TACHE' ou 'ERREUR:Message'
LANGUAGE plpgsql
AS $$
DECLARE
    v_projet_existe BOOLEAN;
    v_utilisateur_existe BOOLEAN;
    v_nouvel_id_tache INT;
    v_tache_nom_existe BOOLEAN; -- Pour la validation d'unicité
BEGIN
    -- 1. Validation de l'existence du Projet
    SELECT EXISTS (SELECT 1 FROM Projets WHERE projet_id = p_projet_id) INTO v_projet_existe;
    IF NOT v_projet_existe THEN
        RETURN 'ERREUR: Le projet spécifié (ID: ' || p_projet_id || ') n''existe pas.';
    END IF;

    -- 2. Validation du Nom de la Tâche
    IF p_nom_tache IS NULL OR TRIM(p_nom_tache) = '' THEN
        RETURN 'ERREUR: Le nom de la tâche ne peut pas être vide.';
    END IF;

    -- Validation d'unicité du nom de tâche par projet (Défi optionnel)
    SELECT EXISTS (SELECT 1 FROM Taches WHERE projet_id = p_projet_id AND nom_tache = p_nom_tache) INTO v_tache_nom_existe;
    IF v_tache_nom_existe THEN
        RETURN 'ERREUR: Une tâche avec le nom ''' || p_nom_tache || ''' existe déjà pour ce projet (ID: ' || p_projet_id || ').';
    END IF;

    -- 3. Validation des Dates de la Tâche
    IF p_date_debut IS NULL THEN
        RETURN 'ERREUR: La date de début de la tâche ne peut pas être NULL.';
    END IF;

    IF p_date_fin IS NOT NULL AND p_date_fin < p_date_debut THEN
        RETURN 'ERREUR: La date de fin de la tâche ne peut pas être antérieure à la date de début.';
    END IF;

    -- 4. Validation du Statut de la Tâche (peut être omise si la contrainte CHECK est suffisante)
    -- Ici, nous la gardons pour un message d'erreur plus convivial avant l'échec de la contrainte.
    IF p_statut IS NULL OR p_statut NOT IN ('À faire', 'En cours', 'Terminée', 'Annulée') THEN
        RETURN 'ERREUR: Le statut de la tâche est invalide. Valeurs acceptées : ''À faire'', ''En cours'', ''Terminée'', ''Annulée''.';
    END IF;

    -- 5. Validation de la Priorité de la Tâche (idem, peut être omise si CHECK est suffisante)
    IF p_priorite IS NULL OR p_priorite NOT IN ('Basse', 'Moyenne', 'Haute', 'Urgent') THEN
        RETURN 'ERREUR: La priorité de la tâche est invalide. Valeurs acceptées : ''Basse'', ''Moyenne'', ''Haute'', ''Urgent''.';
    END IF;

    -- 6. Validation de l'existence de l'Utilisateur assigné
    IF p_assigne_a IS NOT NULL THEN
        SELECT EXISTS (SELECT 1 FROM Utilisateurs WHERE utilisateur_id = p_assigne_a) INTO v_utilisateur_existe;
        IF NOT v_utilisateur_existe THEN
            RETURN 'ERREUR: L''utilisateur assigné (ID: ' || p_assigne_a || ') n''existe pas.';
        END IF;
    END IF;

    -- Si toutes les validations passent, insérer la tâche
    INSERT INTO Taches (
        projet_id,
        nom_tache,
        description,
        date_debut,
        date_fin,
        statut,
        priorite,
        assigne_a
    ) VALUES (
        p_projet_id,
        p_nom_tache,
        p_description,
        p_date_debut,
        p_date_fin,
        p_statut,
        p_priorite,
        p_assigne_a
    )
    RETURNING tache_id INTO v_nouvel_id_tache; -- Récupère l'ID de la tâche insérée

    RETURN 'OK:' || v_nouvel_id_tache;

EXCEPTION
    -- Gestion des erreurs inattendues (par exemple, violation de contrainte non gérée explicitement)
    WHEN OTHERS THEN
        RETURN 'ERREUR: Une erreur inattendue est survenue lors de l''insertion : ' || SQLERRM;
END;
$$;
```



#### **3. Exemples d'Appels et Vérification**



```sql
-- Test 1 : Insertion réussie
SELECT fn_inserer_tache_securisee_v2(
    1, -- projet_id (Refonte Site Web)
    'Tests d''intégration V2', -- Nom unique
    'Réalisation des tests unitaires et d''intégration.',
    '2023-05-10',
    '2023-05-25',
    'En cours',
    'Haute',
    2 -- assigne_a (Bob Martin)
);
-- Résultat attendu : 'OK:X' où X est le nouvel ID de tâche

-- Test 2 : Projet inexistant
SELECT fn_inserer_tache_securisee_v2(
    999, -- projet_id inexistant
    'Nouvelle tâche V2',
    'Description',
    '2023-01-01',
    '2023-01-05',
    'À faire',
    'Moyenne',
    1
);
-- Résultat attendu : 'ERREUR: Le projet spécifié (ID: 999) n'existe pas.'

-- Test 3 : Nom de tâche vide
SELECT fn_inserer_tache_securisee_v2(
    1,
    '', -- Nom vide
    'Description',
    '2023-01-01',
    '2023-01-05',
    'À faire',
    'Moyenne',
    1
);
-- Résultat attendu : 'ERREUR: Le nom de la tâche ne peut pas être vide.'

-- Test 4 : Date de fin antérieure à la date de début
SELECT fn_inserer_tache_securisee_v2(
    1,
    'Tâche avec dates invalides V2',
    'Description',
    '2023-02-01',
    '2023-01-20', -- Date de fin < Date de début
    'À faire',
    'Moyenne',
    1
);
-- Résultat attendu : 'ERREUR: La date de fin de la tâche ne peut pas être antérieure à la date de début.'

-- Test 5 : Statut invalide
SELECT fn_inserer_tache_securisee_v2(
    1,
    'Tâche avec statut invalide V2',
    'Description',
    '2023-01-01',
    '2023-01-05',
    'En attente', -- Statut invalide
    'Moyenne',
    1
);
-- Résultat attendu : 'ERREUR: Le statut de la tâche est invalide...'

-- Test 6 : Utilisateur assigné inexistant
SELECT fn_inserer_tache_securisee_v2(
    1,
    'Tâche assignée à un utilisateur inexistant V2',
    'Description',
    '2023-01-01',
    '2023-01-05',
    'À faire',
    'Moyenne',
    999 -- Utilisateur inexistant
);
-- Résultat attendu : 'ERREUR: L'utilisateur assigné (ID: 999) n'existe pas.'

-- Test 7 : Nom de tâche déjà existant pour le même projet (défi optionnel)
SELECT fn_inserer_tache_securisee_v2(
    1,
    'Tests d''intégration V2', -- Ce nom a été inséré au Test 1
    'Description',
    '2023-01-01',
    '2023-01-05',
    'À faire',
    'Moyenne',
    1
);
-- Résultat attendu : 'ERREUR: Une tâche avec le nom 'Tests d'intégration V2' existe déjà pour ce projet (ID: 1).'

-- Vérifier les insertions réussies
SELECT * FROM Taches ORDER BY tache_id DESC;
```



**Explication de la solution avancée :**

*   **Retour d'ID :** La fonction utilise `RETURNING tache_id INTO v_nouvel_id_tache;` dans l'instruction `INSERT` pour capturer l'ID généré par la séquence `SERIAL`. Le résultat est ensuite formaté comme `'OK:ID_TACHE'`.
*   **Validation d'unicité (Défi optionnel) :** Une nouvelle variable `v_tache_nom_existe` et une validation `IF` sont ajoutées pour vérifier si une tâche avec le même nom existe déjà pour le `projet_id` donné. Une contrainte `UNIQUE` a été ajoutée à la table `Taches` pour renforcer cette règle au niveau de la base de données.
*   **Bloc `EXCEPTION` :** Le bloc `EXCEPTION WHEN OTHERS THEN ...` est ajouté pour attraper toute erreur inattendue qui pourrait survenir pendant l'exécution de la fonction (par exemple, une violation de contrainte non explicitement gérée par un `IF`). `SQLERRM` fournit le message d'erreur standard de PostgreSQL. Cela rend la fonction plus robuste.
*   **Messages d'erreur :** Les messages d'erreur sont préfixés par `'ERREUR:'` pour faciliter le traitement côté application.

Ces deux solutions vous offrent une base solide pour la création de fonctions PL/pgSQL avec validation. La deuxième solution, avec le retour d'ID et la gestion des exceptions, est généralement préférée en production pour sa robustesse et sa facilité d'intégration avec les applications.