# Test Technique – DevOps (Sans serveur réel)

## 🎯 Objectif
Évaluer la capacité du candidat à automatiser build, tests, packaging, déploiement simulé et monitoring sans accès à des serveurs persistants.

---

## 🚩 Contexte
Tu dois prendre un projet "starter" (fourni) et construire une **chaîne CI/CD** automatisée, des artefacts dockerisés, une stratégie de déploiement simulé et une couche d'observabilité. Tout doit être exécutable dans un environnement local (Windows + Docker Desktop).

---

## 🧩 Exercice 1 — Dockerisation complète
- Dockeriser l'application (multi-stage si nécessaire)
- Fournir `docker-compose.yml` qui lance :
  - app (API)
  - DB (Postgres ou MySQL)
- S'assurer que :
  - Les volumes persistants sont configurés
  - Les `.env` ne sont pas commit
  - Un fichier `.env.example` documente les variables nécessaires

**Critères :**
- Images légères et multi-stage
- Bonne séparation des responsabilités
- Persistance fonctionnelle

---

## 🔁 Exercice 2 — CI (GitHub Actions)
- Créer un workflow `ci.yml` qui :
  - Lint le code
  - Lance les tests unitaires
  - Build les images Docker
  - Scanne les vulnérabilités avec Trivy
- Le workflow doit être déclenchable sur PR

**Critères :**
- Étapes claires, jobs parallélisables
- Gestion sécurisée des secrets

---

## 🚀 Exercice 3 — CD (simulé)
Script `scripts/deploy_local.sh` qui :
- Build les images
- Lance `docker-compose -f deploy/docker-compose.yml up -d`
- Copie les artefacts dans `deploy/releases/<timestamp>/`
- Met à jour `deploy/current` (symlink ou fichier texte sous Windows)
- Génère un manifeste `deploy/releases/<timestamp>/manifest.json` avec metadata

Fournir un script `scripts/rollback.sh` qui rétablit vers la release précédente.

**Critères :**
- Automatisation reproducible
- Rollback simple et testé

---

## 📈 Exercice 4 — Observabilité & Alerting
- Fournir un docker-compose qui démarre Prometheus + Grafana
- Dashboard Grafana minimal (CPU, Memory, uptime, endpoint health)
- Décrire dans `README.md` les règles d'alerte critiques

**Critères :**
- Dashboard minimal fonctionnel
- Instructions claires

---

## ❓ Exercice 5 — Questions (answers.md)
Répondre dans `answers.md` aux questions :

1. Comment gérez-vous les secrets en production ? Expliquer la stratégie choisie.
2. Décrire une procédure de rollback en cas de déploiement défectueux.
3. Comment monitoreriez-vous une augmentation soudaine du 500 Errors ?

---

## 📋 Livrables attendus

```
.
├── .github/workflows/
│   └── ci.yml
├── deploy/
│   ├── docker-compose.yml
│   ├── current (symlink ou .txt)
│   └── releases/<timestamp>/manifest.json
├── scripts/
│   ├── deploy_local.sh
│   └── rollback.sh
├── docker-compose.yml
├── Dockerfile
├── .env.example
├── README.md
└── answers.md
```

---

## ⏱ Temps recommandé
- **Exercices 1-5** : Obligatoires

---

## 📊 Barème

- Dockerisation & persistance : 25%
- CI (qualité des workflows) : 30%
- CD simulé (manifests, rollback) : 20%
- Observabilité & alerting : 15%
- Documentation & réponses : 10%

**Critères transversaux :**
- Qualité du code
- Sécurité (pas de secrets)
- Documentation claire
- Git workflow propre

---

## 🖥️ Notes Windows + Docker Desktop

### Symlinks (pour deploy/current)
Sous Windows, les symlinks nécessitent des droits admin. Utilisez plutôt un fichier texte :

```bash
# Au lieu de : ln -sfn "releases/${TIMESTAMP}" "./deploy/current"
# Faire : 
echo "${TIMESTAMP}" > "./deploy/current.txt"
```

Pour lire la release actuelle :
```bash
CURRENT=$(cat ./deploy/current.txt)
```

## 🚀 Soumission
1. Fork le repo
2. Branche `develop`
3. Commits réguliers
4. PR avec titre `[DevOps] Prénom Nom`

---

### Exercices bonus (optionnels)

Si vous avez du temps supplémentaire :

### Exercice 6 — Multi-environnements
- Configs pour dev/staging/prod
- Feature flags par environnement

### Exercice 7 — Sécurité avancée
- SAST (Semgrep)
- SBOM avec Syft
- Secrets chiffrés (SOPS)

### Exercice 8 — Observabilité avancée
- Stack complète (Loki + Jaeger)
- SLI/SLO définis
- Dashboards avancés

### Exercice 9 — Incident Response
- Post-mortem d'incident (scénario fourni dans `incidents/scenario.md`)
- Runbooks opérationnels

## 📞 Support Technique

En cas de blocage :
- Documenter le problème dans votre README
- Proposer une solution alternative
- Continuer sur les autres exercices

**La capacité à gérer les imprévus fait partie de l'évaluation.**

---

**Bonne chance ! 🚀**

*Ce test évalue votre capacité à livrer rapidement une infrastructure fonctionnelle tout en démontrant une vision stratégique.*
