# CHAPITRE 04 – Configuration et personnalisation de Cowrie

---

## 1. Objectifs pédagogiques

À la fin de ce chapitre, l’apprenant sera capable de :

* Modifier les fichiers de configuration principaux de Cowrie
* Créer de faux utilisateurs et mots de passe pour piéger les attaquants
* Simuler une arborescence de fichiers crédible (fs.pickle ou honeyfs)
* Personnaliser le comportement du faux shell
* Activer/désactiver des services comme Telnet, SSH, SCP, SFTP
* Préparer Cowrie pour la collecte réaliste de données

---

## 2. Fichiers de configuration principaux

### 2.1. Fichier `cowrie.cfg`

Le fichier `cowrie.cfg` (dans `etc/`) permet de modifier les paramètres globaux.

Pour l’activer :

```bash
cp etc/cowrie.cfg.dist etc/cowrie.cfg
```

Les sections clés sont :

* `[ssh]`

  * `enabled = true` ou `false`
  * `listen_port = 2222`
* `[telnet]`

  * `enabled = false` (par défaut)
* `[output_jsonlog]`

  * Active la journalisation JSON (utile pour Kibana, Logstash…)
* `[honeypot]`

  * `hostname = debian`
  * `download.enabled = true`

---

### 2.2. Utilisateurs et mots de passe (`userdb.txt`)

Fichier situé dans `etc/userdb.txt`.

Il permet de définir les couples utilisateur/mot de passe qui **permettent une connexion réussie au faux shell** :

Exemple :

```
root:x
admin:admin123
user:test
```

Tous les autres logins seront considérés comme des tentatives échouées (mais enregistrées).

---

## 3. Simuler un système de fichiers crédible

Cowrie peut utiliser deux systèmes :

* `fs.pickle` : plus rapide (fichier binaire sérialisé)
* `honeyfs/` : système de fichiers simulé en clair

### 3.1. Modifier `fs.pickle`

1. Extraire l'arborescence :

```bash
bin/fsctl --dump > fs.txt
```

2. Éditer le fichier `fs.txt` :

```text
/etc/passwd
/bin/ls
/var/log/syslog
/tmp/test.sh
```

3. Recompiler :

```bash
bin/fsctl --build fs.txt
```

Cela crée un nouveau `share/fs.pickle`.

---

### 3.2. Utiliser un système de fichiers personnalisé (optionnel)

Créer un vrai système simulé dans `honeyfs/` :

```bash
mkdir -p honeyfs/etc
echo "root:x:0:0:root:/root:/bin/bash" > honeyfs/etc/passwd
```

Configurer Cowrie pour le lire dans `cowrie.cfg` :

```
[honeypot]
filesystem = honeyfs
```

---

## 4. Personnalisation du comportement du shell

Les commandes simulées sont situées dans :

* `cowrie/commands/`
* `cowrie/fs/` (système de fichiers simulé)
* `cowrie/commands/base_command.py` : pour créer une commande custom

Exemples de commandes disponibles :

* `ls`, `cd`, `uname`, `cat`, `echo`, `rm`, `cp`, `wget`, etc.

Créer une commande personnalisée :

```bash
touch cowrie/commands/my_command.py
```

```python
from cowrie.shell.command import HoneyPotCommand

class command_mycmd(HoneyPotCommand):
    def call(self):
        self.write("Commande personnalisée exécutée\n")
```

---

## 5. Activer ou désactiver des modules

Dans `cowrie.cfg` :

### SSH et Telnet

```
[ssh]
enabled = true
listen_port = 2222

[telnet]
enabled = false
listen_port = 2223
```

### Téléchargement / Téléversement

```
[honeypot]
download.enabled = true
download.output = dl/
```

Permet d’enregistrer tous les fichiers récupérés par `wget`, `curl`, `scp`, `sftp`.

---

## 6. Redirection DNS fictive (optionnel)

Pour éviter qu’un attaquant ne contacte un vrai serveur :

* Modifier `/etc/hosts` :

```bash
127.0.0.1   pastebin.com
127.0.0.1   malware.com
```

* Ou configurer un DNS local pour toutes les requêtes inconnues

---

## 7. TP-04 : Personnalisation complète

**Objectif** : créer un faux système Debian piégé, un utilisateur appât, simuler un shell réaliste, et observer les tentatives d’attaque.

### Étapes :

1. Modifier `etc/userdb.txt` :

```
sysadmin:toor
admin:admin123
```

2. Ajouter des fichiers à `honeyfs/` :

```bash
mkdir -p honeyfs/etc honeyfs/root
echo "127.0.0.1 localhost" > honeyfs/etc/hosts
echo "echo 'mot de passe volé'" > honeyfs/root/motdepasse.sh
```

3. Modifier `cowrie.cfg` :

```
[honeypot]
filesystem = honeyfs
hostname = server-debian
download.enabled = true
```

4. Relancer Cowrie :

```bash
bin/cowrie restart
```

5. Se connecter :

```bash
ssh sysadmin@localhost -p 2222
```

6. Taper :

```bash
ls /root
bash /root/motdepasse.sh
```

7. Observer :

```bash
ls log/cowrie.json
ls dl/
```

---

## 8. Questions de révision

1. Quelle est la différence entre `fs.pickle` et `honeyfs/` ?
2. Comment ajouter un mot de passe valide pour un faux utilisateur ?
3. Où se trouve le code des commandes comme `ls`, `cd`, `wget` ?
4. Pourquoi rediriger pastebin.com vers 127.0.0.1 dans `/etc/hosts` ?
5. Comment capturer les fichiers téléchargés par l’attaquant ?

---

## 9. Ressources

* Exemple de systèmes fictifs : [GitHub Cowrie HoneyFS Samples](https://github.com/micheloosterhof/cowrie/tree/master/honeyfs)
* Documentation commandes personnalisées : [https://cowrie.readthedocs.io](https://cowrie.readthedocs.io)

