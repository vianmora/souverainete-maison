# Stratégie de sauvegarde

Un serveur à la maison sans sauvegarde, ce n'est pas de la souveraineté, c'est juste un point de défaillance unique qu'on a déplacé de chez Google à chez soi. Voici comment je raisonne.

## La règle 3-2-1, adaptée à la maison

Trois copies des données, sur deux supports différents, dont une hors du domicile :

1. **Copie 1 : les données vivantes**, sur le RAID1. Le miroir protège de la panne d'UN disque. C'est tout. Le RAID n'est pas une sauvegarde : un `rm -rf`, un ransomware ou un bug (voir plus bas) se répliquent instantanément sur les deux disques.
2. **Copie 2 : les archives locales**, sur un support distinct du RAID (disque USB externe, ou à défaut un dossier dédié du RAID en attendant mieux). Automatisées, datées, avec rétention.
3. **Copie 3 : hors site**, chiffrée. Chez un proche (un autre Pi chez un ami, chacun héberge les sauvegardes de l'autre : c'est la version conviviale du hors-site), ou sur un stockage objet loué. Oui, ça peut être un cloud : une archive chiffrée chez un prestataire ne lui apprend rien sur moi, c'est très différent de lui confier mes données en clair.

## Quoi sauvegarder

Par ordre d'importance :

1. **Les volumes Docker** : c'est là que vivent toutes les données (bases, fichiers, configs). Avec `data-root` sur le RAID, tout est sous `/mnt/raid/docker/volumes/`.
2. **Les bases de données, via dump** plutôt que par copie brute des fichiers : un `pg_dump` pris à chaud est cohérent, une copie de `/var/lib/postgresql/data` en cours d'écriture ne l'est pas forcément.
3. **Les secrets** : le `.env`, la clé de chiffrement n8n, l'APP_KEY d'Invoice Ninja. Une base restaurée sans sa clé de chiffrement est un coffre-fort sans combinaison. À stocker dans un gestionnaire de mots de passe, pas sur le serveur lui-même.
4. **Ce repo** : la définition de l'infra. C'est sa raison d'être.

Ce qui n'a pas besoin de sauvegarde : le système sur la microSD (réinstallable en 20 minutes via `docs/installation.md`) et les images Docker (retéléchargeables).

## Comment

### Les dumps de bases, chaque nuit

Un cron (ou un workflow n8n, tant qu'à l'avoir) qui dumpe chaque base :

```bash
#!/usr/bin/env bash
# /mnt/raid/scripts/backup-db.sh : dumps quotidiens, rétention 14 jours
set -euo pipefail
DEST=/mnt/raid/backups/db/$(date +%F)
mkdir -p "$DEST"

docker exec n8n-db pg_dump -U n8n n8n | gzip > "$DEST/n8n.sql.gz"
docker exec umami-db pg_dump -U umami umami | gzip > "$DEST/umami.sql.gz"
docker exec listmonk-db pg_dump -U listmonk listmonk | gzip > "$DEST/listmonk.sql.gz"
docker exec espocrm-db sh -c 'mariadb-dump -uroot -p"$MARIADB_ROOT_PASSWORD" espocrm' | gzip > "$DEST/espocrm.sql.gz"

# Rétention : supprimer les dossiers de plus de 14 jours
find /mnt/raid/backups/db -maxdepth 1 -type d -mtime +14 -exec rm -rf {} +
```

### Les volumes de fichiers, avec restic

[restic](https://restic.net) fait exactement ce qu'il faut : incrémental, chiffré de bout en bout, déduplication, rétention. Le même outil gère la copie locale et la copie hors site.

```bash
# Initialiser un dépôt (une fois) ; le mot de passe restic est un
# secret de plus à mettre au gestionnaire de mots de passe.
restic -r /mnt/backup-usb/restic init

# Sauvegarde (cron quotidien)
restic -r /mnt/backup-usb/restic backup /mnt/raid/docker/volumes /mnt/raid/backups/db

# Rétention
restic -r /mnt/backup-usb/restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 6 --prune
```

Pour la copie hors site, le même restic pointe vers un second dépôt distant (SFTP vers le Pi d'un ami, ou un stockage objet S3-compatible). Tout part chiffré : le destinataire ne peut rien lire.

### Nextcloud AIO : son propre système

AIO embarque une sauvegarde intégrée (basée sur BorgBackup, même philosophie que restic) configurable depuis son interface d'admin. Je la pointe vers le disque de sauvegarde et je laisse restic embarquer le résultat vers le hors-site.

## Tester la restauration

Une sauvegarde jamais restaurée est une hypothèse, pas une sauvegarde. Deux vérifications :

- régulièrement : `restic check`, et restaurer un fichier au hasard (`restic restore latest --target /tmp/test --include <fichier>`)
- une fois par an, l'exercice grandeur nature : remonter un service complet (Uptime Kuma est parfait pour ça) sur une machine vierge, uniquement à partir de ce repo et des sauvegardes.

## Le jour où ça a failli mal tourner

Ce qui m'a convaincu de prendre tout ça au sérieux : en juillet 2026, à la suite d'un enchaînement malheureux dans mon outil de déploiement, un service a redémarré sur des volumes neufs et vides au lieu de ses vrais volumes. Tous mes sites semblaient avoir disparu. Les données étaient en réalité toujours là, dans les anciens volumes, et tout a été récupéré. Mais pendant une heure, la seule chose qui me séparait de la perte totale, c'était l'existence des sauvegardes. On ne fait pas des sauvegardes parce qu'on est pessimiste, on en fait parce que les systèmes sont des systèmes.
