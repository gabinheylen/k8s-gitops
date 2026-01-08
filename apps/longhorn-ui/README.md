# Longhorn UI

Ce dossier contient la configuration de l'interface utilisateur Longhorn UI pour GitOps avec Argocd.

## Fichiers

- `deployment.yaml` - Deployment UI (2 réplicas avec anti-affinité)
- `service.yaml` - Service frontend (port 80)
- `ingress.yaml` - Ingress avec TLS et middleware local-only
- `serviceaccount.yaml` - ServiceAccount dédié

## Configuration

- **Réplicas**: 2 avec anti-affinité entre nœuds
- **Resources**: Limites et requêtes définies pour optimiser l'utilisation
- **Ingress**: TLS avec cert-manager et middleware de restriction d'accès
- **Port**: 8000 (container) → 80 (service)

## Déploiement

L'application Argocd `longhorn-ui` déploie ces ressources dans le namespace `longhorn-system`.

## Accès

L'interface est accessible via `https://longhorn.ga-nan.ovh` avec le middleware `traefik-local-only`.
