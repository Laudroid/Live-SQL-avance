# SQL CTF – Forensic Data Challenge : *Operation Inside Job*

## Format du challenge

Ce TP prend la forme d’un **CTF (Capture The Flag) SQL**, orienté **forensic data & investigation interne**.

Vous n’êtes plus de simples analystes : vous êtes une **cellule d’intervention post-incident** chargée d’identifier l’auteur d’une fuite de données à partir d’une base PostgreSQL compromise.

Chaque étape validée vous rapproche du **FLAG final : l’identité du coupable**.

---

## Scénario

> *« Les données ont été volées. Les accès étaient légitimes. Le coupable est forcément l’un des nôtres. »*

Une entreprise partenaire a subi une **exfiltration complète de sa base employés**.
Les audits réseau n’ont révélé aucune intrusion externe.

L’hypothèse principale est désormais : **menace interne**.

Vous disposez d’un **dump SQL** issu de la base RH.

À vous de reconstituer les faits.

---

## Profil du suspect (briefing initial)

Les enquêteurs ont établi un **profil partiel** du suspect :

* A quitté l’entreprise **après juillet 2002**
* S’est plaint d’une **baisse de salaire récente**
* Possède un **haut niveau technique** (capable de pirater une BD seul)
* Poste possible : **Senior Engineer**

Le suspect ne se dénoncera pas.
Les données, elles, ne mentent jamais.

---

## Règles du CTF

* ✅ **SQL uniquement** (PostgreSQL)
* ❌ Aucun script externe
* ❌ Aucune modification des données
* ✅ Toutes les preuves doivent venir de requêtes SQL

Chaque étape correspond à un **mini-flag logique**.

---

## FLAG 1 – Les fantômes de l’entreprise

**Objectif** :
Identifier les employés **ayant quitté l’entreprise**.

Indice narratif :

> *« Un employé qui n’est plus là laisse toujours une trace administrative. »*

Validation :

* Vous obtenez un **ensemble cohérent d’employés sortis**
* Leur **historique salarial** est exploitable

*Preuve attendue* : requête SQL

---

## FLAG 2 – Les derniers versements

**Objectif** :
Pour chaque ancien employé, retrouver **les deux derniers salaires perçus**.

Indice narratif :

> *« Les dernières décisions financières sont souvent les plus révélatrices. »*

Validation :

* Exactement **deux salaires par employé**
* Ordonnés dans le temps

*Preuve attendue* : requête SQL

---

## FLAG 3 – Le mobile

**Objectif** :
Comparer les deux derniers salaires afin de détecter :

* une **baisse de rémunération**
* sur une **période significative**

Indice narratif :

> *« Quand l’argent disparaît, la rancœur apparaît. »*

Validation :

* Les deux salaires doivent pouvoir être comparés sur **une même ligne**
* Une variation négative est identifiable

*Preuve attendue* : requête SQL

---

## FLAG 4 – Croisement des preuves

**Objectif final** :
Isoler l’unique employé qui :

* a quitté l’entreprise **après juillet 2002**
* a subi une **baisse de salaire** sur le dernier mois
* avait suffisamment de compétence pour exfiltrer un base de données

Indice narratif :

> *« La vérité n’apparaît que lorsque toutes les pièces du puzzle sont assemblées. »*

Validation :

🏴 **FLAG FINAL** :
> `FLAG{prenom_nom}`

---

## Livrables

Vous devez fournir :

1. Les **requêtes SQL** utilisées pour chaque flag
2. Une **requête finale consolidée**
3. Le **FLAG final** (nom du suspect)

---

## Bonus CTF (facultatif)

* Proposer des **contre-mesures techniques**
* Identifier les **faiblesses de gouvernance des accès**
* Expliquer comment un SIEM ou un audit SQL aurait pu aider

CTF terminé.

À vous de capturer le flag.
