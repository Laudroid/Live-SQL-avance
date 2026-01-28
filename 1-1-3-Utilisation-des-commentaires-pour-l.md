# Normes de codage SQL : Utilisation des commentaires pour la documentation  

Les commentaires sont un outil puissant pour documenter les requêtes SQL, expliciter les parties complexes, et faciliter la collaboration. Une utilisation structurée et minimaliste des commentaires rehausse la lisibilité sans alourdir le code.

---

## 1. Pourquoi commenter son SQL ?

- **Clarifier le raisonnement** derrière une requête complexe.
- **Informer sur le contexte** des choix techniques (ex : raisons d’une jointure spécifique ou filtre).
- **Documenter les modifications** (historique, dates, auteurs).
- **Faciliter la maintenance** et la relecture par d’autres développeurs.

---

## 2. Bonnes pratiques pour commenter du code SQL

### a) Commentaires concis et ciblés  
Prioriser la clarté : un commentaire doit expliquer *pourquoi*, pas *comment* (car le code montre *comment*).

### b) Positionner les commentaires au bon endroit  
- **Avant un bloc** expliquant sa finalité.  
- **Sur une ligne** pour une ligne spécifique lorsque nécessaire.

### c) Utiliser les formats standards

- **Commentaires sur une ligne** : `-- Commentaire`  
- **Commentaires multi-lignes** :  
```sql
/*  
   Explication détaillée  
   sur plusieurs lignes  
*/
```

---

## 3. Exemple illustratif 

```sql
-- Sélection des clients actifs avec leurs commandes de 2024
SELECT cust.customer_name,
       ord.order_id,
       ord.order_date
FROM customers cust
JOIN orders ord
    ON cust.customer_id = ord.customer_id
-- Filtrage des commandes à partir de 2024-01-01
WHERE ord.order_date >= '2024-01-01'
  AND cust.status = 'active';
```

---

## 4. Quand préférer les commentaires multi-lignes  

Lorsque le contexte est complexe ou l’explication nécessite plusieurs lignes :

```sql
/*
Calcul du chiffre d'affaires total par client.
La somme porte uniquement sur les commandes 
livrées (status = 'delivered').
*/
SELECT cust.customer_id,
       SUM(ord.amount) AS total_revenue
FROM customers cust
JOIN orders ord
    ON cust.customer_id = ord.customer_id
WHERE ord.status = 'delivered'
GROUP BY cust.customer_id;
```

---

## 5. Éviter les pièges fréquents

| Pratique à éviter                                      | Pourquoi ?                                         |
|--------------------------------------------------------|---------------------------------------------------|
| Commenter des évidences                               | Alourdit le code sans valeur ajoutée              |
| Multiplication excessive de commentaires              | Diminue la lisibilité                              |
| Commentaires obsolètes qui ne reflètent plus le code  | Génère confusion                                  |

---

## 6. Bonus : Documenter l’évolution des requêtes  

Un simple champ dans les commentaires peut préserver un historique léger :  

```sql
/*
2024-04-15, J. Dupont :
Ajout du filtre sur le statut 'active' des clients
pour exclure les inactifs.
*/
```

---

## 7. Diagramme Mermaid : organisation optimale des commentaires dans une requête

```mermaid
flowchart LR
    C[Commentaire général sur la requête] --> S[SELECT]
    S --> F[FROM]
    F --> J[JOIN avec explication si nécessaire]
    J --> W[WHERE avec commentaire ligne par ligne]
    W --> G[GROUP BY / HAVING]
    G --> O[ORDER BY]
```

---

## 8. Sources fiables et à jour

- [SQL Style Guide - Simon Holywell (2023)](https://www.sqlstyle.guide/#comments)  
- [Redgate - Best practices for commenting SQL](https://www.red-gate.com/hub/product-learning/sql-prompt/commenting-sql-queries)  
- [Mode Analytics - How to write good SQL comments](https://mode.com/sql-tutorial/sql-comments/#best-practices-for-sql-comments)  
- [Stack Overflow - How to comment SQL effectively](https://stackoverflow.com/questions/4193010/what-is-the-best-way-to-comment-sql-code)

---

## Conclusion

Des commentaires ciblés et bien positionnés sont un élément clé pour rendre le SQL accessible et évolutif. Ils complètent une bonne indentation et une nomination explicite pour produire un code propre, transparent et efficace.