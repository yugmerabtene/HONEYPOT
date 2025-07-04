# CHAPITRE 03 – Installation de Cowrie

---

## 1. Objectifs pédagogiques

À l’issue de ce chapitre, l’apprenant sera capable de :

* Préparer un environnement sécurisé pour installer Cowrie
* Installer Cowrie manuellement sur un système Linux (VM ou serveur)
* Comprendre les dépendances logicielles et la structure des fichiers d’installation
* Vérifier que Cowrie est opérationnel et prêt à capturer des attaques

---

## 2. Pré-requis techniques

* Distribution Linux : Debian 11+, Ubuntu 20.04+ (VM ou VPS)
* Accès root (ou `sudo`)
* Python 3.8+
* Ports 22, 2222 ou 2223 disponibles
* Recommandé : test en VM isolée ou conteneur

---

## 3. Étapes d’installation manuelle

### 3.1. Préparer le système

Mettre à jour et installer les dépendances :

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git python3 python3-pip python3-venv libssl-dev libffi-dev libpython3-dev virtualenv libjpeg-dev build-essential authbind
```

Créer un utilisateur dédié pour Cowrie :

```bash
sudo useradd -r -s /bin/false cowrie
```

---

### 3.2. Télécharger le projet Cowrie

```bash
cd /opt
sudo git clone https://github.com/cowrie/cowrie.git
sudo chown -R $USER:$USER cowrie
cd cowrie
```

---

### 3.3. Créer un environnement virtuel Python

```bash
python3 -m venv cowrie-env
source cowrie-env/bin/activate
```

Installer les dépendances :

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

---

### 3.4. Configurer Cowrie

Copier les fichiers de configuration :

```bash
cp etc/cowrie.cfg.dist etc/cowrie.cfg
cp etc/userdb.txt.dist etc/userdb.txt
```

Éditer `etc/cowrie.cfg` si besoin :

* `listen_port = 2222`
* `hostname = debian`
* `enabled = true` pour ssh ou telnet
* `downloads.output = dl/`

---

### 3.5. Créer les répertoires nécessaires

```bash
mkdir -p var/run log dl share
```

Donner les droits :

```bash
sudo chown -R $USER:$USER .
```

---

### 3.6. Lancer Cowrie

Toujours depuis l’environnement virtuel :

```bash
source cowrie-env/bin/activate
bin/cowrie start
```

Vérifier que le service écoute :

```bash
ss -tuln | grep 2222
```

---

### 3.7. Tester la connexion

Depuis le système hôte ou une autre machine :

```bash
ssh root@<IP_VM> -p 2222
```

Entrer un mot de passe (ex. : `admin`, `123456`, `toor`)

---

### 3.8. Logs générés

* `log/cowrie.log` : journal principal
* `log/cowrie.json` : version structurée JSON
* `log/tty/*` : session complète enregistrée (rejouable)
* `dl/` : fichiers téléchargés par l’attaquant

---

## 4. Astuce pour port 22 (optionnel)

Cowrie écoute par défaut sur 2222. Pour simuler un vrai serveur SSH :

Rediriger le port 22 vers 2222 avec `iptables` :

```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222
```

ou avec `authbind` :

```bash
sudo apt install authbind
sudo touch /etc/authbind/byport/22
sudo chown cowrie /etc/authbind/byport/22
sudo chmod 500 /etc/authbind/byport/22
authbind --deep bin/cowrie start
```

---

## 5. TP-03 : Installation complète pas à pas

**Objectif** : Installer Cowrie manuellement, le lancer, simuler une attaque et observer les logs.

**Instructions** :

1. Créer une VM Debian 11 minimale avec 1 Go RAM, 1 vCPU, 10 Go disque
2. Suivre les étapes 3.1 à 3.7
3. Tester une connexion SSH simulée
4. Observer les logs générés
5. Arrêter Cowrie proprement :

```bash
bin/cowrie stop
```

---

## 6. Questions de révision

1. Pourquoi installe-t-on Cowrie dans un environnement virtuel Python ?
2. Quelle est la différence entre `cowrie.log` et `cowrie.json` ?
3. Pourquoi utiliser un port autre que 22 par défaut ?
4. Comment savoir si Cowrie fonctionne correctement ?
5. Où sont stockés les fichiers uploadés par l’attaquant ?

---

## 7. Ressources utiles

* Installation officielle : [https://cowrie.readthedocs.io/en/latest/INSTALL.html](https://cowrie.readthedocs.io/en/latest/INSTALL.html)
* GitHub du projet : [https://github.com/cowrie/cowrie](https://github.com/cowrie/cowrie)
* Guide rapide (CLI) : `bin/cowrie start` / `bin/cowrie stop`
* Rejouer une session : `bin/playlog.py log/tty/ttylog.20250704`

