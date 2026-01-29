Voici deux solutions pour le TP sur les CTE récursifs, chacune avec une approche légèrement différente pour vous aider à comprendre la flexibilité de cet outil.

---

### **Chapitre 1 : Parcours Simple de la Hiérarchie avec Niveau**

Cette première solution se concentre sur l'objectif principal : identifier tous les subordonnés directs et indirects d'un manager donné et leur attribuer un niveau hiérarchique. Elle utilise une structure de CTE récursif classique avec un membre d'ancrage et un membre récursif.


```sql
-- Solution 1: Parcours simple de la hiérarchie avec niveau
WITH RECURSIVE SubordonnesHierarchie AS (
    -- Membre d'ancrage : Sélectionne le manager de départ (Alice Dupont, EmployeID = 2)
    -- et initialise son niveau hiérarchique à 0.
    SELECT
        e.EmployeID,
        e.Nom,
        e.Prenom,
        e.Titre,
        e.ManagerID,
        0 AS NiveauHierarchique -- Le manager de départ est au niveau 0
    FROM
        Employes e
    WHERE
        e.EmployeID = 2 -- Cible le manager avec l'EmployeID 2 (Alice Dupont)

    UNION ALL

    -- Membre récursif : Joint la CTE elle-même (SubordonnesHierarchie) avec la table Employes
    -- pour trouver les subordonnés directs des employés de l'itération précédente.
    -- Le niveau hiérarchique est incrémenté à chaque étape.
    SELECT
        e_sub.EmployeID,
        e_sub.Nom,
        e_sub.Prenom,
        e_sub.Titre,
        e_sub.ManagerID,
        sh.NiveauHierarchique + 1 -- Incrémente le niveau pour chaque subordonné trouvé
    FROM
        Employes e_sub
    JOIN
        SubordonnesHierarchie sh ON e_sub.ManagerID = sh.EmployeID
)
-- Requête finale : Sélectionne les informations des subordonnés.
-- La clause WHERE NiveauHierarchique > 0 exclut le manager de départ lui-même du résultat final,
-- car nous cherchons uniquement ses subordonnés.
SELECT
    sh.EmployeID,
    sh.Prenom || ' ' || sh.Nom AS "Nom Complet", -- Concaténation du prénom et du nom
    sh.Titre,
    sh.NiveauHierarchique
FROM
    SubordonnesHierarchie sh
WHERE
    sh.NiveauHierarchique > 0
ORDER BY
    sh.NiveauHierarchique, sh.EmployeID; -- Ordonne par niveau puis par ID pour une meilleure lisibilité
```


**Explication :**

*   **`WITH RECURSIVE SubordonnesHierarchie AS (...)`** : Déclare un CTE récursif nommé `SubordonnesHierarchie`. Le mot-clé `RECURSIVE` est essentiel.
*   **Membre d'ancrage** : C'est le point de départ de la récursion. Ici, nous sélectionnons l'employé avec l'`EmployeID = 2` (Alice Dupont) et lui attribuons un `NiveauHierarchique` de 0.
*   **`UNION ALL`** : Combine le résultat du membre d'ancrage avec celui du membre récursif. `UNION ALL` est utilisé car nous ne nous soucions pas des doublons (chaque employé n'apparaît qu'une fois dans la hiérarchie à un niveau donné).
*   **Membre récursif** : Cette partie se réfère à `SubordonnesHierarchie` elle-même (`sh`). Elle joint la table `Employes` (`e_sub`) avec le résultat de l'itération précédente (`sh`) sur la condition `e_sub.ManagerID = sh.EmployeID`. Cela permet de trouver tous les employés dont le manager est l'un des employés trouvés à l'étape précédente. Le `NiveauHierarchique` est incrémenté de 1 à chaque nouvelle "génération" de subordonnés.
*   **Requête finale** : Sélectionne les colonnes demandées à partir du CTE `SubordonnesHierarchie`. La condition `WHERE sh.NiveauHierarchique > 0` est appliquée pour n'afficher que les subordonnés et non le manager initial.
*   **`||` pour concaténation** : Notez que `||` est utilisé pour concaténer les chaînes de caractères (`Prenom` et `Nom`). Cette syntaxe est courante dans PostgreSQL et SQLite. Pour MySQL, vous utiliseriez `CONCAT(sh.Prenom, ' ', sh.Nom)`, et pour SQL Server, `sh.Prenom + ' ' + sh.Nom`.

---

### **Chapitre 2 : Parcours de la Hiérarchie avec Chemin Complet**

Cette deuxième solution va un peu plus loin en intégrant la construction du chemin hiérarchique complet pour chaque subordonné, en plus de son niveau. Cela démontre comment les CTE récursifs peuvent être utilisés pour accumuler des informations au fur et à mesure de la traversée de la hiérarchie.


```sql
-- Solution 2: Parcours de la hiérarchie avec chemin complet et niveau
WITH RECURSIVE SubordonnesAvecChemin AS (
    -- Membre d'ancrage : Sélectionne le manager de départ, initialise son niveau à 0
    -- et crée le chemin hiérarchique initial avec son propre nom.
    SELECT
        e.EmployeID,
        e.Nom,
        e.Prenom,
        e.Titre,
        e.ManagerID,
        0 AS NiveauHierarchique,
        e.Prenom || ' ' || e.Nom AS CheminHierarchique -- Le chemin commence avec le manager initial
    FROM
        Employes e
    WHERE
        e.EmployeID = 2 -- Cible le manager avec l'EmployeID 2 (Alice Dupont)

    UNION ALL

    -- Membre récursif : Trouve les subordonnés, incrémente le niveau
    -- et ajoute le nom du subordonné au chemin hiérarchique existant.
    SELECT
        e_sub.EmployeID,
        e_sub.Nom,
        e_sub.Prenom,
        e_sub.Titre,
        e_sub.ManagerID,
        swc.NiveauHierarchique + 1,
        swc.CheminHierarchique || ' -> ' || e_sub.Prenom || ' ' || e_sub.Nom -- Ajoute le subordonné au chemin
    FROM
        Employes e_sub
    JOIN
        SubordonnesAvecChemin swc ON e_sub.ManagerID = swc.EmployeID
)
-- Requête finale : Sélectionne les informations des subordonnés, y compris le chemin complet.
-- Encore une fois, nous excluons le manager de départ pour n'afficher que les subordonnés.
SELECT
    swc.EmployeID,
    swc.Prenom || ' ' || swc.Nom AS "Nom Complet",
    swc.Titre,
    swc.NiveauHierarchique,
    swc.CheminHierarchique
FROM
    SubordonnesAvecChemin swc
WHERE
    swc.NiveauHierarchique > 0
ORDER BY
    swc.NiveauHierarchique, swc.EmployeID;
```


**Explication :**

*   **`WITH RECURSIVE SubordonnesAvecChemin AS (...)`** : Déclare le CTE récursif.
*   **Membre d'ancrage** : Similaire à la solution précédente, mais nous initialisons une nouvelle colonne `CheminHierarchique` avec le nom complet du manager de départ.
*   **Membre récursif** : Lors de chaque itération, en plus d'incrémenter le `NiveauHierarchique`, nous concaténons le nom complet du subordonné actuel au `CheminHierarchique` hérité de son manager (`swc.CheminHierarchique || ' -> ' || e_sub.Prenom || ' ' || e_sub.Nom`). Cela construit progressivement le chemin complet depuis le manager initial jusqu'au subordonné actuel.
*   **Requête finale** : Affiche toutes les colonnes, y compris le `CheminHierarchique` qui fournit une vue détaillée de la position de chaque subordonné dans l'organigramme.
*   **Adaptation de la concaténation** : Comme pour la solution 1, la syntaxe `||` pour la concaténation de chaînes est spécifique à certains SGBD (PostgreSQL, SQLite). N'oubliez pas d'adapter cette partie si vous utilisez MySQL (`CONCAT()`) ou SQL Server (`+`).

Ces deux solutions vous donnent une base solide pour travailler avec les CTE récursifs. La clé est de bien comprendre comment le membre d'ancrage initialise la récursion et comment le membre récursif s'appuie sur les résultats de l'itération précédente pour progresser dans la structure hiérarchique.