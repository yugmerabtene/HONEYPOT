# SYLLABUS – COURS COMPLET SUR COWRIE (HONEYPOT SSH/TELNET)

---

## Objectifs pédagogiques généraux


* Comprendre le fonctionnement d’un honeypot interactif
* Déployer, configurer et surveiller Cowrie
* Analyser les attaques captées (commandes, fichiers, IPs, comportements)
* Exploiter les données pour améliorer la sécurité d’un SI
* Créer des règles de détection personnalisées (fail2ban, Suricata, SIEM)


---

### CHAPITRE 01 – Introduction aux Honeypots

* Définition : honeypot vs honeynet
* Types : faible interaction, haute interaction
* Objectifs : détection, tromperie, collecte d’IOC
* Exemples d’usage en Red Team et Blue Team
* TP-01 : Déployer un honeypot simple avec Honeyd ou Dionaea

---

### CHAPITRE 02 – Présentation de Cowrie

* Historique (fork de Kippo)
* Technologies utilisées : Python, Twisted, shell simulé
* Ce que Cowrie simule : SSH, Telnet, shell Linux, SFTP, SCP
* Architecture générale de Cowrie
* TP-02 : Étude des capacités de Cowrie en environnement déconnecté

---

### CHAPITRE 03 – Installation de Cowrie

* Pré-requis système (VM Debian/Ubuntu)
* Environnement isolé avec VirtualBox ou Proxmox
* Installation manuelle ou via Docker
* Configuration initiale (cowrie.cfg)
* TP-03 : Installation manuelle pas à pas

---

### CHAPITRE 04 – Configuration et Personnalisation

* cowrie.cfg, cowrie.cfg.dist
* Faux utilisateurs et mots de passe
* Simuler des commandes Unix, faux fichiers système (fs.pickle)
* Activer/désactiver Telnet, SSH, SCP, SFTP
* Intégration DNS : redirection de toutes les IP vers 127.0.0.1
* TP-04 : Personnaliser un faux système cible

---

### CHAPITRE 05 – Surveillance et Collecte de Données

* Emplacement des logs : log/cowrie.json, log/tty/
* Interprétation des logs : commandes, sessions, uploads
* Scripts d’analyse automatique
* Alerting via e-mail, Discord, Telegram (modules externes)
* TP-05 : Visualiser les logs et créer un mini dashboard

---

### CHAPITRE 06 – Détection et Réaction

* Analyse comportementale : commandes, scans, fichiers malveillants
* Blocage automatique via fail2ban ou iptables
* Export vers un SIEM (Elastic Stack, Wazuh)
* TP-06 : Intégration avec fail2ban pour bloquer automatiquement les IPs

---

### CHAPITRE 07 – Sécurité et Cloisonnement

* Exécution dans un environnement sécurisé : VM isolée, namespace, Docker
* Reverse shell et détection d’escape
* Sauvegarde et nettoyage automatique
* TP-07 : Cloisonner Cowrie dans un conteneur sécurisé (Docker)

---

### CHAPITRE 08 – Analyse Avancée des Données

* Extraction des IOCs
* Requêtes IP à AbuseIPDB, VirusTotal
* Visualisation avec Kibana ou Grafana (via Logstash)
* TP-08 : Construire un mini tableau de bord avec Grafana et Cowrie

---

### CHAPITRE 09 – Cas Concrets d’Attaques

* Analyse de sessions réelles : brute-force, reverse shell, malware, cryptominers
* Étude de fichiers malveillants : strings, yara, VirusTotal
* Préparation de rapports d’incidents
* TP-09 : Forensic sur une session capturée par Cowrie

---

### CHAPITRE 10 – Projet Final

Projet : Créer, déployer et documenter une plateforme honeypot opérationnelle avec Cowrie, alertes automatiques, et journalisation dans un SIEM

* Déploiement sur VPS ou réseau isolé
* Documentation complète : diagrammes, scripts, dashboard
* Présentation orale ou rapport écrit

Livrables :

* VM ou conteneur prêt à l’emploi
* Rapport PDF ou Markdown
* Captures, logs, analyse

---

## Outils utilisés

* Python, Twisted
* Docker, systemd
* fail2ban, iptables
* Elasticsearch, Logstash, Kibana ou Grafana
* VirusTotal, AbuseIPDB, Shodan
