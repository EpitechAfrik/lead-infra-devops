# Scénario d'Incident - Exercice 10

## 📋 Contexte de l'incident

**Date** : 2024-01-15  
**Heure de début** : 14:32 UTC  
**Heure de fin** : 15:17 UTC  
**Durée totale** : 45 minutes  
**Sévérité** : SEV-2 (Service Degraded)

---

## 🚨 Alerte initiale

```
[ALERT] API Latency Critical
Time: 14:32 UTC
Metric: api_request_duration_p99 > 5000ms
Threshold: 500ms (SLO breach)
Current value: 5847ms
Duration: 5 minutes
```

---

## 📊 Métriques observées

### Au moment de l'alerte (14:32)
- **Latency p50** : 450ms (normal: 80ms)
- **Latency p95** : 2300ms (normal: 150ms)
- **Latency p99** : 5847ms (normal: 250ms)
- **Error rate** : 8.5% (normal: 0.1%)
- **Request rate** : 850 req/s (normal: 900 req/s)
- **Timeout rate** : 30% des requêtes
- **Database connections** : 95/100 (pool saturé)

### Évolution pendant l'incident
- **14:35** : Error rate monte à 15%
- **14:40** : Certains utilisateurs signalent des timeouts
- **14:45** : Database connections à 100/100 (saturation complète)
- **14:50** : Début de la résolution
- **15:10** : Retour à la normale progressif
- **15:17** : Métriques stabilisées

---

## 🔍 Logs pertinents

### Application logs
```
[2024-01-15 14:32:15] ERROR: Database query timeout after 30s
Query: SELECT * FROM users WHERE email = ? AND status = 'active'
Execution time: 30000ms

[2024-01-15 14:33:42] ERROR: Connection pool exhausted
Available: 0/100
Waiting: 47 requests

[2024-01-15 14:35:18] WARN: Slow query detected
Query: SELECT orders.* FROM orders JOIN users ON orders.user_id = users.id WHERE users.created_at > ?
Execution time: 8500ms
Rows scanned: 2,450,000
```

### Database logs
```
[2024-01-15 14:31:00] INFO: Migration applied: 20240115_add_user_preferences
[2024-01-15 14:31:05] INFO: Migration completed successfully

[2024-01-15 14:32:10] WARN: Sequential scan on table 'users' (2.5M rows)
[2024-01-15 14:32:10] WARN: No index found for column 'status'

[2024-01-15 14:35:00] ERROR: Lock wait timeout exceeded
[2024-01-15 14:35:00] ERROR: Too many connections (100/100)
```

---

## 🎯 Cause racine (Root Cause)

### Cause immédiate
Migration de base de données déployée à 14:31 UTC qui a ajouté une nouvelle colonne `user_preferences` à la table `users` sans créer l'index nécessaire sur la colonne `status` qui est utilisée dans de nombreuses requêtes.

### Causes contributives
1. **Missing index** : La colonne `users.status` n'avait pas d'index, causant des sequential scans sur 2.5M de lignes
2. **Migration non testée en staging** : La migration a été déployée directement en production sans test de performance
3. **Pas de query analysis** : Aucune analyse EXPLAIN n'a été faite avant le déploiement
4. **Connection pool sous-dimensionné** : Pool de 100 connexions insuffisant pour absorber les requêtes lentes
5. **Monitoring incomplet** : Pas d'alerte sur les slow queries avant saturation

### Chronologie détaillée
1. **14:31** : Déploiement de la migration `20240115_add_user_preferences`
2. **14:31-14:32** : Les requêtes commencent à ralentir progressivement
3. **14:32** : Première alerte de latency (p99 > 5s)
4. **14:33** : Connection pool commence à saturer
5. **14:35** : Error rate augmente, utilisateurs impactés
6. **14:40** : Équipe identifie le problème (missing index)
7. **14:50** : Création de l'index en cours (opération longue sur 2.5M lignes)
8. **15:10** : Index créé, performances reviennent progressivement
9. **15:17** : Retour complet à la normale

---

## 💥 Impact business

### Utilisateurs affectés
- **Total users impacted** : ~12,000 utilisateurs (30% de la base active)
- **Failed requests** : ~22,500 requêtes en erreur
- **Timeout requests** : ~67,500 requêtes en timeout

### Impact financier estimé
- **Revenus perdus** : ~8,500€ (transactions non complétées)
- **SLA credits** : ~2,300€ (compensation clients enterprise)
- **Coût engineering** : ~1,200€ (3 engineers × 45min × taux horaire)
- **Total** : ~12,000€

### Impact SLO
- **Availability SLO** : 99.9% → 99.87% (breach de 0.03%)
- **Latency SLO** : p99 < 500ms → Breach pendant 45min
- **Error budget** : 43.2 minutes consommées sur 43.8 minutes mensuelles (98.6% du budget)

---

## ✅ Résolution appliquée

### Actions immédiates (14:40-15:10)
1. Identification du problème via logs et query analysis
2. Création de l'index manquant :
   ```sql
   CREATE INDEX CONCURRENTLY idx_users_status ON users(status);
   ```
3. Monitoring de la création de l'index (30 minutes)
4. Validation du retour à la normale

### Actions de mitigation
- Pas de rollback nécessaire (fix forward plus rapide)
- Communication aux clients via status page
- Monitoring renforcé pendant 2h post-incident

---

## 📝 Votre mission

En tant que Senior DevOps, vous devez :

1. **Rédiger un post-mortem complet** (`post-mortem-2024-01-15.md`)
   - Timeline détaillée
   - Root Cause Analysis (5 Whys ou Fishbone)
   - Impact quantifié
   - Action items avec responsables et deadlines

2. **Créer un runbook** (`runbooks/high-latency-response.md`)
   - Symptômes à surveiller
   - Étapes de diagnostic
   - Procédure de résolution
   - Escalation path

3. **Améliorer le monitoring**
   - Ajouter alertes manquantes (slow queries, missing indexes, etc.)
   - Dashboard Grafana pour ce type d'incident
   - Script de diagnostic automatique

4. **Définir les actions préventives**
   - Process de review des migrations
   - Tests de performance obligatoires
   - Amélioration du CI/CD
   - Formation de l'équipe

5. **Bonus : Simuler l'incident**
   - Script qui reproduit le problème
   - Démontrer la détection et résolution

---

## 🎓 Critères d'évaluation

- **Analyse** : Profondeur de la RCA, identification des causes multiples
- **Pragmatisme** : Actions concrètes et réalistes
- **Communication** : Clarté du post-mortem (blameless culture)
- **Prévention** : Qualité des mesures préventives proposées
- **Automatisation** : Scripts et outils pour éviter la récurrence
