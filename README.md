# Souveraineté maison

Mon infrastructure numérique personnelle, auto-hébergée sur un Raspberry Pi 5, documentée pour être comprise et reproduite.

## Pourquoi ce repo

Je veux être souverain numériquement, autant que possible. Je ne travaille pas pour l'État, je n'ai aucune prise sur les grandes décisions. L'endroit où j'ai le plus d'influence, c'est chez moi. Alors c'est là que j'agis : chaque service que j'héberge sous mon toit est un service de moins confié à une plateforme qui vit de mes données.

Mais faire ça dans son coin ne sert à rien si personne ne le voit. Ce repo existe pour rendre ma démarche visible et reproductible. Si une seule personne le lit et se dit « en fait, je pourrais le faire aussi », il aura rempli son rôle.

Il sert deux publics :

- **Moi**, quand mon serveur plantera (il plantera) et qu'il faudra tout redéployer sans paniquer.
- **Toi**, si tu es curieux de comprendre ce qui tourne ici, combien ça coûte vraiment, et par où commencer.

## Le matériel

Tout tourne sur une seule machine, posée dans un placard, silencieuse, qui consomme moins qu'une ampoule :

| Composant | Détail | Prix approximatif |
|---|---|---|
| Raspberry Pi 5 | 16 Go de RAM, ARM 64 bits | 130 € |
| 2× SSD NVMe 1 To | montés en RAID1 (miroir) : si un disque meurt, l'autre a tout | 150 € |
| HAT NVMe double | la carte qui connecte les deux SSD au Pi | 40 € |
| Alimentation 27 W, boîtier, microSD de boot | | 50 € |
| **Total matériel** | investissement unique | **~370 €** |

Les coûts récurrents, honnêtement comptés :

- Électricité : environ 10 W en moyenne, soit à peu près 90 kWh et **20 à 25 € par an**.
- Nom de domaine : **~10 € par an**.
- Tunnel Cloudflare Zero Trust (pour exposer les services sans ouvrir ma box) : **0 €**.

Soit environ 3 € par mois une fois le matériel amorti. Un seul abonnement cloud coûte plus cher.

## Ce qui tourne chez moi

Chaque service ci-dessous est un logiciel libre, à 0 € de licence. Le « coût réel » est ma part d'électricité ; ce que je liste, c'est ce que je ne paie plus (ou ne paierais pas) ailleurs.

### Reprendre la main sur mes données

| Service | À quoi il me sert | Ce qu'il remplace | Coût évité |
|---|---|---|---|
| [Nextcloud](https://nextcloud.com) (All-in-One) | mes fichiers, photos, agenda, contacts, édition de documents avec Collabora | Google Drive, Google Photos, Google Docs | Google One : 2 à 10 €/mois |
| [Umami](https://umami.is) | statistiques de mes sites, sans cookie, sans profilage | Google Analytics | la vie privée de mes visiteurs |

### Communiquer sans intermédiaire

| Service | À quoi il me sert | Ce qu'il remplace | Coût évité |
|---|---|---|---|
| [Listmonk](https://listmonk.app) | mes newsletters | Mailchimp | dès 13 €/mois |
| [Mautic](https://mautic.org) | marketing automation, relances | HubSpot Marketing | dès 15 €/mois |
| [EspoCRM](https://espocrm.com) | suivi de mes contacts et opportunités | Salesforce, Pipedrive | dès 15 €/mois/utilisateur |

### Gérer mon activité

| Service | À quoi il me sert | Ce qu'il remplace | Coût évité |
|---|---|---|---|
| [Invoice Ninja](https://invoiceninja.com) | devis et factures | QuickBooks, Henrri | dès 10 €/mois |
| [Hi.Events](https://hi.events) | billetterie de mes événements | Eventbrite | commission sur chaque billet |
| [Webstudio](https://webstudio.is) | création visuelle de mes sites web | Webflow, Wix | dès 14 €/mois |

### Automatiser

| Service | À quoi il me sert | Ce qu'il remplace | Coût évité |
|---|---|---|---|
| [n8n](https://n8n.io) | mes automatisations (le ciment entre tous les autres services) | Zapier, Make | dès 20 €/mois |
| [Dagster](https://dagster.io) | mes pipelines de données (projet perso de scraping de concertations publiques) | un cloud data quelconque | variable |

### Faire tourner et surveiller tout ça

| Service | À quoi il me sert | Ce qu'il remplace | Coût évité |
|---|---|---|---|
| [Coolify](https://coolify.io) | mon PaaS auto-hébergé : je déploie tous les services ci-dessus depuis une interface web | Heroku, Vercel | dès 5 €/mois/app |
| [Homepage](https://gethomepage.dev) | la page d'accueil qui rassemble tout (config dans ce repo) | rien, c'est le tableau de bord de la maison | |
| [Uptime Kuma](https://github.com/louislam/uptime-kuma) | il me prévient quand quelque chose tombe | UptimeRobot, Pingdom | dès 7 €/mois |
| [Grafana](https://grafana.com) + [Prometheus](https://prometheus.io) | courbes de température, RAM, disque, containers | Datadog | dès 15 €/hôte/mois |
| [Cloudflared](https://developers.cloudflare.com/cloudflare-one/) | le tunnel qui expose mes services au monde sans ouvrir un seul port sur ma box | une IP fixe, un reverse proxy exposé | |
| [Supabase](https://supabase.com) | backend (base de données, auth, API) pour mes projets de dev | Firebase | dès 25 €/mois |

Tout est en images arm64, vérifiées : ce sont exactement celles qui tournent sur mon Pi au moment où j'écris ces lignes.

## Par où commencer si tu pars de zéro

Ne fais pas tout d'un coup. Voici l'ordre que je recommanderais si je recommençais :

1. **Le matériel.** Un Raspberry Pi 4 ou 5 avec 8 Go suffit largement pour débuter. Un seul SSD fait l'affaire au début ; le RAID viendra quand tu auras des données qui comptent.
2. **Docker.** Installe Docker et Docker Compose. C'est la seule compétence vraiment indispensable ici : tout le reste, ce sont des fichiers YAML qu'on copie et qu'on adapte.
3. **Un premier service sans enjeu : Uptime Kuma.** Il s'installe en deux minutes (`docker-compose/uptime-kuma.yml`), il est joli, et il t'apprendra les bases sans risque : si tu le casses, tu ne perds rien.
4. **Homepage.** Ta page d'accueil (`docker-compose/homepage.yml` + le dossier `homepage/`). À partir de là, ton serveur a un visage.
5. **Un nom de domaine et le tunnel Cloudflare.** C'est le moment charnière : tes services deviennent accessibles depuis l'extérieur, proprement, sans toucher à ta box. Tout est expliqué dans `docs/cloudflare-tunnel.md`.
6. **Le premier service qui compte : Nextcloud.** C'est le plus gros morceau mais c'est celui qui change la vie : tes fichiers reviennent chez toi.
7. **Les sauvegardes, avant d'aller plus loin.** À partir du moment où ton serveur héberge des données réelles, lis `docs/sauvegardes.md`. Un serveur sans sauvegarde n'est pas de la souveraineté, c'est de l'imprudence.
8. **Coolify, quand gérer les fichiers compose à la main devient pénible.** Ensuite seulement, ajoute les services métier (CRM, newsletters, facturation) selon tes besoins réels, pas selon ce qui est à la mode.

## Structure du repo

```
souverainete-maison/
├── README.md              <- tu es ici
├── .env.example           <- toutes les variables, documentées, sans aucune vraie valeur
├── docker-compose/        <- un fichier par service, commenté, images arm64
├── homepage/              <- ma config Homepage, organisée par intention
├── docs/
│   ├── installation.md    <- de la carte SD au premier container
│   ├── cloudflare-tunnel.md
│   └── sauvegardes.md
└── CHANGELOG.md           <- le journal de bord de cette aventure
```

## Ce que tu ne trouveras pas ici

Ce repo est public, et il le restera. Donc :

- Aucun domaine réel : partout, `exemple.fr` et ses sous-domaines.
- Aucun mot de passe, token ou clé : uniquement `.env.example` avec des valeurs `CHANGEME`.
- Aucune IP, aucun identifiant de tunnel.

Si tu forkes ce repo pour ta propre installation, garde ce réflexe : ton `.env` réel ne doit jamais approcher un `git add`.

## Licence

MIT. Prends, adapte, redistribue. C'est le but.
