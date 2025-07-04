# CHAPITRE 01 – Introduction aux Honeypots

---

## 1. Objectifs pédagogiques

À la fin de ce chapitre, l’apprenant sera capable de :

* Définir ce qu’est un honeypot
* Différencier les types de honeypots (faible/forte interaction)
* Comprendre les usages en cybersécurité (défensif et offensif)
* Déployer un honeypot simple à des fins d’analyse

---

## 2. Partie théorique

### 2.1. Qu’est-ce qu’un honeypot ?

Un **honeypot** est un système informatique factice, conçu pour être attaqué, infiltré ou sondé. Il sert à **tromper l’attaquant** et à **collecter des informations sur son comportement**, sans risque pour le système réel.

On l’oppose à un système de production : ici, **tout accès est considéré comme malveillant par nature**.

---

### 2.2. Objectifs principaux

* **Détection** : identifier les tentatives d’intrusion (ex. : brute-force SSH)
* **Tromperie** : faire perdre du temps à l’attaquant, récolter ses techniques
* **Collecte d’IOCs** : adresses IP, noms de fichiers, commandes, indicateurs de compromission
* **Recherche** : étudier des malwares, outils d’exploitation, scripts automatisés
* **Leurre** : détourner l’attention loin des vraies ressources critiques

---

### 2.3. Typologie des honeypots

| Type d’interaction     | Description                                                                        | Exemple                 |
| ---------------------- | ---------------------------------------------------------------------------------- | ----------------------- |
| **Faible interaction** | Simule uniquement quelques ports/services (pas de système complet)                 | Honeyd, Glastopf        |
| **Forte interaction**  | Exécute un vrai système ou shell simulé, plus réaliste mais plus risqué            | Cowrie, Dionaea, Conpot |
| **Client honeypot**    | Simule un client (navigateur, FTP...) qui interagit avec des serveurs malveillants | Thug, HoneyC            |

---

### 2.4. Honeypot vs Honeynet

* **Honeypot** : une machine ou un service factice isolé
* **Honeynet** : un **réseau entier** d’honeypots, souvent géré avec des passerelles (honeygate), conçu pour observer des comportements plus larges (attaques internes, latérales…)

---

### 2.5. Usages pratiques dans un SOC

* Intégration dans un **système de détection d’intrusion (IDS)** ou dans un **SIEM**
* Création de règles de détection basées sur les IOCs observés
* **Détection proactive** de scanners, bots, APTs silencieux
* **Feed de Threat Intelligence** interne

---

## 3. Étude de cas (orale ou en vidéo)

**Cas : Brute-force SSH sur un serveur Cowrie en ligne**

* L’attaquant tente 243 mots de passe en 1 minute
* Cowrie simule une connexion réussie
* L’attaquant téléverse un script `.sh` via SCP (download depuis pastebin)
* Objectif : télécharger un cryptominer dissimulé

---

## 4. Travaux Pratiques

### TP-01 : Déploiement d’un honeypot simple avec Honeyd (ou `snare`)

**Objectif** : Comprendre les principes d’écoute de ports, d’illusion de services, et capturer une connexion automatisée.

#### Étapes :

1. Préparer une VM Debian/Ubuntu à jour (ou WSL2 si besoin)
2. Installer `honeyd` ou `snare`


----
Installer Snare :  https://snare.readthedocs.io/en/latest/quick-start.html
----
Sur **Ubuntu 22.04+**, le paquet `honeyd` **n'existe plus dans les dépôts officiels**, mais tu peux l’installer **manuellement en le compilant depuis les sources**.


# INSTALLATION DE HONEYD SUR UBUNTU 22.04

---

## 1. Prérequis

Ouvre un terminal et installe les dépendances nécessaires :

```bash
sudo apt update
sudo apt install -y git autoconf libtool libevent-dev libpcap-dev \
libdnet-dev bison flex libconfig-dev libglib2.0-dev make g++ zlib1g-dev
```

---

## 2. Télécharger et compiler `libdnet` corrigé

La version officielle de `libdnet` pose problème sur Ubuntu > 20.04. On utilise un fork corrigé.

```bash
cd /usr/local/src
sudo git clone https://github.com/dugsong/libdnet.git
cd libdnet
sudo ./configure
sudo make
sudo make install
sudo ldconfig
```

---

## 3. Télécharger et compiler Honeyd

```bash
cd /usr/local/src
sudo git clone https://github.com/DataSoft/Honeyd.git
cd Honeyd
sudo ./configure
sudo make
sudo make install
```

---

## 4. Vérification

Tape simplement :

```bash
honeyd -v
```

Tu devrais voir s'afficher :

```text
Honeyd version 1.6c
```

---

## 5. Exemple de configuration simple

Créer un fichier `honeyd.conf` :

```ini
create default
set default personality "Linux 3.10"
set default default tcp action reset
add default tcp port 22 open
add default tcp port 80 open
```

Puis exécuter Honeyd avec une IP virtuelle (adapter à ton réseau) :

```bash
sudo honeyd -d -f honeyd.conf -i eth0 192.168.1.100
```

---

## 6. Vérification avec `nmap` depuis une autre machine

```bash
nmap -sS -Pn 192.168.1.100
```

Tu devrais voir les ports 22 et 80 "ouverts", mais simulés.

---

 ----
3. Configurer un service fictif sur le port 22 (ou 80, selon le besoin)
   Exemple `honeyd.conf` :

   ```
   create template
   set template personality "Linux 3.0"
   set template default tcp action reset
   add template tcp port 22 "sh /home/user/fake-ssh.sh"
   bind 192.168.1.100 template
   ```
4. Lancer le honeypot

   ```bash
   sudo honeyd -d -f honeyd.conf -i eth0
   ```
5. Depuis un autre poste, lancer un scan avec `nmap` :

   ```bash
   nmap -A 192.168.1.100
   ```
6. Observer les logs produits et l’apparente "existence" d’un service SSH

---

## 5. Questions / Quiz de fin de chapitre

1. Quelle est la différence entre un honeypot à faible et à forte interaction ?
2. À quoi sert un honeypot dans une architecture réseau d’entreprise ?
3. Quels sont les risques de faire tourner un honeypot mal configuré ?
4. Peut-on utiliser un honeypot pour détecter les malwares 0-day ?
5. Quelle est la valeur ajoutée d’un honeynet par rapport à un simple honeypot ?

---

## 6. Ressources complémentaires

* [https://cowrie.readthedocs.io/](https://cowrie.readthedocs.io/)
* [https://github.com/micheloosterhof/cowrie](https://github.com/micheloosterhof/cowrie)
* [https://projects.thehoneynet.org](https://projects.thehoneynet.org)
* Paper : "Know Your Enemy" – The Honeynet Project
