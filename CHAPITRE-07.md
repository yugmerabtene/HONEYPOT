# CHAPITRE 07 – Sécurisation et cloisonnement de Cowrie

---

## 1. Objectifs pédagogiques

À l’issue de ce chapitre, l’apprenant sera capable de :

* Comprendre les risques liés à l’exploitation d’un honeypot non isolé
* Déployer Cowrie dans un environnement cloisonné et sécurisé (VM, conteneur)
* Protéger le système hôte contre les actions accidentelles ou malveillantes
* Appliquer les bonnes pratiques de sandboxing et de surveillance
* Réaliser un déploiement Dockerisé de Cowrie

---

## 2. Risques associés à un honeypot mal cloisonné

### 2.1. Compromission potentielle de l’hôte

Même si Cowrie **n’exécute pas réellement les commandes**, une mauvaise configuration peut exposer :

* Des ports non contrôlés
* Le système de fichiers réel
* Le réseau local de l’entreprise
* Des outils comme wget/curl réels (si mal filtrés)

### 2.2. Risques légaux

Si l’honeypot n’est pas isolé :

* Il peut être utilisé comme **pivot pour attaquer d'autres systèmes**
* Il peut **télécharger réellement des malwares** sur Internet
* Il peut **héberger involontairement des fichiers illicites**

---

## 3. Bonnes pratiques de sécurité pour Cowrie

* Utiliser **un utilisateur système non privilégié**
* Bloquer **tout accès sortant Internet** (sauf si analysé)
* **Restreindre les ports ouverts** (seulement 2222 ou 22 redirigé)
* Ne jamais installer Cowrie sur un système de production
* Activer **le logging local uniquement** (pas d’accès à des services sensibles)
* Mettre en place **une politique de nettoyage automatique**

---

## 4. Cloisonnement avec une machine virtuelle (méthode classique)

### 4.1. Avantages

* Isolation mémoire, disque, CPU, réseau
* Possibilité de snapshot / rollback rapide
* Pas d’impact sur le système hôte

### 4.2. Configuration recommandée

* Hyperviseur : VirtualBox, KVM, VMware, Proxmox
* VM Debian 11 ou Ubuntu 20.04 LTS
* 1 vCPU, 2 Go RAM, 10 Go disque
* Réseau NAT ou "réseau interne" (pas bridge sauf pour CTF)

---

## 5. Cloisonnement avec Docker (méthode moderne)

### 5.1. Avantages

* Déploiement rapide et portable
* Isolation réseau/fichier/processus
* Limitation facile des ressources CPU/RAM

### 5.2. Étapes de déploiement

#### 1. Dockerfile minimal pour Cowrie

```dockerfile
FROM python:3.10-slim

RUN apt update && apt install -y git libssl-dev libffi-dev libjpeg-dev \
    libpython3-dev build-essential authbind iproute2

RUN useradd -m cowrie
USER cowrie

WORKDIR /home/cowrie
RUN git clone https://github.com/cowrie/cowrie.git
WORKDIR /home/cowrie/cowrie
RUN python3 -m venv cowrie-env && \
    . cowrie-env/bin/activate && \
    pip install -r requirements.txt

CMD ["bin/cowrie", "start", "-n"]
```

#### 2. Build et run

```bash
docker build -t cowrie-secure .
docker run -d --name cowrie \
  -p 2222:2222 \
  --read-only \
  --cap-drop=ALL \
  --memory=512m \
  --cpus=0.5 \
  cowrie-secure
```

### 5.3. Options de sécurité Docker recommandées

* `--read-only` : le conteneur ne peut pas écrire dans le FS
* `--cap-drop=ALL` : suppression de toutes les capacités Linux
* `--network=none` (si besoin d’un honeypot *off-line*)
* `--memory=512m` / `--cpus=0.5` : limitation des ressources

---

## 6. Cloisonnement réseau

* Utiliser une carte réseau NAT ou un VLAN dédié
* Interdire le trafic sortant (firewall)
* Option : utiliser `dnsmasq` ou `/etc/hosts` pour simuler Internet

---

## 7. Nettoyage et purge automatique

Script bash à lancer en cron pour purger les logs après 7 jours :

```bash
#!/bin/bash
find /opt/cowrie/log/ -type f -mtime +7 -delete
find /opt/cowrie/dl/ -type f -mtime +7 -delete
```

Éventuellement, archiver :

```bash
tar -czf /opt/archives/cowrie_logs_$(date +%F).tar.gz /opt/cowrie/log
```

---

## 8. TP-07 : Déploiement sécurisé de Cowrie avec Docker

### Objectif

Déployer Cowrie en conteneur sécurisé, sans accès au système hôte, avec contrôle des ressources et des logs.

### Étapes

1. Créer un Dockerfile selon l’exemple ci-dessus
2. Construire l’image :

```bash
docker build -t cowrie-secure .
```

3. Lancer le conteneur :

```bash
docker run -d --name cowrie -p 2222:2222 --read-only --cap-drop=ALL cowrie-secure
```

4. Tester une connexion SSH :

```bash
ssh root@localhost -p 2222
```

5. Vérifier que les logs sont stockés dans un volume externe (montage si besoin)

---

## 9. Questions de révision

1. Pourquoi est-il dangereux d’exécuter Cowrie sur une machine non isolée ?
2. Quelles sont les différences principales entre cloisonnement via VM et Docker ?
3. Comment empêcher Cowrie de sortir sur Internet ?
4. Pourquoi limiter les capacités (`cap-drop`) d’un conteneur Docker ?
5. Quels fichiers doivent être nettoyés régulièrement sur une instance Cowrie active ?

---

## 10. Ressources

* Docker : [https://docs.docker.com/](https://docs.docker.com/)
* Proxmox : [https://www.proxmox.com/en/](https://www.proxmox.com/en/)
* Cowrie Hardening (GitHub issues et Wiki)
* Seccomp, AppArmor, cgroups : pour aller plus loin en confinement
