# 🤖 Agent SRE Autonome — Zabbix + n8n + IA

> Détection → Diagnostic IA → Ticket GLPI → Auto-remédiation. Cycle complet, zéro intervention manuelle.

![n8n](https://img.shields.io/badge/n8n-workflow-EA4B71?style=flat&logo=n8n&logoColor=white)
![Zabbix](https://img.shields.io/badge/Zabbix-monitoring-CC0000?style=flat)
![Prometheus](https://img.shields.io/badge/Prometheus-metrics-E6522C?style=flat&logo=prometheus&logoColor=white)
![Loki](https://img.shields.io/badge/Loki-logs-F5A623?style=flat)
![GLPI](https://img.shields.io/badge/GLPI-ITSM-0078D4?style=flat)
![Gemini](https://img.shields.io/badge/Google-Gemini-4285F4?style=flat&logo=google&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green?style=flat)

---

## Description

Ce workflow n8n automatise le cycle de vie complet d'un incident infrastructure, de la détection à la clôture, sans intervention humaine.

**Le problème résolu :** Les équipes SRE passent trop de temps à créer manuellement des tickets, copier-coller des métriques et chercher les logs au moment d'une alerte. Ce workflow fait tout ça automatiquement, en quelques secondes.

---

## Flux de traitement

```
Zabbix (alerte)
    │
    ▼
Webhook n8n
    │
    ├─── PROBLEM ──► Récupération Token GLPI
    │                    │
    │               Recherche hôte GLPI
    │                    │
    │         ┌──────────┼──────────┐
    │         ▼          ▼          ▼
    │      Loki       Zabbix    Prometheus
    │     (logs)    (métriques) (métriques)
    │         └──────────┼──────────┘
    │                    ▼
    │              Agent IA (Gemini)
    │              Diagnostic Root Cause
    │                    │
    │              Création Ticket GLPI
    │              (avec diagnostic IA inclus)
    │                    │
    │              Acquittement Zabbix
    │              + N° ticket en commentaire
    │
    └─── RESOLVED ► Recherche ticket GLPI
                         │
                    Ajout solution
                         │
                    Clôture automatique
```

---

## Stack technique

| Composant | Rôle |
|-----------|------|
| **Zabbix** | Supervision et déclenchement des alertes via webhook |
| **n8n** | Orchestration du workflow |
| **Loki** | Récupération des logs système (5 dernières minutes) |
| **Prometheus + cAdvisor** | Métriques système et conteneurs Docker |
| **Google Gemini** | Analyse IA et diagnostic root cause |
| **GLPI** | Création, mise à jour et clôture des tickets ITSM |

---

## Ce que fait le workflow

**Sur PROBLEM :**
- Récupère les logs Loki des 5 dernières minutes
- Récupère les métriques Zabbix de l'hôte concerné
- Récupère les métriques Prometheus (CPU, RAM, disque, Top 3 conteneurs)
- Soumet tout à l'IA pour un diagnostic root cause
- Crée un ticket GLPI avec le diagnostic rédigé automatiquement
- Acquitte l'événement Zabbix et pose le numéro de ticket en commentaire

**Sur RESOLVED :**
- Recherche le ticket GLPI correspondant via l'event_id
- Ajoute une solution de clôture
- Ferme le ticket automatiquement

---

## Prérequis

- n8n >= 1.0
- Zabbix >= 6.0 avec webhook configuré
- GLPI avec API REST activée
- Loki accessible via HTTP
- Prometheus + Node Exporter + cAdvisor
- Clé API Google Gemini

---

## Installation

### 1. Importer le workflow

Dans n8n : **Settings → Import workflow** → sélectionner `workflow.json`

### 2. Configurer les credentials n8n

| Credential | Type |
|-----------|------|
| `Header - Zabbix Token` | HTTP Header Auth |
| `Zabbix account` | Zabbix API |
| `Google Gemini(PaLM) Api account` | Google PaLM API |

### 3. Remplacer les placeholders

| Placeholder | Valeur |
|------------|--------|
| `YOUR_GLPI_HOST` | IP ou FQDN de votre serveur GLPI |
| `YOUR_GLPI_APP_TOKEN` | App-Token GLPI |
| `YOUR_GLPI_USER_TOKEN` | User-Token GLPI |
| `YOUR_ZABBIX_HOST` | IP ou FQDN de votre serveur Zabbix |
| `YOUR_PROMETHEUS_HOST` | IP ou FQDN de votre serveur Prometheus |
| `YOUR_LOKI_HOST` | IP ou FQDN de votre serveur Loki |
| `YOUR-WEBHOOK-UUID` | UUID généré automatiquement par n8n à l'activation |

### 4. Configurer le webhook Zabbix

URL du webhook : `https://YOUR_N8N_HOST/webhook/YOUR-WEBHOOK-UUID`

Paramètres à passer dans le webhook Zabbix :

```
alert         → {TRIGGER.NAME}
severity      → {TRIGGER.SEVERITY}
host          → {HOST.NAME}
host_id       → {HOST.ID}
ip            → {HOST.IP}
event_id      → {EVENT.ID}
status        → {EVENT.STATUS}
zabbix_url    → {$ZABBIX.URL}/tr_events.php?triggerid={TRIGGER.ID}&eventid={EVENT.ID}
```

---

## Notes

- Le workflow est livré avec `"active": false` — à activer manuellement après configuration
- L'IA utilise Gemini avec `temperature: 0.1` pour des diagnostics déterministes
- Testé sur Zabbix 7.4 et GLPI 10.x

---

## Licence

MIT — Libre d'utilisation, modification et distribution.

---

*Partagé par [Xavier Maugendre](https://www.linkedin.com/in/xmaugendre/) — Expert Observabilité & SRE*
