# TP PostgreSQL Avancé – ERP *From Scratch*

## Objectifs pédagogiques

À l’issue de ce TP, vous serez capables de :

* Concevoir une **base de données d’ERP réaliste** et cohérente
* Écrire un **script SQL complet et industrialisable**
* Mettre en œuvre des **concepts SQL avancés PostgreSQL**
* Gérer la **sécurité**, les **transactions**, les **concurrences** et les **incidents**
* Penser la base de données comme un **composant logiciel critique**

---

## Contexte métier

Votre entreprise développe un **ERP interne** destiné à gérer :

* les employés
* les départements
* les projets
* la facturation
* les droits d’accès

L’ERP sera utilisé par :

* des utilisateurs métiers
* des administrateurs
* des applications backend

La base PostgreSQL est **le cœur du système**.

## Contraintes techniques

* SGBD : **PostgreSQL uniquement**
* SQL natif (pas d’ORM)
* Scripts **rejouables**
* Toute décision doit être **justifiable techniquement**

---

## PARTIE 1 – Modélisation
### Objectif

Concevoir le **schéma relationnel** de l’ERP.

### Entités minimales attendues

* `employee`
* `department`
* `project`
* `employee_project`
* `salary_history`
* `invoice`
* `invoice_line`
* `audit_log`

### Contraintes obligatoires

* clés primaires (simples et/ou composées)
* clés étrangères avec règles ON DELETE adaptées
* contraintes `CHECK`
* contraintes `UNIQUE`
* valeurs par défaut
* types adaptés (`NUMERIC`, `DATE`, `TIMESTAMP`, `ENUM`, etc.)

**Livrable** : script `01_schema.sql`

---

## PARTIE 2 – Intégrité avancée & logique métier

### Objectif

Implémenter des **règles métier au niveau base**.

### Exemples de règles attendues

* un employé ne peut pas être affecté deux fois au même projet
* un salaire ne peut pas diminuer sans justification
* une facture doit avoir au moins une ligne
* une facture validée devient immuable

### Outils proposés

* `TRIGGER`
* `FUNCTION plpgsql`
* contraintes différées (`DEFERRABLE INITIALLY DEFERRED`)

**Livrable** : script `02_business_rules.sql`

---

## PARTIE 3 – Sécurité, rôles et privilège

### Objectif

Mettre en place une **politique de sécurité réaliste**.

### Rôles attendus

* `erp_admin`
* `erp_app`
* `erp_hr`
* `erp_readonly`

### Contraintes

* aucun accès direct aux tables sensibles
* accès via vues ou fonctions
* séparation stricte lecture / écriture
* principe du **least privilege**

### Concepts à utiliser

* `CREATE ROLE`
* `GRANT / REVOKE`
* `SECURITY DEFINER`
* vues sécurisées

**Livrable** : script `03_security.sql`

---

## PARTIE 4 – Transactions, concurrence & deadlocks

### Objectif

Comprendre et maîtriser les **problèmes de concurrence**.

### Travaux demandés

1. Écrire une transaction critique (ex : validation de facture)
2. Simuler un **deadlock**
3. Proposer une stratégie de prévention

### Concepts attendus

* niveaux d’isolation
* verrous (`FOR UPDATE`, `NOWAIT`, `SKIP LOCKED`)
* gestion d’erreur transactionnelle

**Livrable** : script `04_transactions.sql` + explication

---

## PARTIE 5 – Maintenance, audit & forensic

### Objectif

Préparer la base pour la **production**.

### Attendus

* table d’audit automatique
* triggers d’historisation
* détection d’anomalies (salaires, accès)
* procédures de purge

### Concepts avancés

* `EVENT TRIGGER`
* `LISTEN / NOTIFY`
* vues matérialisées
* stratégies d’indexation

**Livrable** : script `05_maintenance.sql`

---

## Livrables finaux

* Scripts SQL versionnés
* README technique justifiant les choix
* Diagramme relationnel


