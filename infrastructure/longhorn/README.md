# Longhorn Infrastructure

Ce dossier contient la configuration principale de Longhorn pour GitOps avec Argocd.

## Fichiers

- `namespace.yaml` - Namespace longhorn-system
- `priorityclass.yaml` - PriorityClass pour les pods critiques
- `serviceaccount.yaml` - ServiceAccount principal
- `clusterrole.yaml` - Permissions RBAC complètes
- `clusterrolebinding.yaml` - Liaison RBAC
- `daemonset.yaml` - DaemonSet Longhorn Manager
- `service.yaml` - Service backend
- `driver-deployer.yaml` - Deployment CSI Driver
- `settings.yaml` - Configuration par défaut
- `storageclass.yaml` - StorageClass Longhorn
- `support-bundle.yaml` - Support Bundle ServiceAccount
- `crds.yaml` - CRDs (installées automatiquement)

## Déploiement

L'application Argocd `longhorn` déploie ces ressources dans le namespace `longhorn-system`.

## Séparation

L'interface utilisateur Longhorn UI est gérée séparément dans `apps/longhorn-ui/`.
