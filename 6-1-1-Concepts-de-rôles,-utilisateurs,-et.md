# Concepts de rôles, utilisateurs et groupes dans le modèle de sécurité PostgreSQL

PostgreSQL repose sur un modèle de sécurité basé sur des **rôles**, qui facilitent la gestion des permissions et la hiérarchisation des accès aux ressources de la base. Comprendre ces concepts permet d’implémenter une politique de sécurité efficace et adaptée aux besoins métier.

---

## 1. Rôles : une notion unifiée pour utilisateurs et groupes

PostgreSQL ne fait pas de distinction stricte entre **utilisateurs** et **groupes**. Ces deux notions sont unifiées sous le concept de **rôles**. Un rôle peut :

- Avoir des privilèges (par exemple, CREATE, SELECT, INSERT)  
- Se connecter à la base (`LOGIN`) — dans ce cas, c’est un rôle "utilisateur"  
- Être membre d’autre(s) rôle(s) — formant ainsi un groupe

### Exemple : création d’un rôle utilisateur

```sql
CREATE ROLE julien LOGIN PASSWORD 'motdepasse123';
```

### Exemple : création d’un rôle groupe

```sql
CREATE ROLE equipe_technique NOLOGIN;
```

---

## 2. Gestion des membres : rôles comme groupes

Un rôle peut être membre d’autres rôles via la commande `GRANT`, accordant ainsi les privilèges de ces groupes.

```sql
GRANT equipe_technique TO julien;
```

Julien aura alors tous les privilèges associés au rôle `equipe_technique`.

---

## 3. Propriétés utiles des rôles

- **SUPERUSER** : super-privileges, accès global  
- **CREATEDB** : permet de créer des bases de données  
- **CREATEROLE** : autorise à créer et modifier d’autres rôles  
- **LOGIN** : permet la connexion directe

Un rôle peut combiner ces propriétés selon ses besoins.

---

## 4. Modèle d’héritage des droits

Par défaut, un rôle hérite des permissions de tous les rôles dont il est membre. Cela simplifie la gestion centralisée des droits.

Si nécessaire, l’héritage peut être désactivé (`NOINHERIT`), limitant l’accès direct aux permissions du rôle.

---

## 5. Exemple concret : création et attribution de rôles

```sql
-- Création d’un groupe admin avec droits élevés
CREATE ROLE admin NOLOGIN CREATEDB CREATEROLE;

-- Création d’un utilisateur avec connexion et mot de passe
CREATE ROLE claire LOGIN PASSWORD 'secret!';

-- Attribuer le rôle admin à claire
GRANT admin TO claire;
```

Clare pourra se connecter avec son mot de passe et aura les droits pour créer des bases et gérer des rôles.

---

## 6. Diagramme Mermaid : relation rôles, utilisateurs et groupes

```mermaid
graph TD
  Utilisateur[Utilisateur (rôle LOGIN)]
  Groupe[Groupe (rôle NOLOGIN)]
  Utilisateur -->|membre de| Groupe
  Groupe -->|possède privilèges| Privilèges[Privilèges (CREATE, SELECT...)]

  subgraph Propriétés
    LOGIN
    CREATEDB
    CREATEROLE
    SUPERUSER
  end

  Utilisateur --> Propriétés
  Groupe --> Propriétés
```

---

## 7. Commandes utiles

- Lister tous les rôles :

```sql
\du
```

- Modifier un rôle :

```sql
ALTER ROLE claire WITH CREATEDB;
```

- Supprimer un rôle :

```sql
DROP ROLE julien;
```

---

## 8. Sources et références

- [PostgreSQL Documentation - SQL Syntax - Role Management](https://www.postgresql.org/docs/current/sql-createrole.html)  
- [PostgreSQL Documentation - Role Attributes](https://www.postgresql.org/docs/current/user-manag.html#USER-MANAG-ROLES)  
- [Cybertec PostgreSQL - User and Role Management](https://www.cybertec-postgresql.com/en/postgresql-user-management/)  
- [Official PostgreSQL Wiki - Roles and Privileges](https://wiki.postgresql.org/wiki/User_Management)  

---

La gestion des rôles dans PostgreSQL permet une administration fine et flexible des droits, que ce soit pour isoler les utilisateurs, regrouper des permissions ou matérialiser une hiérarchie d’accès. Cette base solide est indispensable pour la sécurisation et la gouvernance des données.