# Automatisation de tâches récurrentes avec fonctions et procédures stockées en PostgreSQL

Automatiser des opérations régulières au sein de la base de données améliore la performance et la fiabilité des systèmes. PostgreSQL, grâce à ses fonctions et procédures stockées, offre un cadre efficace pour gérer ces tâches récurrentes sans intervention manuelle.

---

## 1. Pourquoi automatiser avec des fonctions/procédures ?

- Exécuter des opérations périodiques (nettoyage, calcul, mise à jour)  
- Assurer la cohérence des données via des règles d’affaires emballées dans la base  
- Décharger la couche applicative  
- Minimiser les erreurs humaines tout en optimisant la performance

---

## 2. Exemple : mise à jour automatique des statuts

Imaginons une table `commandes` dont le statut doit passer à `expiré` si la date de livraison prévue est dépassée.

### Fonction PL/pgSQL pour mise à jour des commandes expirées

```sql
CREATE OR REPLACE FUNCTION maj_commandes_expirees()
RETURNS VOID AS $$
BEGIN
  UPDATE commandes
  SET statut = 'expiré'
  WHERE statut = 'en attente'
    AND date_livraison <= CURRENT_DATE;
END;
$$ LANGUAGE plpgsql;
```

---

## 3. Programmation de la tâche avec `pg_cron` ou `pgAgent`

PostgreSQL ne gère pas nativement les tâches planifiées internes, mais plusieurs extensions comme `pg_cron` ou `pgAgent` permettent de lancer des fonctions périodiquement.

### Exemple d’utilisation `pg_cron` pour exécuter la fonction chaque nuit à 2h

```sql
SELECT cron.schedule('0 2 * * *', 'SELECT maj_commandes_expirees();');
```

---

## 4. Exemple avancé : purge des anciennes entrées de logs

```sql
CREATE OR REPLACE FUNCTION purge_logs(anciennete INT)
RETURNS VOID AS $$
BEGIN
  DELETE FROM journalisation
  WHERE date_evenement < NOW() - INTERVAL '1 day' * anciennete;
END;
$$ LANGUAGE plpgsql;
```

Cette fonction peut être appelée régulièrement pour maintenir la taille raisonnable d’une table de logs.

---

## 5. Exemple pratique avec procédure et paramètres INOUT

Une procédure met à jour des soldes et retourne un message via paramètre INOUT.

```sql
CREATE OR REPLACE PROCEDURE maj_solde_compte(
  id_compte INT,
  montant NUMERIC,
  INOUT message TEXT
)
LANGUAGE plpgsql AS $$
BEGIN
  UPDATE comptes SET solde = solde + montant WHERE id = id_compte;

  IF FOUND THEN
    message := 'Solde mis à jour avec succès';
  ELSE
    message := 'Compte non trouvé';
  END IF;
END;
$$;
```

---

## 6. Schéma Mermaid : workflow d’automatisation d’une tâche récurrente

```mermaid
flowchart TD
    Start((Début))
    Timer[Événement Horaire (cron)]
    CallFunc[Appel fonction/procédure]
    Traitement[Mise à jour / Nettoyage]
    Vérification{Succès ?}
    LogSuccess[Log succès]
    LogError[Log erreur et alerte]
    End((Fin))

    Start --> Timer --> CallFunc --> Traitement --> Vérification
    Vérification -- Oui --> LogSuccess --> End
    Vérification -- Non --> LogError --> End
```

---

## 7. Bonnes pratiques

- Externaliser la planification avec des outils dédiés (`pg_cron`, `pgAgent`), éviter des procédures en boucle infinie.  
- Rendre les fonctions/procédures idempotentes (réexécutables sans effets indésirables).  
- Utiliser des logs internes ou notifications pour suivre les exécutions.  
- Documenter clairement le rôle et la fréquence d’exécution des routines.

---

## 8. Sources et références

- [PostgreSQL Documentation - pgAgent](https://www.pgadmin.org/docs/pgadmin4/latest/pgagent.html)  
- [pg_cron GitHub repository](https://github.com/citusdata/pg_cron)  
- [PostgreSQL Documentation - PL/pgSQL](https://www.postgresql.org/docs/current/plpgsql.html)  
- [Cybertec PostgreSQL - Automatiser des tâches dans PostgreSQL](https://www.cybertec-postgresql.com/en/automate-tasks-or-schedules-in-postgresql-with-pgagent/)  

---

L’automatisation via fonctions et procédures stockées facilite la gestion des opérations récurrentes en garantissant leur exécution régulière, optimisée, et supervisée, donnant ainsi plus de fiabilité et de maintenabilité aux bases PostgreSQL.