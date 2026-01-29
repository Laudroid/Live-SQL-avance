
# TP SQL – Common Table Expressions récursives et parcours hiérarchique

## Contexte pédagogique

De nombreux systèmes d’information manipulent des **structures hiérarchiques** :
organigrammes d’entreprise, catégories imbriquées, commentaires, arbres de fichiers, etc.

Le SQL moderne propose un mécanisme puissant pour traiter ce type de données : les **Common Table Expressions récursives (CTE récursives)**.

Dans ce TP, vous allez apprendre à :

* parcourir une hiérarchie stockée en base relationnelle,
* comprendre le fonctionnement d’une récursion en SQL,
* enrichir progressivement les résultats calculés à chaque niveau de la hiérarchie.

L’objectif n’est pas seulement d’obtenir un résultat, mais de **maîtriser la logique de récursion**, compétence clé en génie logiciel.

---

## Schéma de données

On considère la table suivante :

### **Employes**

| Colonne   | Description                                                                                |
| --------- | ------------------------------------------------------------------------------------------ |
| EmployeID | Identifiant unique de l’employé                                                            |
| Nom       | Nom de l’employé                                                                           |
| Prenom    | Prénom de l’employé                                                                        |
| Titre     | Poste occupé                                                                               |
| ManagerID | Identifiant du manager direct (clé étrangère vers Employes.EmployeID, NULL pour la racine) |

La hiérarchie est représentée par une **relation réflexive** :
un employé peut être le manager d’un autre employé.

---

## Objectif fonctionnel

À partir de l’identifiant d’un manager donné :

1. Retrouver **tous ses subordonnés**, qu’ils soient :

   * directs
   * ou indirects (subordonnés de subordonnés)
2. Calculer pour chaque subordonné :

   * son **niveau hiérarchique** par rapport au manager de départ
3. (Variante avancée) Construire le **chemin hiérarchique complet** depuis le manager initial jusqu’au subordonné.

---

## Travail demandé

### Partie 1 – Comprendre la hiérarchie et la récursion

1. Analysez la structure de la table `Employes`.
2. Expliquez (à l’écrit) pourquoi une requête SQL classique avec de simples `JOIN` n’est **pas suffisante** pour parcourir une hiérarchie de profondeur inconnue.
3. Rappelez le principe général d’une **récursion** :

   * condition d’arrêt
   * répétition basée sur un état précédent

---

### Partie 2 – CTE récursif : parcours simple avec niveau hiérarchique

Dans cette partie, vous devez écrire une requête SQL utilisant un **CTE récursif** afin de parcourir la hiérarchie des employés.

#### Consignes

1. Créez un CTE récursif nommé de manière explicite.
2. Le CTE devra contenir :

   * un **membre d’ancrage** :

     * correspondant au manager de départ (par exemple `EmployeID = 2`)
     * avec un niveau hiérarchique initial fixé à 0
   * un **membre récursif** :

     * permettant de retrouver les subordonnés des employés trouvés à l’itération précédente
     * incrémentant le niveau hiérarchique à chaque étape
3. La requête finale devra :

   * afficher uniquement les subordonnés (exclure le manager de départ)
   * afficher pour chaque subordonné :

     * son identifiant
     * son nom complet
     * son titre
     * son niveau hiérarchique
   * trier les résultats par niveau puis par identifiant

Objectif pédagogique :
comprendre **le rôle du membre d’ancrage, du membre récursif et du `UNION ALL`**.

---

### Partie 3 – Enrichissement : construction du chemin hiérarchique

Vous allez maintenant proposer une **version enrichie** de la requête précédente.

#### Consignes

1. Reprenez le CTE récursif précédent.
2. Ajoutez une colonne permettant de stocker le **chemin hiérarchique complet** :

   * le chemin commence par le nom du manager de départ
   * à chaque itération, le nom du subordonné est ajouté au chemin
3. Le chemin devra être lisible, par exemple :

   ```
   Alice Dupont -> Jean Martin -> Sophie Bernard
   ```
4. La requête finale affichera :

   * les mêmes informations que précédemment
   * le chemin hiérarchique complet

Objectif pédagogique :
montrer que les CTE récursifs peuvent **accumuler des données** au fil de la récursion, pas seulement parcourir des lignes.
---

## Contraintes techniques

* Vous devez utiliser la syntaxe `WITH RECURSIVE`.
* La requête doit être :

  * correctement indentée
  * commentée
  * compréhensible sans explication orale
* La concaténation de chaînes devra être adaptée au SGBD utilisé
  (PostgreSQL / SQLite / MySQL / SQL Server).

---

## Livrables attendus

* Un fichier `.sql` contenant :

  * la requête avec parcours simple
  * la requête avec chemin hiérarchique
* Un court document (ou commentaires intégrés) répondant aux questions de réflexion

---

## Compétences évaluées

* Compréhension des structures hiérarchiques en base relationnelle
* Maîtrise des CTE récursifs
* Raisonnement algorithmique appliqué au SQL
* Qualité de structuration et de documentation du code
* Capacité à enrichir progressivement une solution existante


