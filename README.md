# 📚 Tutoriel sur SSH

[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Language](https://img.shields.io/badge/Langue-Français-blue.svg)]()

🔐 **Auteur :** Nicolas Deoux - NDXDev@gmail.com
🔗 **LinkedIn :** [Nicolas Deoux](https://www.linkedin.com/in/nicolas-deoux-ab295980/)
📅 **Date de création :** 14 novembre 2025
🌐 **Langue :** Français

---

## 📋 Table des matières

- [Introduction à SSH](#-introduction-%C3%A0-ssh)
- [Installation de SSH](#%EF%B8%8F-installation-de-ssh)
- [Génération de clés SSH](#-g%C3%A9n%C3%A9ration-de-cl%C3%A9s-ssh)
- [Connexion à un serveur distant](#-connexion-%C3%A0-un-serveur-distant)
- [Configuration SSH avancée](#%EF%B8%8F-configuration-ssh-avanc%C3%A9e)
- [Transfert de fichiers avec SCP et SFTP](#-transfert-de-fichiers-avec-scp-et-sftp)
- [Bonnes pratiques de sécurité](#%EF%B8%8F-bonnes-pratiques-de-s%C3%A9curit%C3%A9)
- [FAQ et dépannage](#-faq-et-d%C3%A9pannage)
- [Ressources complémentaires](#-ressources-compl%C3%A9mentaires)

---

## 🔍 Introduction à SSH

**SSH (Secure Shell)** est un protocole de réseau cryptographique qui permet d'accéder en toute sécurité à un ordinateur distant. Il remplace les anciens protocoles non sécurisés comme Telnet et rlogin qui transmettaient les données en clair.

### Pourquoi utiliser SSH ?

- ✅ **Sécurité** : Chiffrement de bout en bout de toutes les communications
- ✅ **Authentification** : Plusieurs méthodes d'authentification (mot de passe, clés publiques, certificats)
- ✅ **Flexibilité** : Transfert de fichiers, tunneling, port forwarding, X11 forwarding
- ✅ **Automatisation** : Exécution de scripts et tâches automatisées sans interaction humaine
- ✅ **Polyvalence** : Compatible avec tous les systèmes d'exploitation majeurs

### Concepts de base

- **Client SSH** : Machine qui initie la connexion (votre ordinateur)
- **Serveur SSH** : Machine qui accepte les connexions (serveur distant, port 22 par défaut)
- **Clés SSH** : Paire de clés cryptographiques (publique/privée) pour l'authentification sans mot de passe
- **Known Hosts** : Fichier contenant les empreintes des serveurs connus pour éviter les attaques MITM
- **Agent SSH** : Programme qui garde en mémoire les clés privées déchiffrées

---

## ⚙️ Installation de SSH

### Sur Linux (Debian/Ubuntu)

```bash
# Installation du client SSH
sudo apt update
sudo apt install openssh-client

# Installation du serveur SSH (si nécessaire)
sudo apt install openssh-server

# Démarrer et activer le service
sudo systemctl start ssh
sudo systemctl enable ssh
```

### Sur Linux (RHEL/CentOS/Fedora)

```bash
# Installation du client SSH
sudo dnf install openssh-clients

# Installation du serveur SSH
sudo dnf install openssh-server

# Démarrer et activer le service
sudo systemctl start sshd
sudo systemctl enable sshd
```

### Sur macOS

SSH est préinstallé sur macOS. Vérifiez la version :

```bash
ssh -V
```

Pour activer le serveur SSH :
1. Préférences Système → Partage
2. Cocher "Session à distance"

### Sur Windows

**Option 1 : OpenSSH (Windows 10/11 - Recommandé)**
1. Paramètres → Applications → Fonctionnalités optionnelles
2. Ajouter une fonctionnalité → OpenSSH Client
3. Pour le serveur : OpenSSH Server

**Option 2 : Git Bash**
- Installez [Git for Windows](https://git-scm.com/download/win) qui inclut OpenSSH

**Option 3 : WSL (Windows Subsystem for Linux)**
- Installez WSL2 et une distribution Linux

**Option 4 : PuTTY (Interface graphique)**
- Téléchargez depuis [putty.org](https://www.putty.org/)

---

## 🔑 Génération de clés SSH

### Étape 1 : Créer une paire de clés

```bash
# Algorithme Ed25519 (recommandé - plus rapide et plus sécurisé)
ssh-keygen -t ed25519 -C "NDXDev@gmail.com"

# Alternative : RSA 4096 bits (compatible avec tous les systèmes)
ssh-keygen -t rsa -b 4096 -C "NDXDev@gmail.com"
```

**Options importantes :**
- `-t` : Type d'algorithme (ed25519, rsa, ecdsa, dsa)
- `-b` : Nombre de bits (pour RSA, minimum 2048, recommandé 4096)
- `-C` : Commentaire pour identifier la clé
- `-f` : Chemin et nom du fichier (ex: `~/.ssh/id_github`)

**Lors de la génération :**
1. Appuyez sur Entrée pour accepter l'emplacement par défaut
2. Entrez une phrase de passe forte (optionnel mais recommandé)
3. Confirmez la phrase de passe

### Étape 2 : Configurer les permissions

```bash
# Sécuriser le répertoire SSH
chmod 700 ~/.ssh

# Permissions pour la clé privée (lecture seule pour vous)
chmod 600 ~/.ssh/id_ed25519

# Permissions pour la clé publique
chmod 644 ~/.ssh/id_ed25519.pub
```

### Étape 3 : Ajouter la clé à l'agent SSH

```bash
# Démarrer l'agent SSH
eval "$(ssh-agent -s)"

# Ajouter la clé privée à l'agent
ssh-add ~/.ssh/id_ed25519

# Vérifier les clés chargées
ssh-add -l
```

### Étape 4 : Copier la clé publique sur le serveur

**Méthode automatique (recommandée) :**

```bash
ssh-copy-id utilisateur@serveur.com
```

**Méthode manuelle :**

```bash
# Afficher la clé publique
cat ~/.ssh/id_ed25519.pub

# Copier manuellement sur le serveur
cat ~/.ssh/id_ed25519.pub | ssh utilisateur@serveur.com "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

**Sur le serveur distant :**

```bash
# Vérifier que la clé a bien été ajoutée
cat ~/.ssh/authorized_keys
```

---

## 🌐 Connexion à un serveur distant

### Connexion basique

```bash
# Syntaxe standard
ssh utilisateur@adresse_serveur

# Exemples
ssh root@exemple.com
ssh admin@192.168.1.100
ssh user@mon-serveur.local
```

### Connexion avec options

```bash
# Spécifier le port
ssh -p 2222 utilisateur@serveur.com

# Utiliser une clé spécifique
ssh -i ~/.ssh/autre_cle utilisateur@serveur.com

# Mode verbose (pour débogage)
ssh -v utilisateur@serveur.com
ssh -vvv utilisateur@serveur.com  # Niveau maximum

# Compression des données
ssh -C utilisateur@serveur.com

# Combinaison d'options
ssh -p 2222 -i ~/.ssh/ma_cle -C utilisateur@serveur.com
```

### Exécution de commandes à distance

```bash
# Exécuter une commande simple
ssh utilisateur@serveur.com "ls -la /home"

# Exécuter plusieurs commandes
ssh utilisateur@serveur.com "cd /var/www && git pull && npm install"

# Rediriger la sortie vers un fichier local
ssh utilisateur@serveur.com "cat /etc/os-release" > os_info.txt

# Utiliser des variables locales
VERSION=$(ssh utilisateur@serveur.com "cat /etc/debian_version")
echo "Version Debian: $VERSION"
```

### Déconnexion

```bash
# Méthodes pour se déconnecter
exit
logout
# Ou Ctrl+D
```

---

## ⚙️ Configuration SSH avancée

### Fichier de configuration `~/.ssh/config`

Créez ou modifiez le fichier `~/.ssh/config` :

```bash
# Configuration globale pour tous les hôtes
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    TCPKeepAlive yes
    ConnectTimeout 10
    IdentitiesOnly yes
    AddKeysToAgent yes

# Serveur de production
Host prod
    HostName production.exemple.com
    User admin
    Port 2222
    IdentityFile ~/.ssh/id_ed25519_prod
    ForwardAgent no
    Compression yes

# Serveur de développement
Host dev
    HostName dev.exemple.com
    User developer
    Port 22
    IdentityFile ~/.ssh/id_ed25519_dev
    ForwardAgent yes

# Serveur local (Raspberry Pi par exemple)
Host pi
    HostName 192.168.1.100
    User pi
    IdentityFile ~/.ssh/id_ed25519_pi
    LocalForward 8080 localhost:80

# GitHub
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_ed25519
    IdentitiesOnly yes

# Configuration multi-comptes GitHub
Host github-perso
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_perso

Host github-travail
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_travail

# Jump Host (bastion)
Host serveur-interne
    HostName 10.0.0.50
    User internal
    ProxyJump bastion

Host bastion
    HostName bastion.exemple.com
    User jumpuser
    IdentityFile ~/.ssh/bastion_key
```

**Utilisation avec le fichier config :**

```bash
# Au lieu de : ssh -p 2222 admin@production.exemple.com
# Utilisez simplement :
ssh prod
```

### Options de configuration importantes

| Option | Description | Valeur recommandée |
|--------|-------------|-------------------|
| `ServerAliveInterval` | Envoie un paquet keep-alive toutes les N secondes | 60 |
| `ServerAliveCountMax` | Nombre de paquets sans réponse avant déconnexion | 3 |
| `TCPKeepAlive` | Maintient la connexion TCP active | yes |
| `ConnectTimeout` | Délai d'attente de connexion (secondes) | 10 |
| `IdentitiesOnly` | Utilise uniquement les clés spécifiées | yes |
| `Compression` | Compresse les données transmises | yes |
| `ForwardAgent` | Transfère l'agent SSH (attention sécurité) | yes/no |
| `AddKeysToAgent` | Ajoute automatiquement les clés à l'agent | yes |

### Tunneling SSH (Port Forwarding)

**Port Forwarding Local :**

```bash
# Accéder à un service distant via localhost
ssh -L [port_local]:[host_distant]:[port_distant] utilisateur@serveur.com

# Exemple : Accéder à une base de données distante
ssh -L 3306:localhost:3306 utilisateur@serveur-bd.com
# Maintenant : mysql -h 127.0.0.1 -P 3306

# Exemple : Accéder à un site web interne
ssh -L 8080:localhost:80 utilisateur@serveur.com
# Maintenant : http://localhost:8080
```

**Port Forwarding Distant :**

```bash
# Exposer un service local sur le serveur distant
ssh -R [port_distant]:[host_local]:[port_local] utilisateur@serveur.com

# Exemple : Exposer votre serveur web local
ssh -R 9000:localhost:3000 utilisateur@serveur.com
# Le serveur peut maintenant accéder à localhost:9000
```

**Dynamic Port Forwarding (Proxy SOCKS) :**

```bash
# Créer un proxy SOCKS
ssh -D 1080 utilisateur@serveur.com

# Configurer votre navigateur pour utiliser :
# Proxy SOCKS5: localhost:1080
```

**Tunnel en arrière-plan :**

```bash
# Option -f : exécute SSH en arrière-plan
# Option -N : ne pas exécuter de commande distante
ssh -f -N -L 8080:localhost:80 utilisateur@serveur.com
```

---

## 📦 Transfert de fichiers avec SCP et SFTP

### SCP (Secure Copy)

**Copier vers le serveur :**

```bash
# Fichier unique
scp fichier.txt utilisateur@serveur.com:/chemin/destination/

# Dossier complet (récursif)
scp -r mon_dossier/ utilisateur@serveur.com:/chemin/destination/

# Plusieurs fichiers
scp fichier1.txt fichier2.txt utilisateur@serveur.com:/chemin/
```

**Copier depuis le serveur :**

```bash
# Fichier unique
scp utilisateur@serveur.com:/chemin/fichier.txt .

# Dossier complet
scp -r utilisateur@serveur.com:/chemin/dossier/ .
```

**Options SCP utiles :**

```bash
# Port personnalisé
scp -P 2222 fichier.txt utilisateur@serveur.com:/chemin/

# Avec compression
scp -C gros_fichier.tar.gz utilisateur@serveur.com:/chemin/

# Limiter la bande passante (en Kbit/s)
scp -l 1000 fichier.txt utilisateur@serveur.com:/chemin/

# Mode verbose
scp -v fichier.txt utilisateur@serveur.com:/chemin/

# Préserver les métadonnées
scp -p fichier.txt utilisateur@serveur.com:/chemin/
```

### SFTP (SSH File Transfer Protocol)

**Mode interactif :**

```bash
# Se connecter au serveur
sftp utilisateur@serveur.com

# Commandes SFTP
sftp> ls                  # Lister les fichiers distants
sftp> lls                 # Lister les fichiers locaux
sftp> pwd                 # Afficher le répertoire distant
sftp> lpwd                # Afficher le répertoire local
sftp> cd /chemin          # Changer de répertoire distant
sftp> lcd /chemin         # Changer de répertoire local
sftp> get fichier.txt     # Télécharger un fichier
sftp> put fichier.txt     # Envoyer un fichier
sftp> get -r dossier/     # Télécharger un dossier
sftp> put -r dossier/     # Envoyer un dossier
sftp> rm fichier.txt      # Supprimer un fichier distant
sftp> mkdir nouveau       # Créer un répertoire distant
sftp> exit                # Quitter
```

**Mode batch (automatique) :**

```bash
# Créer un fichier de commandes batch
cat > commandes_sftp.txt << EOF
cd /var/www
put index.html
put style.css
chmod 644 index.html
bye
EOF

# Exécuter les commandes
sftp -b commandes_sftp.txt utilisateur@serveur.com
```

---

## 🛡️ Bonnes pratiques de sécurité

### 1. Configuration du serveur SSH (`/etc/ssh/sshd_config`)

```bash
# Changer le port par défaut
Port 2222

# Désactiver la connexion root
PermitRootLogin no

# Désactiver l'authentification par mot de passe
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM no

# Autoriser uniquement certains utilisateurs
AllowUsers utilisateur1 utilisateur2
# Ou interdire certains utilisateurs
DenyUsers utilisateur_suspect root

# Limiter les tentatives de connexion
MaxAuthTries 3
MaxSessions 2

# Timeout de connexion
LoginGraceTime 30

# Protocole SSH version 2 uniquement
Protocol 2

# Désactiver le transfert X11 si non nécessaire
X11Forwarding no

# Désactiver les tunnels si non nécessaires
AllowTcpForwarding no
GatewayPorts no

# Algorithmes cryptographiques forts uniquement
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
```

**Après modification, redémarrer SSH :**

```bash
# Tester la configuration
sudo sshd -t

# Redémarrer le service
sudo systemctl restart sshd
```

### 2. Configuration du pare-feu

**UFW (Ubuntu) :**

```bash
# Autoriser SSH sur le nouveau port
sudo ufw allow 2222/tcp

# Bloquer l'ancien port si changé
sudo ufw deny 22/tcp

# Activer le pare-feu
sudo ufw enable

# Vérifier les règles
sudo ufw status
```

**Firewalld (RHEL/CentOS) :**

```bash
# Ajouter le nouveau port
sudo firewall-cmd --permanent --add-port=2222/tcp

# Retirer l'ancien port
sudo firewall-cmd --permanent --remove-service=ssh

# Recharger
sudo firewall-cmd --reload
```

### 3. Fail2Ban - Protection contre les attaques par force brute

```bash
# Installation
sudo apt install fail2ban

# Configuration SSH
sudo nano /etc/fail2ban/jail.local
```

Ajoutez :

```ini
[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
findtime = 600
```

```bash
# Redémarrer Fail2Ban
sudo systemctl restart fail2ban

# Vérifier les bannissements
sudo fail2ban-client status sshd
```

### 4. Surveillance et audit

**Vérifier les connexions actives :**

```bash
# Utilisateurs connectés
who
w

# Historique des connexions
last
last -a | grep still  # Connexions actives

# Tentatives de connexion échouées
sudo lastb
```

**Surveiller les logs :**

```bash
# En temps réel
sudo tail -f /var/log/auth.log

# Avec systemd
sudo journalctl -u sshd -f

# Analyser les tentatives de connexion
sudo grep "Failed password" /var/log/auth.log
sudo grep "Accepted publickey" /var/log/auth.log
```

### 5. Gestion des clés SSH

**Bonnes pratiques :**
- ✅ Utilisez des phrases de passe fortes pour vos clés
- ✅ Rotation régulière des clés (tous les 6-12 mois)
- ✅ Une clé par service (GitHub, serveur1, serveur2, etc.)
- ✅ Sauvegardez vos clés dans un endroit sûr (coffre-fort de mots de passe)
- ✅ Révoquez immédiatement les clés compromises
- ❌ Ne partagez jamais vos clés privées
- ❌ Ne commitez jamais de clés dans Git

**Révoquer une clé compromise :**

```bash
# Sur le serveur, éditez authorized_keys
nano ~/.ssh/authorized_keys
# Supprimez la ligne contenant la clé compromise

# Ou automatiquement
sed -i '/clé_compromise/d' ~/.ssh/authorized_keys
```

### 6. Authentification à deux facteurs (2FA)

```bash
# Installation de Google Authenticator
sudo apt install libpam-google-authenticator

# Configuration pour l'utilisateur
google-authenticator

# Modifier PAM
sudo nano /etc/pam.d/sshd
# Ajouter : auth required pam_google_authenticator.so

# Modifier sshd_config
sudo nano /etc/ssh/sshd_config
# Définir : ChallengeResponseAuthentication yes

# Redémarrer SSH
sudo systemctl restart sshd
```

### 7. Checklist de sécurité

- [ ] Port SSH modifié (différent de 22)
- [ ] Authentification par clés uniquement
- [ ] Connexion root désactivée
- [ ] Fail2Ban installé et configuré
- [ ] Pare-feu configuré
- [ ] Logs surveillés régulièrement
- [ ] Clés SSH avec phrases de passe
- [ ] Permissions correctes sur ~/.ssh
- [ ] Mise à jour régulière du système
- [ ] Algorithmes cryptographiques forts
- [ ] 2FA activé (optionnel mais recommandé)

---

## ❓ FAQ et dépannage

### Problème : "Permission denied (publickey)"

**Causes possibles et solutions :**

1. **Permissions incorrectes :**
   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/id_ed25519
   chmod 644 ~/.ssh/id_ed25519.pub

   # Sur le serveur
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

2. **Clé publique non présente sur le serveur :**
   ```bash
   # Vérifier sur le serveur
   cat ~/.ssh/authorized_keys

   # Ajouter votre clé
   ssh-copy-id utilisateur@serveur.com
   ```

3. **Mauvaise clé utilisée :**
   ```bash
   # Spécifier explicitement la clé
   ssh -i ~/.ssh/id_ed25519 utilisateur@serveur.com

   # Mode verbose pour diagnostic
   ssh -v utilisateur@serveur.com
   ```

4. **Configuration serveur :**
   ```bash
   # Sur le serveur, vérifier sshd_config
   sudo grep "PubkeyAuthentication" /etc/ssh/sshd_config
   # Doit être : PubkeyAuthentication yes
   ```

### Problème : "Connection timed out"

**Solutions :**

1. **Vérifier la connectivité réseau :**
   ```bash
   ping serveur.com
   ```

2. **Vérifier que le port est ouvert :**
   ```bash
   telnet serveur.com 22
   # Ou
   nc -zv serveur.com 22
   # Ou
   nmap -p 22 serveur.com
   ```

3. **Vérifier le pare-feu local :**
   ```bash
   # Linux
   sudo iptables -L
   sudo ufw status

   # Windows
   # Panneau de configuration → Pare-feu Windows
   ```

4. **Augmenter le timeout :**
   ```bash
   # Dans ~/.ssh/config
   Host *
       ConnectTimeout 30
   ```

### Problème : "Host key verification failed"

**Cause :** L'empreinte du serveur a changé (réinstallation, attaque MITM)

**Solution :**

```bash
# Supprimer l'ancienne empreinte
ssh-keygen -R serveur.com

# Ou pour un port spécifique
ssh-keygen -R [serveur.com]:2222

# Ou éditer manuellement
nano ~/.ssh/known_hosts
```

### Problème : "Too many authentication failures"

**Cause :** L'agent SSH essaie trop de clés

**Solutions :**

```bash
# Option 1 : Spécifier la clé exacte
ssh -o IdentitiesOnly=yes -i ~/.ssh/cle_correcte utilisateur@serveur.com

# Option 2 : Vider l'agent SSH
ssh-add -D
ssh-add ~/.ssh/cle_correcte

# Option 3 : Configuration permanente dans ~/.ssh/config
Host serveur
    IdentitiesOnly yes
    IdentityFile ~/.ssh/cle_correcte
```

### Problème : L'agent SSH ne fonctionne pas

**Solutions :**

```bash
# Vérifier si l'agent tourne
ps aux | grep ssh-agent

# Redémarrer l'agent
eval "$(ssh-agent -s)"

# Ajouter les clés
ssh-add ~/.ssh/id_ed25519

# Lister les clés chargées
ssh-add -l

# Problème de démarrage automatique ?
# Ajouter dans ~/.bashrc ou ~/.zshrc
if [ -z "$SSH_AUTH_SOCK" ]; then
   eval "$(ssh-agent -s)"
   ssh-add ~/.ssh/id_ed25519 2>/dev/null
fi
```

### Problème : "Could not open a connection to your authentication agent"

**Solution :**

```bash
# Démarrer l'agent
eval "$(ssh-agent)"
```

### Comment vérifier la version de SSH ?

```bash
ssh -V
# Exemple : OpenSSH_9.0p1, OpenSSL 3.0.2

# Version du serveur
ssh -v serveur.com 2>&1 | grep "remote software version"
```

### Comment tester une connexion SSH ?

```bash
# Test basique
ssh -T git@github.com

# Avec verbosité maximale
ssh -vvv utilisateur@serveur.com

# Tester la configuration sans se connecter
ssh -G utilisateur@serveur.com
```

### Problème : Session SSH se déconnecte fréquemment

**Solutions :**

```bash
# Dans ~/.ssh/config
Host *
    ServerAliveInterval 30
    ServerAliveCountMax 5
    TCPKeepAlive yes
```

---

## 📚 Ressources complémentaires

### Documentation officielle

- [OpenSSH Official Documentation](https://www.openssh.com/manual.html)
- [SSH.com - SSH Academy](https://www.ssh.com/academy/ssh)
- [GitHub SSH Guide](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [ArchWiki - SSH](https://wiki.archlinux.org/title/SSH)
- [Ubuntu SSH Documentation](https://help.ubuntu.com/community/SSH)

### Outils recommandés

**Clients SSH :**
- **[Termius](https://termius.com/)** : Client SSH multiplateforme avec synchronisation cloud
- **[MobaXterm](https://mobaxterm.mobatek.net/)** : Terminal avancé pour Windows avec X11
- **[PuTTY](https://www.putty.org/)** : Client SSH classique pour Windows
- **[iTerm2](https://iterm2.com/)** : Terminal avancé pour macOS

**Outils de configuration :**
- **[SSH Config Generator](https://www.sshconfig.com/)** : Générateur de configuration SSH en ligne
- **[SSH Config Editor](https://github.com/cloudflare/sshconfig)** : Outil CLI pour gérer ~/.ssh/config

**Sécurité et monitoring :**
- **[Fail2Ban](https://www.fail2ban.org/)** : Protection contre les attaques par force brute
- **[OSSEC](https://www.ossec.net/)** : Système de détection d'intrusion
- **[Denyhosts](https://github.com/denyhosts/denyhosts)** : Alternative à Fail2Ban
- **[SSHGuard](https://www.sshguard.net/)** : Protection temps réel contre les attaques SSH

**Gestion des clés :**
- **[ssh-audit](https://github.com/arthepsy/ssh-audit)** : Audit de configuration SSH
- **[ssh-keyscan](https://man.openbsd.org/ssh-keyscan)** : Scanner de clés SSH (inclus avec OpenSSH)
- **[Keychain](https://www.funtoo.org/Keychain)** : Gestionnaire d'agent SSH

### Tutoriels et guides avancés

- [DigitalOcean - SSH Essentials](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys)
- [Hardening SSH - Mozilla Guidelines](https://infosec.mozilla.org/guidelines/openssh)
- [SSH Tunneling Guide](https://www.ssh.com/academy/ssh/tunneling)

### Commandes utiles

Consultez le fichier [ssh_commands.md](ssh_commands.md) pour une référence complète des commandes SSH les plus utiles.

### Exemples pratiques

Consultez le fichier [EXAMPLES.md](EXAMPLES.md) pour des cas d'usage concrets et des scénarios d'utilisation.

---

## 📄 Licence

Ce tutoriel est distribué sous licence [MIT](LICENSE).

Vous êtes libre de :
- ✅ Utiliser ce tutoriel à des fins personnelles ou commerciales
- ✅ Modifier et adapter le contenu
- ✅ Distribuer et partager

---

## 👤 À propos de l'auteur

**Nicolas Deoux**
📧 Email : NDXDev@gmail.com
🔗 LinkedIn : [Nicolas Deoux](https://www.linkedin.com/in/nicolas-deoux-ab295980/)

---

## 🔄 Historique des versions

- **v1.0.0** (14 novembre 2025) : Version initiale du tutoriel

---

> **💡 Conseil final :** La sécurité SSH est un processus continu. Restez informé des dernières vulnérabilités, mettez à jour régulièrement vos systèmes, et n'hésitez pas à consulter la documentation officielle pour des cas d'usage spécifiques.

> **⚠️ Avertissement :** Testez toujours les configurations dans un environnement de développement avant de les appliquer en production. Une mauvaise configuration SSH peut vous verrouiller hors de votre serveur !

