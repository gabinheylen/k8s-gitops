# Longhorn Infrastructure

Ce dossier contient la configuration principale de Longhorn pour GitOps avec Argocd.

## 🚨 ATTENTION - Migration en cours

Plusieurs services utilisent actuellement Longhorn :
- **vaultwarden** : 5Gi (actif)
- **postgres** : 20Gi (en migration)
- **homeassistant** : 5Gi (en terminaison)

### Plan de migration :
1. Créer un nouveau storageclass temporaire
2. Migrer les volumes critiques (vaultwarden)
3. Supprimer l'ancienne installation Longhorn
4. Déployer la nouvelle configuration
5. Recréer les volumes avec le nouveau Longhorn

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
