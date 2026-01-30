# Gestion des exceptions dans les fonctions PL/pgSQL

La gestion des erreurs est un aspect fondamental dans le développement de fonctions et procédures stockées en PostgreSQL. PL/pgSQL offre un mécanisme robuste de traitement d’exceptions permettant d’anticiper, capturer et traiter les erreurs pour éviter des plantages ou comportements indésirables.

---

## 1. Principe de base : le bloc `EXCEPTION`

Une section `EXCEPTION` peut être ajoutée à la fin d’un bloc `BEGIN ... END` pour capturer des erreurs qui se produisent dans ce bloc. Cela permet de gérer différentes erreurs par type, ou bien d’exécuter du code alternatif en cas d’échec.

### Syntaxe simplifiée

```plpgsql
BEGIN
  -- code pouvant générer une erreur
EXCEPTION
  WHEN exception_name THEN
    -- gestion spécifique
  WHEN OTHERS THEN
    -- gestion générale
END;
```

---

## 2. Exemples courants

### Gestion d’une division par zéro

```sql
CREATE OR REPLACE FUNCTION safe_division(a NUMERIC, b NUMERIC)
RETURNS NUMERIC AS $$
BEGIN
  RETURN a / b;
EXCEPTION
  WHEN division_by_zero THEN
    RETURN NULL;  -- éviter l’erreur fatale, retourner NULL
END;
$$ LANGUAGE plpgsql;
```

### Capturer un conflit unique (violation de contrainte unique)

```sql
BEGIN
  INSERT INTO utilisateurs(id, email) VALUES (1, 'exemple@example.com');
EXCEPTION
  WHEN unique_violation THEN
    RAISE NOTICE 'Email déjà utilisé.';
END;
```

---

## 3. Types d’exceptions prédéfinies et personnalisées

PostgreSQL dispose d’une riche liste de conditions d’erreur nommées (ex: `division_by_zero`, `unique_violation`, `foreign_key_violation`) qu’il est possible de gérer directement.

Pour capturer toute erreur non gérée, utiliser le `WHEN OTHERS` qui fonctionne comme un "catch all".

---

## 4. Reraise d’exception

Il est possible de relancer une erreur détectée :

```plpgsql
EXCEPTION
  WHEN unique_violation THEN
    RAISE NOTICE 'Conflit détecté';
    RAISE; -- relance l’exception originale
```

Cela permet de loguer ou effectuer un traitement avant de transmettre l’erreur.

---

## 5. Exemple complet : fonction avec gestion avancée d’exceptions

```plpgsql
CREATE OR REPLACE FUNCTION insert_utilisateur(p_email TEXT)
RETURNS TEXT AS $$
BEGIN
  INSERT INTO utilisateurs(email) VALUES (p_email);
  RETURN 'Insertion réussie';
EXCEPTION
  WHEN unique_violation THEN
    RETURN 'Erreur : email déjà utilisé';
  WHEN foreign_key_violation THEN
    RETURN 'Erreur : clé étrangère invalide';
  WHEN OTHERS THEN
    RETURN 'Erreur inconnue, contacter l’administrateur';
END;
$$ LANGUAGE plpgsql;
```

---

## 6. Diagramme Mermaid : schéma de gestion d’exceptions

```mermaid
flowchart TD
    Start((Début))
    TryBloc[Bloc TRY (BEGIN)]
    ErrorCond{Erreur détectée ?}
    HandleUnique["Gérer unique_violation"]
    HandleFK["Gérer foreign_key_violation"]
    HandleOthers["Gérer autres erreurs"]
    Success[Retourner succès]
    End((Fin))

    Start --> TryBloc --> ErrorCond
    ErrorCond -- Oui --> HandleUnique --> End
    ErrorCond -- Oui --> HandleFK --> End
    ErrorCond -- Oui --> HandleOthers --> End
    ErrorCond -- Non --> Success --> End
```

---

## 7. Bonnes pratiques

- Toujours anticiper les erreurs courantes selon le contexte métier (violations de contraintes, erreurs de conversion, etc.).  
- Utiliser `WHEN OTHERS` pour capturer les erreurs inattendues, au moins pour loguer.  
- Ne pas cacher les erreurs critiques : parfois il vaut mieux les relancer.  
- Documenter clairement les erreurs gérées pour faciliter l’utilisation des fonctions.

---

## 8. Sources et références

- [PostgreSQL Documentation - PL/pgSQL - Exception Handling](https://www.postgresql.org/docs/current/plpgsql-control-structures.html#PLPGSQL-EXCEPTION-CLAUSE)  
- [Cybertec PostgreSQL - Exception Handling in PL/pgSQL](https://www.cybertec-postgresql.com/en/plpgsql-exception-handling-in-postgresql/)  
- [PostgreSQL Tutorial - PL/pgSQL Exception Handling](https://www.postgresqltutorial.com/plpgsql-exception-handling/)  
- [Official PostgreSQL Error Codes](https://www.postgresql.org/docs/current/errcodes-appendix.html)  

---

La gestion des exceptions dans PL/pgSQL améliore la robustesse des fonctions en encadrant les erreurs, ce qui garantit des réponses maîtrisées, évite des blocages inattendus, et facilite le débogage et la maintenance.