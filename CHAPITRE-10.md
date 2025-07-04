# CHAPITRE 10 – Projet Final : Déploiement et exploitation d’un honeypot Cowrie

---

## 1. Objectifs pédagogiques

À l’issue de ce projet, l’apprenant sera capable de :

* Déployer une instance complète et sécurisée de Cowrie
* Capturer et analyser des attaques en conditions réelles
* Extraire des IOCs et produire un rapport d’incident
* Intégrer les logs dans un outil de visualisation (ELK, Grafana, etc.)
* Mettre en œuvre des mesures de réaction automatisées (blocage, alertes)

---

## 2. Cahier des charges du projet

### Nom du projet

**“Cowrie Threat Intel Node – CTIN-01”**

### Objectif

Mettre en place un honeypot SSH/Telnet interactif sécurisé, analyser les attaques reçues pendant plusieurs jours, enrichir les données et présenter un tableau de bord ou un rapport de Threat Intelligence.

---

## 3. Environnement de travail

* **Support recommandé** : Machine virtuelle (Proxmox, VirtualBox, VPS) ou conteneur Docker
* **OS** : Debian 11 ou Ubuntu 20.04+
* **Réseau** : NAT, accès SSH ou console
* **Accès root ou sudo** requis

---

## 4. Livrables attendus

### 4.1. Dossier compressé : `prenom_nom_projet-cowrie.zip` contenant :

* Répertoire du projet Cowrie avec configuration personnalisée
* Captures d’écran des connexions, logs, fichiers malveillants
* Rapport complet d’analyse
* Fichier CSV ou dashboard exporté (PDF ou JSON Grafana)
* Script(s) d’automatisation (bash, jq, alertes…)

---

## 5. Étapes détaillées du projet

---

### Étape 1 – Déploiement sécurisé de Cowrie

* Installer Cowrie manuellement ou via Docker
* Isoler le système (utilisateur dédié, pare-feu, etc.)
* Personnaliser le système simulé (`fs.pickle` ou `honeyfs`)
* Activer SSH et/ou Telnet

---

### Étape 2 – Observation sur 48h à 5 jours

* Laisser Cowrie actif sur le port 2222 (ou redirigé depuis 22)
* Capturer les sessions
* Noter les IPs, scripts, comportements observés
* Identifier les typologies d’attaque

---

### Étape 3 – Analyse approfondie

* Rejouer les sessions (`ttylog/`)
* Extraire les commandes (`cowrie.command.input`)
* Identifier les fichiers téléchargés (`dl/`)
* Enrichir les IPs et URL avec AbuseIPDB, VirusTotal

---

### Étape 4 – Intégration dans un outil de visualisation

* Option 1 : **Kibana (via Filebeat + Elasticsearch)**
* Option 2 : **Grafana (via Loki ou JSON Datasource)**
* Option 3 : **CSV exploité dans Google Sheets ou Excel**

Créer :

* Une heatmap des IP par pays
* Un graphique des commandes tapées
* Un top des fichiers/malwares rencontrés

---

### Étape 5 – Réaction et défense

* Activer fail2ban
* Ajouter un script Bash pour bloquer les IPs après 5 tentatives
* Créer un script d’alerte mail ou webhook (optionnel)

---

### Étape 6 – Rapport d’incident

Contenu attendu :

| Élément               | Contenu attendu                        |
| --------------------- | -------------------------------------- |
| Page de garde         | Titre, auteur, date                    |
| Résumé exécutif       | Objectif du projet                     |
| Environnement         | OS, config réseau, services Cowrie     |
| Attaques observées    | IP, scripts, type de comportement      |
| Fichiers malveillants | SHA256, analyse statique               |
| IOCs                  | Liste d’indicateurs réutilisables      |
| Recommandations       | Blocage, feed IOC, actions préventives |
| Annexes               | Logs, captures, commandes utilisées    |

Format : PDF ou Markdown (`rapport.md`)

---

## 6. Évaluation

| Critère                                        | Points   |
| ---------------------------------------------- | -------- |
| Déploiement fonctionnel de Cowrie              | 5        |
| Configuration personnalisée crédible           | 5        |
| Collecte et analyse pertinente des attaques    | 5        |
| Enrichissement des données (IOCs)              | 5        |
| Rapport rédigé et structuré                    | 5        |
| Automatisation (script, fail2ban, dashboard)   | 5        |
| Présentation orale ou soutenance (facultative) | +5 bonus |

**Total : /30 (+5 bonus éventuels)**

---

## 7. Questions de synthèse (à l’écrit ou à l’oral)

1. Quelle est la principale limite de Cowrie face à un attaquant expérimenté ?
2. Quelle méthode as-tu utilisée pour extraire les IOCs ?
3. Quelle valeur ajoutée Cowrie apporte-t-il dans une démarche de cyberdéfense ?
4. Que ferais-tu pour passer à l’échelle et déployer plusieurs honeypots ?
5. Comment intégrer Cowrie dans un SOC ou un SIEM d’entreprise ?

---

## 8. Ressources pour le projet

* Documentation officielle : [https://cowrie.readthedocs.io](https://cowrie.readthedocs.io)
* Dépôt GitHub : [https://github.com/cowrie/cowrie](https://github.com/cowrie/cowrie)
* AbuseIPDB API : [https://www.abuseipdb.com/api.html](https://www.abuseipdb.com/api.html)
* Docker Compose ELK : [https://github.com/deviantony/docker-elk](https://github.com/deviantony/docker-elk)
* Script jq : [https://stedolan.github.io/jq/](https://stedolan.github.io/jq/)
