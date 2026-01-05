# K8s GitOps Infrastructure

Ce dépôt contient l'infrastructure Kubernetes gérée par GitOps avec ArgoCD.

## 🏗️ Architecture

### Infrastructure de base
- **ArgoCD**: Outil de GitOps pour le déploiement continu
- **Traefik**: Ingress Controller avec support HTTPS/TLS
- **MetalLB**: Load Balancer pour l'exposition des services
- **cert-manager**: Gestion automatique des certificats SSL

### Applications déployées
- **Vaultwarden**: Gestionnaire de mots de passe (Bitwarden compatible)
- **Longhorn UI**: Interface de gestion du stockage distribué

## 🚀 Déploiement

### Prérequis
- Cluster Kubernetes (v1.20+)
- kubectl configuré
- Accès au dépôt Git

### Installation
1. Installer ArgoCD manuellement :
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

2. Déployer l'application "app-of-apps" :
```bash
kubectl apply -f infrastructure/argocd/app-of-apps.yaml
```

3. Accéder à ArgoCD :
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
URL: `https://argocd.local`

## 🔧 Configuration

### Domaines configurés
- **ArgoCD**: `https://argocd.local`
- **Vaultwarden**: `https://vaultwarden.local`
- **Longhorn UI**: `https://longhorn.localhost`

### Certificats SSL
- Utilisation de Let's Encrypt via cert-manager
- Renouvellement automatique des certificats
- Support staging et production

### Stockage
- **Vaultwarden**: 5Gi PVC avec Longhorn
- **Longhorn**: Stockage distribué pour le cluster

## 📊 Monitoring

### Health Checks
- Liveness et readiness probes configurés
- Surveillance automatique de l'état des applications
- Redémarrage automatique en cas de défaillance

### Logs
- Logs des applications disponibles via kubectl
- Traefik logs pour le debugging réseau

## 🔒 Sécurité

### TLS/HTTPS
- Tous les services accessibles via HTTPS
- Redirection automatique HTTP→HTTPS
- Certificats SSL valides

### Network
- Isolation des namespaces
- Network policies recommandées
- Accès restreint par défaut

## 🛠️ Maintenance

### Mises à jour
- Images Docker versionnées (pas de `latest`)
- Mises à jour automatisées via ArgoCD
- Tests de validation avant déploiement

### Backups
- Données Vaultwarden sur PVC Longhorn
- Snapshots réguliers recommandés
- Export des configurations ArgoCD

## 📝 Notes importantes

### Configuration requise
- **Email**: Remplacer `your-email@example.com` dans les fichiers de configuration cert-manager
- **Domaines**: Assurer la résolution DNS des domaines locaux
- **Ressources**: Adapter les limits/requests selon votre infrastructure

### Développement
- Utiliser l'environnement staging pour les tests
- Valider les manifests avant déploiement
- Tester les mises à jour dans un namespace séparé

## 🆘 Support

### Debugging
```bash
# Vérifier le statut des applications ArgoCD
kubectl get applications -n argocd

# Logs des pods
kubectl logs -f deployment/vaultwarden -n vaultwarden

# Événements du namespace
kubectl get events -n vaultwarden --sort-by='.lastTimestamp'
```

### Ressources utiles
- [Documentation ArgoCD](https://argo-cd.readthedocs.io/)
- [Documentation Traefik](https://doc.traefik.io/traefik/)
- [Documentation cert-manager](https://cert-manager.io/docs/)

---

**Auteur**: Gabin Heylen  
**Version**: 1.0  
**Dernière mise à jour**: 2026-01-05
