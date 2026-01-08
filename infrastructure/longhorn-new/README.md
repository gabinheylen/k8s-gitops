# Longhorn New (Installation Parallèle)

Ce dossier contient une nouvelle installation Longhorn pour migration propre.

## 🚨 PROBLÈME ACTUEL

Les managers Longhorn essaient d'accéder aux CRDs globales de l'ancienne installation au lieu d'en créer de nouvelles.

### Solutions possibles :

1. **Supprimer l'ancienne installation** complètement d'abord
2. **Utiliser un préfixe différent** pour les CRDs (non supporté par Longhorn)
3. **Attendre la fin d'initialisation** automatique

## 🎯 Objectif

Installer un Longhorn parallèle sans perturber l'existant, puis migrer les services progressivement.

## Configuration

- **Namespace**: `longhorn-system-new`
- **StorageClass**: `longhorn-new`
- **UI**: `https://longhorn-new.ga-nan.ovh`
- **Ressources**: Suffixées avec `-new`
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
- `crds.yaml` - CRDs (expérimental)

## Plan de migration

1. **Supprimer l'ancienne installation** Longhorn
2. **Attendre la fin** de la suppression du namespace
3. **Redéployer** cette nouvelle installation
4. **Mettre à jour** les services pour utiliser `longhorn-new`
5. **Tester** la création de volumes

## Déploiement

L'application Argocd `longhorn-new` déploie ces ressources.
