# CHAPITRE 08 – Analyse avancée des données

---

## 1. Objectifs pédagogiques

À l’issue de ce chapitre, l’apprenant sera capable de :

* Extraire des indicateurs de compromission (IOC) à partir des logs de Cowrie
* Identifier les IPs malveillantes, les types de commandes utilisées, les fichiers téléchargés
* Visualiser les attaques avec Kibana, Grafana, ou un outil maison
* Utiliser des outils d’enrichissement comme VirusTotal, AbuseIPDB, Shodan
* Mettre en place un mini tableau de bord de threat intelligence

---

## 2. Rappel des formats de logs

### 2.1. `cowrie.json` (structuré)

Contient tous les événements importants (connexion, login, commandes, téléchargement...).

Lecture recommandée :

```bash
cat log/cowrie.json | jq '.'
```

### 2.2. `ttylog/` (session binaire)

Permet la relecture d’une session complète :

```bash
bin/playlog.py log/tty/ttylog.20250704-abc.log
```

### 2.3. `dl/`

Contient les fichiers uploadés ou téléchargés par les attaquants.

---

## 3. Extraction d’IOCs

### 3.1. Adresses IP attaquantes

```bash
jq 'select(.eventid=="cowrie.session.connect") | .src_ip' log/cowrie.json | sort | uniq -c | sort -nr
```

### 3.2. Commandes utilisées

```bash
jq 'select(.eventid=="cowrie.command.input") | .input' log/cowrie.json | sort | uniq -c | sort -nr
```

### 3.3. Fichiers téléchargés

```bash
jq 'select(.eventid=="cowrie.session.file_download") | {url: .url, dest: .outfile}' log/cowrie.json
```

---

## 4. Enrichissement des IOCs

### 4.1. VirusTotal (hash, fichier)

* Calculer un hash :

```bash
sha256sum dl/evil.sh
```

* Rechercher sur [https://www.virustotal.com/](https://www.virustotal.com/)

### 4.2. AbuseIPDB (adresse IP)

```bash
curl https://api.abuseipdb.com/api/v2/check \
  -G --data-urlencode "ipAddress=185.213.21.88" \
  -H "Key: VOTRE_CLE_API" \
  -H "Accept: application/json"
```

### 4.3. Shodan

Utiliser un outil comme `shodan` (CLI ou API) :

```bash
shodan host 185.213.21.88
```

---

## 5. Visualisation avec Kibana (Stack ELK)

### 5.1. Architecture

* Cowrie produit un `cowrie.json`
* Filebeat (ou Logstash) le lit et l’envoie à Elasticsearch
* Kibana affiche les événements

### 5.2. Exemple de pipeline simplifié

1. Installer Filebeat :

```bash
sudo apt install filebeat
```

2. Configuration dans `/etc/filebeat/filebeat.yml` :

```yaml
filebeat.inputs:
- type: log
  paths:
    - /opt/cowrie/log/cowrie.json
  json.keys_under_root: true
  json.add_error_key: true

output.elasticsearch:
  hosts: ["localhost:9200"]
```

3. Lancer Filebeat :

```bash
sudo systemctl restart filebeat
```

4. Ouvrir Kibana et créer un index `filebeat-*`

5. Visualisation :

* Histogramme des attaques par heure
* Heatmap des IP par pays
* Liste des commandes utilisées

---

## 6. Visualisation avec Grafana + Loki

1. Installer Grafana et Loki
2. Configurer une source Loki sur le fichier JSON de Cowrie
3. Créer des dashboards :

* Nombre de connexions par IP
* Détail des commandes tapées
* Téléchargements suspects

---

## 7. TP-08 : Tableau de bord simple avec `jq` et `gnuplot` ou `csv`

### Objectif

Créer une extraction CSV des attaques pour visualisation manuelle ou dans Excel.

### Étapes

1. Extraire les IPs et commandes :

```bash
jq -r 'select(.eventid=="cowrie.command.input") | [.timestamp, .src_ip, .input] | @csv' log/cowrie.json > attacks.csv
```

2. Ouvrir dans Excel, LibreOffice ou Google Sheets

3. Créer un histogramme des commandes tapées par IP

---

## 8. Questions de révision

1. Quelle est la différence entre `cowrie.log` et `cowrie.json` ?
2. Comment extraire les IPs qui ont tenté une connexion avec `jq` ?
3. Quelles sont les fonctions d’AbuseIPDB et VirusTotal ?
4. Quelle est la chaîne ELK et son intérêt pour Cowrie ?
5. Comment enrichir des IOCs capturés par Cowrie pour un rapport de sécurité ?

---

## 9. Ressources

* [https://www.virustotal.com/](https://www.virustotal.com/)
* [https://www.abuseipdb.com/](https://www.abuseipdb.com/)
* [https://www.shodan.io/](https://www.shodan.io/)
* [https://www.elastic.co/](https://www.elastic.co/)
* jq : [https://stedolan.github.io/jq/](https://stedolan.github.io/jq/)
