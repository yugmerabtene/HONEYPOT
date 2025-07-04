# CHAPITRE 06 – Détection et Réaction

---

## 1. Objectifs pédagogiques

À l’issue de ce chapitre, l’apprenant sera capable de :

* Automatiser la détection d’activités malveillantes à partir des logs Cowrie
* Intégrer Cowrie à des outils comme fail2ban, iptables, Wazuh ou Suricata
* Créer des règles personnalisées de blocage et d’alerte
* Mettre en place une chaîne de réaction basique (prévention, alerte, blocage)

---

## 2. Introduction à la détection comportementale

Cowrie permet de **capturer le comportement des attaquants** :

* Commandes tapées (`cowrie.command.input`)
* IP source de connexion (`cowrie.session.connect`)
* Téléversements et téléchargements (`cowrie.session.file_*`)

**Réagir rapidement** à ces événements permet de :

* Bloquer les IP malveillantes (firewall, fail2ban)
* Alerter un administrateur (mail, webhook, log SIEM)
* Générer des IOC dynamiques (indicateurs réutilisables dans d’autres outils)

---

## 3. Utilisation de fail2ban avec Cowrie

### 3.1. Présentation

`fail2ban` surveille des fichiers de log et **bannit les IP** si elles remplissent un certain critère (tentatives d’accès, mots-clés, fréquence).

### 3.2. Installation

```bash
sudo apt install fail2ban
```

### 3.3. Fichier de filtre Cowrie

Créer le fichier `/etc/fail2ban/filter.d/cowrie.conf` :

```ini
[Definition]
failregex = "login attempt \[.*\] failed" from <HOST>
ignoreregex =
```

> Le `<HOST>` est automatiquement remplacé par l’IP source

### 3.4. Fichier jail Cowrie

Ajouter dans `/etc/fail2ban/jail.local` :

```ini
[cowrie]
enabled = true
port = 2222
filter = cowrie
logpath = /opt/cowrie/log/cowrie.log
maxretry = 3
findtime = 600
bantime = 3600
```

Redémarrer fail2ban :

```bash
sudo systemctl restart fail2ban
```

Tester :

```bash
fail2ban-client status cowrie
```

---

## 4. Intégration avec un pare-feu (iptables ou nftables)

Bloquer automatiquement une IP :

```bash
sudo iptables -A INPUT -s 185.213.21.88 -j DROP
```

Script de bannissement automatique à partir des logs :

```bash
#!/bin/bash
for ip in $(jq -r 'select(.eventid=="cowrie.login.failed") | .src_ip' log/cowrie.json | sort | uniq -c | awk '$1 > 5 {print $2}'); do
    sudo iptables -C INPUT -s $ip -j DROP 2>/dev/null || sudo iptables -A INPUT -s $ip -j DROP
done
```

---

## 5. Intégration avec Wazuh

Wazuh est un SIEM open source capable d’analyser les fichiers JSON.

Étapes générales :

1. Configurer un **agent Wazuh** sur la machine Cowrie
2. Surveiller le fichier `cowrie.json`
3. Créer une règle personnalisée (ex : IP inconnue + téléchargement)

Exemple de règle (dans `ruleset/rules/local_rules.xml`) :

```xml
<rule id="100010" level="7">
  <if_sid>100000</if_sid>
  <match>cowrie.session.file_download</match>
  <description>Possible malware download detected in Cowrie</description>
</rule>
```

---

## 6. Intégration avec Suricata

### 6.1. Pourquoi ?

Suricata ne lit pas les fichiers JSON de Cowrie, mais peut utiliser les **IOCs extraits** pour détecter en amont les connexions malveillantes.

### 6.2. Exemple : générer une règle Suricata à partir d'une IP captée

```bash
jq -r 'select(.eventid=="cowrie.session.connect") | .src_ip' log/cowrie.json | sort | uniq | while read ip; do
  echo "alert ip $ip any -> any any (msg:\"Cowrie attacker IP\"; sid:1000001; rev:1;)"
done > cowrie.rules
```

Ajouter cette règle à Suricata :

```bash
sudo mv cowrie.rules /etc/suricata/rules/
```

Et recharger :

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml -v
```

---

## 7. TP-06 : Blocage automatique d'une IP malveillante

**Objectif** : capturer les tentatives SSH d’une IP attaquante via Cowrie, et la bloquer automatiquement avec iptables ou fail2ban.

### Étapes :

1. Lancer Cowrie :

```bash
bin/cowrie start
```

2. Depuis une autre machine, effectuer plusieurs tentatives SSH avec de mauvais mots de passe

3. Observer les logs :

```bash
cat log/cowrie.json | jq '.'
```

4. Vérifier la présence de l’IP dans les tentatives échouées :

```bash
jq -r 'select(.eventid=="cowrie.login.failed") | .src_ip' log/cowrie.json
```

5. Ajouter une règle iptables automatiquement avec un script ou via fail2ban

6. Vérifier avec :

```bash
sudo iptables -L -n | grep DROP
```

---

## 8. Questions de révision

1. Quelle est la différence entre fail2ban et iptables ?
2. Quelle est l’utilité de `cowrie.json` pour un outil comme Wazuh ?
3. Comment créer une règle de blocage basée sur une fréquence d’attaque ?
4. Comment transformer une IP attaquante captée par Cowrie en IOC exploitable ?
5. Quelle est la bonne pratique pour éviter de bloquer de fausses alertes ?

---

## 9. Ressources

* [https://www.fail2ban.org/](https://www.fail2ban.org/)
* [https://wazuh.com/](https://wazuh.com/)
* [https://suricata.io/](https://suricata.io/)
* Cowrie readthedocs - Output Plugins & Logging
