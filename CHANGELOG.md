# Journal de bord

Pas un fichier de versions : le récit daté de cette infrastructure, ses avancées et ses leçons. Les entrées les plus récentes en premier.

## 2026-07-14

Naissance de ce repo. L'infra, elle, existait déjà : un Raspberry Pi 5 (16 Go, 2× NVMe 1 To en RAID1) qui fait tourner une quinzaine de services via Coolify, exposés par un tunnel Cloudflare. Ce qui manquait, c'était la partie « visible et reproductible » de la démarche. C'est fait : intention dans le README, un compose par service, la config Homepage, et les notes d'installation, de tunnel et de sauvegarde.

Prochaine étape concrète : déployer Homepage (documenté ici avant même d'être en production chez moi, pour une fois que la doc précède le code).

## 2026-07 (début du mois)

Grosse frayeur : après un redémarrage déclenché depuis l'outil de déploiement, un service est reparti sur des volumes Docker neufs et vides. Sites disparus en apparence. Cause : un bug de nommage des volumes dans l'orchestrateur. Données finalement intactes dans les anciens volumes, tout est rentré dans l'ordre. Leçons tirées, désormais gravées dans `docs/sauvegardes.md` : le RAID ne protège pas des bugs, seules les sauvegardes testées comptent.

## 2026-05

Mise en place du monitoring sérieux : Prometheus, Grafana, node-exporter, cAdvisor, AlertManager. Le Pi a maintenant des courbes de température et des alertes. On ne gère bien que ce qu'on mesure.

## Avant

Accumulation progressive des services : Nextcloud AIO pour sortir de Google Drive, n8n pour automatiser, Uptime Kuma pour dormir tranquille, puis les outils d'activité (EspoCRM, Listmonk, Mautic, Invoice Ninja, Hi.Events) et les outils de création (Webstudio, Supabase). Coolify comme chef d'orchestre. Chaque ajout suivait un besoin réel, jamais l'inverse.
