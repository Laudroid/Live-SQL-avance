# TP SQL – Transactions concurrentes, deadlocks et stratégies de résolution

## Contexte pédagogique

Les bases de données relationnelles sont très souvent utilisées dans des environnements **concurrents**, où plusieurs utilisateurs ou processus exécutent des transactions en parallèle.

Dans ces contextes, les mécanismes de verrouillage sont indispensables pour garantir :

* la **cohérence des données**,
* l’**isolation transactionnelle**.

Cependant, une mauvaise gestion des verrous peut conduire à des situations critiques appelées **deadlocks** (interblocages), dans lesquelles deux transactions ou plus se bloquent mutuellement.

Ce TP a pour objectif de vous faire :

* **provoquer volontairement un deadlock**,
* l’**analyser finement**,
* et mettre en œuvre une **stratégie de conception robuste** pour l’éviter.

Il s’agit d’un TP fondamental en génie logiciel, directement lié aux problématiques rencontrées en production.

---

## Domaine fonctionnel

On considère un système bancaire simplifié avec des **comptes** et des **transferts d’argent**.

### Table `Comptes`

| Colonne       | Description           |
| ------------- | --------------------- |
| id_compte     | Identifiant du compte |
| nom_titulaire | Nom du titulaire      |
| solde         | Solde du compte       |

Chaque transfert d’argent est implémenté par une transaction contenant :

* un débit sur un compte,
* un crédit sur un autre compte.

---

## Objectifs du TP

À l’issue du TP, vous devrez être capables de :

1. Expliquer ce qu’est un **deadlock** et pourquoi il survient.
2. Reproduire un deadlock à l’aide de transactions concurrentes.
3. Analyser le comportement du SGBD face à un deadlock.
4. Proposer et implémenter une **stratégie de prévention** efficace.
5. Relier ces mécanismes à des bonnes pratiques de conception logicielle.

---

## Travail demandé

### Partie 1 – Mise en place de l’environnement

1. Créez la table `Comptes`.
2. Insérez des données initiales représentant deux comptes bancaires.
3. Vérifiez l’état initial des soldes.

Objectif pédagogique :
disposer d’un environnement minimal mais suffisant pour illustrer la concurrence.

---

### Partie 2 – Simulation d’un deadlock

Dans cette partie, vous allez volontairement créer une situation de deadlock.

#### Consignes

1. Ouvrez **deux sessions SQL distinctes** (deux connexions à la base).
2. Dans chaque session :

   * démarrez explicitement une transaction (`BEGIN` ou `BEGIN TRANSACTION`),
   * effectuez une mise à jour sur un compte différent.
3. Introduisez volontairement un **délai d’exécution** (`sleep`) afin de forcer l’entrelacement des transactions.
4. Essayez ensuite d’accéder, dans chaque transaction, au compte déjà verrouillé par l’autre transaction.

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

Vous devez observer :

* un blocage temporaire,
* puis l’échec d’une des transactions.

Objectif pédagogique :
comprendre que le deadlock est une **conséquence logique** d’un mauvais ordre d’accès aux ressources.

---

### Partie 3 – Analyse du deadlock

Dans un court texte explicatif :

1. Décrivez précisément :

   * quelles ressources sont verrouillées,
   * par quelle transaction,
   * et lesquelles sont attendues.
2. Expliquez la notion d’**attente circulaire (circular wait)**.
3. Indiquez le rôle du **détecteur de deadlock du SGBD** :

   * comment il identifie le problème,
   * pourquoi il annule une transaction,
   * et ce que cela implique pour l’application.

Objectif pédagogique :
passer d’un simple constat d’erreur à une **compréhension systémique**.

---

### Partie 4 – Résolution du deadlock par conception

Vous allez maintenant corriger le problème **sans supprimer la concurrence**, mais en modifiant la logique des transactions.

#### Consignes

1. Réinitialisez les soldes des comptes.
2. Modifiez les transactions pour imposer un **ordre de verrouillage cohérent** :

   * par exemple, toujours accéder d’abord au compte ayant l’identifiant le plus petit.
3. Rejouez exactement le même scénario concurrent (deux sessions, délais, etc.).
4. Vérifiez que :

   * aucune transaction n’est annulée,
   * les soldes finaux sont cohérents.

Objectif pédagogique :
comprendre que les deadlocks se préviennent **principalement par la conception**, pas par des rustines techniques.

---

## Contraintes techniques

* Vous devez travailler avec des **transactions explicites**.
* Les tests doivent être réalisés avec **au moins deux sessions concurrentes**.
* Les scripts doivent être :

  * lisibles,
  * commentés,
  * reproductibles.
* Les fonctions de temporisation (`sleep`) doivent être adaptées au SGBD utilisé.

---

## Livrables attendus

* Un fichier `.sql` contenant :

  * la création de la table,
  * la simulation du deadlock,
  * la version corrigée des transactions.
* Un court document expliquant :

  * le deadlock observé,
  * la stratégie de résolution adoptée.

---

## Compétences évaluées

* Compréhension des transactions et des verrous
* Analyse des problèmes de concurrence
* Capacité à reproduire et diagnostiquer un deadlock
* Mise en œuvre de bonnes pratiques de conception concurrente
* Raisonnement “production ready” en génie logiciel
