# TP - SQL

## **Validation métier et insertion sécurisée avec PL/pgSQL**

### Objectifs pédagogiques

À l’issue de ce TP, l’étudiant devra être capable de :

* Concevoir une **fonction PL/pgSQL** robuste
* Implémenter des **règles de validation métier** côté base de données
* Gérer les **erreurs fonctionnelles** sans interrompre brutalement l’exécution
* Comprendre la différence entre :

  * validation applicative
  * contraintes SQL
  * gestion des exceptions
* Produire une solution exploitable par une application cliente

---

## Contexte

Vous travaillez sur une base de données de **gestion de projets** contenant notamment les tables suivantes :

* `Projets`
* `Utilisateurs`
* `Taches`

Une tâche :

* appartient obligatoirement à un projet
* peut être assignée à un utilisateur
* possède des informations métiers (dates, statut, priorité, etc.)

L’objectif est de **sécuriser l’insertion d’une tâche** en centralisant les contrôles dans une fonction stockée.

---

## Schéma logique (fourni)

Les tables sont déjà créées et peuplées.
Vous disposez notamment de :

* `Projets(projet_id, …)`
* `Utilisateurs(utilisateur_id, …)`
* `Taches(tache_id, projet_id, nom_tache, description, date_debut, date_fin, statut, priorite, assigne_a)`

---

## Travail demandé

### **Partie 1 – Fonction de validation et d’insertion sécurisée **

Vous devez créer une fonction PostgreSQL nommée :

```sql
fn_inserer_tache_securisee(...)
```

#### Paramètres attendus

La fonction prendra en paramètre :

* l’identifiant du projet
* le nom de la tâche
* la description
* la date de début
* la date de fin (optionnelle)
* le statut
* la priorité
* l’utilisateur assigné (optionnel)

---

### Règles de validation à implémenter

La fonction devra **vérifier explicitement** les règles suivantes :

1. **Existence du projet**

   * Le projet référencé doit exister
   * Sinon, la fonction doit retourner un message d’erreur

2. **Nom de la tâche**

   * Ne doit pas être `NULL`
   * Ne doit pas être vide ou composé uniquement d’espaces

3. **Dates**

   * La date de début est obligatoire
   * Si une date de fin est fournie, elle ne doit pas être antérieure à la date de début

4. **Statut**

   * Doit appartenir à l’ensemble :

     * `À faire`
     * `En cours`
     * `Terminée`
     * `Annulée`

5. **Priorité**

   * Doit appartenir à l’ensemble :

     * `Basse`
     * `Moyenne`
     * `Haute`
     * `Urgent`

6. **Utilisateur assigné**

   * Facultatif
   * S’il est renseigné, l’utilisateur doit exister

---

### Comportement attendu (Partie 1)

* Si **une validation échoue** :

  * la fonction **ne doit pas insérer de donnée**
  * elle retourne un **message textuel explicite**
* Si **toutes les validations passent** :

  * la tâche est insérée
  * la fonction retourne `'OK'`

 **Aucune exception SQL ne doit être levée dans cette partie**

---

## Partie 2 – Version avancée

Créer une seconde fonction :

```sql
fn_inserer_tache_securisee_v2(...)
```

Cette version devra :

1. **Retourner l’identifiant de la tâche insérée**

   * Format attendu :

     ```text
     OK:ID_TACHE
     ```

2. **Préfixer les messages d’erreur**

   * Format :

     ```text
     ERREUR: message descriptif
     ```

3. **Gérer les erreurs inattendues**

   * Utiliser un bloc :

     ```sql
     EXCEPTION WHEN OTHERS THEN
     ```
   * Retourner un message d’erreur générique incluant `SQLERRM`

---

## Tests attendus

Vous devrez fournir des appels de test couvrant au minimum :

* insertion valide
* projet inexistant
* nom vide
* dates incohérentes
* statut invalide
* priorité invalide
* utilisateur inexistant
* doublon de nom (si défi réalisé)





