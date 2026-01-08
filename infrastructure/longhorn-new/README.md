# Longhorn New (Installation Parallèle)

Ce dossier contient une nouvelle installation Longhorn pour migration propre.

## 🎯 Objectif

Installer un Longhorn parallèle sans perturber l'existant, puis migrer les services progressivement.

## Configuration

- **Namespace**: `longhorn-system-new`
- **StorageClass**: `longhorn-new`
- **UI**: `https://longhorn-new.ga-nan.ovh`
- **Ressources**: Suffixées avec `-new`

## Fichiers

- `namespace.yaml` - Namespace longhorn-system-new
- `priorityclass.yaml` - PriorityClass longhorn-critical-new
- `serviceaccount.yaml` - ServiceAccount principal
- `clusterrole.yaml` - Permissions RBAC longhorn-role-new
- `clusterrolebinding.yaml` - Liaison RBAC longhorn-bind-new
- `daemonset.yaml` - DaemonSet Longhorn Manager
- `service.yaml` - Service backend
- `storageclass.yaml` - StorageClass longhorn-new
- `ui.yaml` - Deployment + Service UI
- `ui-serviceaccount.yaml` - ServiceAccount UI
- `ingress.yaml` - Ingress avec domaine différent

## Plan de migration

1. **Déployer** cette nouvelle installation
2. **Tester** le fonctionnement
3. **Mettre à jour** les services pour utiliser `longhorn-new`
4. **Supprimer** l'ancienne installation
5. **Renommer** la nouvelle en `longhorn-system`

## Déploiement

L'application Argocd `longhorn-new` déploie ces ressources.
