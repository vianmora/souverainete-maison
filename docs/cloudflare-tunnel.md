# Le tunnel Cloudflare Zero Trust

Comment mes services sont accessibles depuis Internet alors que ma box n'a aucun port ouvert.

## Le principe

Le schéma classique de l'auto-hébergement, c'est : ouvrir les ports 80/443 sur sa box, les rediriger vers le serveur, et exposer un reverse proxy au monde entier. Ça marche, mais ça suppose une IP fixe (ou du DNS dynamique), et ça met une porte d'entrée directe chez soi.

Le tunnel inverse la logique : un petit container (`cloudflared`) établit une connexion **sortante** permanente vers Cloudflare. Quand quelqu'un visite `service.exemple.fr`, Cloudflare reçoit la requête et la fait redescendre par ce tunnel jusqu'au bon service local.

Concrètement :

- aucun port ouvert sur la box, rien à configurer côté routeur
- pas besoin d'IP fixe
- HTTPS automatique, certificats gérés par Cloudflare
- mon IP résidentielle n'apparaît nulle part

Le compromis à connaître : Cloudflare voit passer le trafic en clair entre son réseau et le visiteur. C'est un tiers de confiance, donc une concession sur la souveraineté totale. Je l'assume pour l'instant : c'est le meilleur rapport simplicité/sécurité pour débuter, et c'est réversible (l'alternative auto-hébergée, c'est un VPS à soi avec un tunnel WireGuard).

## Mise en place

1. Un compte Cloudflare (gratuit), et ton domaine dont les DNS pointent chez eux.
2. Dashboard [Zero Trust](https://one.dash.cloudflare.com) > Networks > Tunnels > Create a tunnel (type "Cloudflared").
3. Cloudflare affiche un token : c'est **le secret** du tunnel. Le mettre dans le `.env` (`CLOUDFLARE_TUNNEL_TOKEN`), jamais ailleurs.
4. Lancer le container :

```bash
docker compose -f docker-compose/cloudflared.yml --env-file .env up -d
```

5. Dans le dashboard, onglet "Public Hostname" du tunnel : associer chaque sous-domaine à un service local.

| Sous-domaine | Service local |
|---|---|
| `maison.exemple.fr` | `http://localhost:3000` (Homepage) |
| `status.exemple.fr` | `http://localhost:3001` (Uptime Kuma) |
| `n8n.exemple.fr` | `http://localhost:5678` (n8n) |
| `nextcloud.exemple.fr` | `http://localhost:11000` (Nextcloud AIO) |

Chaque entrée crée automatiquement l'enregistrement DNS. Le service est en ligne dans la seconde.

## Protéger les interfaces d'administration

Tout n'a pas vocation à être public. Zero Trust > Access permet de mettre une authentification devant un sous-domaine avant même que la requête n'atteigne le Pi : je m'en sers pour Coolify, Grafana ou Prometheus. Une policy "email se terminant par mon adresse" + code à usage unique, et l'interface d'admin devient invisible pour le reste du monde. Gratuit jusqu'à 50 utilisateurs.

## Cas particulier : Nextcloud AIO

Nextcloud AIO derrière un tunnel demande deux choses :

- `APACHE_PORT=11000` dans le compose (déjà fait dans `docker-compose/nextcloud-aio.yml`) : Apache écoute en HTTP simple, le HTTPS vient du tunnel.
- Dans le Public Hostname du tunnel, pointer vers `http://localhost:11000`.

La doc officielle AIO a une page dédiée au reverse proxy qui couvre le cas Cloudflare Tunnel.
