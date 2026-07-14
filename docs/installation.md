# Installation : de la carte SD au premier container

Mes notes pour remonter la machine de zéro. C'est le chemin que je referais en cas de crash total, dans cet ordre.

## 1. Le système

1. Flasher **Raspberry Pi OS Lite 64 bits** sur la microSD avec Raspberry Pi Imager. Dans les options de l'imager : activer SSH, définir l'utilisateur, renseigner le Wi-Fi si besoin (mais préférer l'Ethernet pour un serveur).
2. Premier boot, connexion SSH, puis les classiques :

```bash
sudo apt update && sudo apt full-upgrade -y
sudo raspi-config   # locale, timezone
```

3. Durcir un minimum le SSH : authentification par clé uniquement, pas de mot de passe.

```bash
# Depuis ton poste :
ssh-copy-id utilisateur@ip-du-pi
# Puis sur le Pi, dans /etc/ssh/sshd_config :
#   PasswordAuthentication no
sudo systemctl restart ssh
```

## 2. Le stockage en RAID1

Deux SSD NVMe identiques sur un HAT double, en miroir logiciel avec mdadm. Si un disque meurt, rien n'est perdu.

```bash
sudo apt install -y mdadm

# ATTENTION : détruit tout sur les deux disques.
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/nvme0n1 /dev/nvme1n1

sudo mkfs.ext4 /dev/md0
sudo mkdir -p /mnt/raid
```

Monter au boot via `/etc/fstab` (utiliser l'UUID donné par `blkid`), puis rendre la config mdadm persistante :

```bash
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u
```

Vérifier la santé du miroir à tout moment : `cat /proc/mdstat` (les deux disques doivent afficher `[UU]`).

Le système, lui, reste sur la microSD : elle ne contient rien d'irremplaçable, tout ce qui compte vit sur le RAID.

## 3. Docker

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
# Se déconnecter/reconnecter pour que le groupe prenne effet
```

Point important : déplacer les données Docker (images, volumes) sur le RAID, sinon tout vit sur la microSD. Créer `/etc/docker/daemon.json` :

```json
{
  "data-root": "/mnt/raid/docker"
}
```

Puis `sudo systemctl restart docker`.

## 4. Le premier service

```bash
git clone https://github.com/CHANGEME/souverainete-maison.git
cd souverainete-maison
cp .env.example .env
# Éditer .env : remplacer chaque CHANGEME
cd docker-compose
docker compose -f uptime-kuma.yml --env-file ../.env up -d
```

Uptime Kuma répond sur `http://ip-du-pi:3001`. À partir de là, chaque service du dossier `docker-compose/` suit le même schéma.

## 5. La suite

- Exposer les services proprement : [cloudflare-tunnel.md](cloudflare-tunnel.md)
- Et surtout, avant de confier des données réelles à la machine : [sauvegardes.md](sauvegardes.md)

## Note sur Coolify

À partir d'une dizaine de services, gérer les compose à la main devient fastidieux. J'ai fini par tout piloter avec [Coolify](https://coolify.io), un PaaS auto-hébergé : interface web, déploiements, variables d'environnement, logs. Son installation officielle :

```bash
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | sudo bash
```

Les fichiers compose de ce repo restent la référence : ce sont les mêmes définitions, que Coolify se contente d'orchestrer.
