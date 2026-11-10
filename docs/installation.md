# Installation : étapes principales

Résumé des étapes pour reproduire l'infrastructure. Les adresses en `192.168.100.x` et tous les mots de passe sont à remplacer par vos propres valeurs. **Ne jamais publier de vrai mot de passe.**

## 1. VMware

1. Éditeur de réseau virtuel :
   - **VMnet8** en NAT, sous-réseau 192.168.150.0/24, DHCP activé
   - **VMnet1** en Host-Only, sous-réseau 192.168.100.0/24, DHCP désactivé
2. Créer les deux VM Ubuntu Server 26.04 LTS :
   - **VM Web** : 4 Go RAM, 2 vCPU, 30 Go, cartes réseau sur VMnet8 et VMnet1
   - **VM BDD** : 4 Go RAM, 1 vCPU, 20 Go, carte réseau sur VMnet1 uniquement

## 2. Réseau des VM

IP fixes sur VMnet1 avec Netplan : voir [config/netplan-web.yaml.example](../config/netplan-web.yaml.example) et [config/netplan-bdd.yaml.example](../config/netplan-bdd.yaml.example).

```bash
sudo netplan apply
ip a
ping 192.168.100.x   # vérifier que les deux VM se joignent
```

## 3. VM BDD : MariaDB

```bash
sudo apt update && sudo apt install -y mariadb-server
sudo mysql_secure_installation
```

Écouter sur l'interface Host-Only (`bind-address` dans `/etc/mysql/mariadb.conf.d/50-server.cnf`), puis créer la base et l'utilisateur limité à l'IP de la VM Web : voir [config/mariadb-glpi.sql.example](../config/mariadb-glpi.sql.example).

```bash
sudo systemctl restart mariadb
```

## 4. VM Web : Apache, PHP, GLPI

```bash
sudo apt update
sudo apt install -y apache2 php libapache2-mod-php php-mysql php-curl php-gd php-intl php-mbstring php-xml php-zip php-bz2 php-ldap
sudo a2enmod rewrite
```

Télécharger GLPI 11.0.6 depuis le dépôt officiel (github.com/glpi-project/glpi/releases), l'extraire dans `/var/www/`, donner les droits à `www-data`, puis activer le virtual host : voir [config/glpi-apache.conf.example](../config/glpi-apache.conf.example).

```bash
sudo a2ensite glpi.conf && sudo systemctl reload apache2
```

## 5. Assistant d'installation GLPI

Depuis le navigateur de l'hôte (adresse NAT de la VM Web) :
- serveur SQL : IP Host-Only de la VM BDD
- utilisateur : `glpi_user`, base : `glpi`

Après l'installation : changer les mots de passe des comptes par défaut et supprimer `install/install.php`.

## 6. Configuration fonctionnelle

Entités, groupes, profil Technicien Filiale, gabarits d'inventaire, états du cycle de vie, SLA, formulaires et base de connaissances : voir le [README](../README.md) et le support de soutenance.
