# Homelab — Galaxy

Repo chapeau, public : documente l'architecture globale. Le code (composes, scripts, Terraform) et la documentation
d'exploitation vivent dans [`galaxy`](https://github.com/matxsu/galaxy), privé.

> **Statut au 2026-10-10.** L'architecture k3s sur Raspberry Pi est abandonnée : Galaxy tourne désormais sur
> Proxmox (conteneurs LXC et une VM, services en Docker Compose) et sur un Pi dédié au DNS et à la supervision.

## Architecture

```
                Internet
                   │  Cloudflare Tunnel (un seul service public)
                   ▼
  ┌──────────────── Corulag — Proxmox VE ────────────────┐      ┌──── Kuat — Raspberry Pi 5 ────┐
  │ alderaan  Traefik · Vaultwarden · Portainer          │◄────►│ Technitium DNS (split-horizon) │
  │ kamino    Authentik (SSO, MFA)                       │      │ Uptime Kuma · Signal · Homarr  │
  │ scarif    Jellyfin · Seerr · *arr · qBittorrent+VPN  │      │ Prometheus · Grafana           │
  │ endor     Pelican Panel + Wings (jeux)               │      └────────────────────────────────┘
  └──────────────────────────────────────────────────────┘
          accès distant : Tailscale · identité : Authentik
```

| Machine | Rôle | Codename |
|---|---|---|
| Dell OptiPlex 3000 Micro (i5-12500T, 16 Go) | hyperviseur Proxmox | Corulag |
| Raspberry Pi 5 (16 Go, NVMe) | DNS, supervision, tableau de bord | Kuat |
| HP EliteDesk 800 G5 Micro (i5-9500T, 16 Go) | futur stockage et sauvegardes | Coruscant (à installer) |

| Invité | Type | Services |
|---|---|---|
| alderaan | LXC non privilégié | Traefik (wildcard TLS en DNS-01), Vaultwarden, Portainer |
| kamino | LXC non privilégié | Authentik + PostgreSQL |
| scarif | LXC privilégié (iGPU QuickSync) | Jellyfin, Seerr, Sonarr, Radarr, Prowlarr, Bazarr, Recyclarr, qBittorrent derrière Gluetun |
| endor | VM Ubuntu | Pelican Panel, serveurs Minecraft |

## Principes

- **Tailscale d'abord** : un seul service est publié sur Internet, par Cloudflare Tunnel.
- **Filtrage avant authentification** : tout passe par Traefik, filtré par réseau puis par Authentik (MFA obligatoire).
- **La supervision ne vit pas sur ce qu'elle surveille** : Uptime Kuma, Signal et Grafana sont sur Kuat.
- **Pas de mise à jour automatique des conteneurs** : sauvegarde vzdump, mise à jour à la main, un service à la fois.
- **Nommage** : machines et invités portent des noms de planètes de Star Wars.

## Matériel

### Réseau
- UniFi Cloud Gateway Fiber, switch UniFi 8 ports PoE, AP UniFi WiFi 7
- VLAN séparés (serveurs, IoT, postes)

### Rack
Rack 10" imprimé en 3D — 3 slots HDD SATA + 5 slots SSD SATA. Designs dans `hardware/rack-mounts/`.
Carte M.2 → SATA (JMB585) achetée pour Coruscant.

## Outils annexes
- **rackula** — planification visuelle de rack (self-hosted, `ghcr.io/rackulalives/rackula`), hors de ce repo.

## Structure de ce repo
```
hardware/      — fiches matériel, designs d'impression 3D (racks, supports)
assets/        — icônes et ressources pour dashboards (Homarr, etc.)
```
