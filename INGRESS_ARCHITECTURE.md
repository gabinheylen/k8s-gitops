# 🌐 Architecture Ingress Simplifiée et Fiable

## 📋 Vue d'ensemble

Architecture réseau avec deux Traefik séparés pour des usages distincts :
- **Traefik Public** : Services exposés sur internet (Vaultwarden)
- **Traefik Private** : Services locaux uniquement (.lan)

## 🏗️ Configuration Réseau

### MetalLB - Deux pools d'IPs
```
Pool Public : 192.168.1.225 (IP fixe pour Traefik Public)
Pool LAN    : 192.168.1.230-192.168.1.235 (pour Traefik Private)
```

### Traefik Public (traefik-public namespace)
- **IP** : 192.168.1.225
- **Usage** : Services publics avec TLS Let's Encrypt
- **Services** :
  - Vaultwarden : `https://vaultwarden.ga-nan.ovh`
  - Admin Vaultwarden : `https://vaultwarden.ga-nan.ovh/admin` (BasicAuth)

### Traefik Private (traefik namespace)
- **IP** : 192.168.1.230-192.168.1.235
- **Usage** : Services locaux via DNS (192.168.1.221)
- **Services** :
  - ArgoCD : `http://argocd.lan`
  - Longhorn UI : `http://longhorn.lan` (BasicAuth)
  - Vaultwarden local : `http://vaultwarden.lan`

## 🔧 Configuration DNS

Votre serveur DNS (192.168.1.221) doit résoudre :
```
argocd.lan        -> 192.168.1.230
longhorn.lan      -> 192.168.1.230
vaultwarden.lan   -> 192.168.1.230
```

## 📁 Fichiers d'ingress créés/modifiés

### Services Publics (Traefik Public)
- `vaultwarden/ingress.yaml` - Vaultwarden principal
- `vaultwarden/ingress-admin.yaml` - Interface admin avec BasicAuth

### Services Locaux (Traefik Private)
- `argocd/ingress-local.yaml` - ArgoCD interface
- `longhorn/ingress-local.yaml` - Longhorn UI avec BasicAuth
- `vaultwarden/ingress-local.yaml` - Vaultwarden local

### Middlewares d'authentification
- `traefik-public/middleware-admin-basicauth.yaml` (namespace vaultwarden)
- `traefik-public/middleware-longhorn-basicauth.yaml` (namespace longhorn-system)

## 🚀 Déploiement

Les applications ArgoCD créées :
- `vaultwarden-ingress` - Déploie les ingress Vaultwarden
- `longhorn-ingress` - Déploie les ingress Longhorn

## 🔐 Sécurité

### BasicAuth
- **Vaultwarden Admin** : `admin:password` (secret vaultwarden-admin-auth)
- **Longhorn UI** : `admin:admin` (secret longhorn-admin-auth)

### TLS
- **Services publics** : Let's Encrypt automatique
- **Services locaux** : HTTP uniquement (réseau local)

## 🛠️ Dépannage

### Vérifier les IPs MetalLB
```bash
kubectl get svc -n traefik-public
kubectl get svc -n traefik
```

### Vérifier les ingress
```bash
kubectl get ingress -A
kubectl describe ingress <nom> -n <namespace>
```

### Tester la résolution DNS
```bash
nslookup argocd.lan 192.168.1.221
nslookup longhorn.lan 192.168.1.221
nslookup vaultwarden.lan 192.168.1.221
```

### Logs Traefik
```bash
kubectl logs -n traefik-public deployment/traefik-public
kubectl logs -n traefik deployment/traefik
```

## ✅ Avantages de cette architecture

1. **Séparation claire** : Public vs Privé
2. **Sécurité** : TLS pour public, authentification où nécessaire
3. **Fiabilité** : Configuration simple et standard
4. **Maintenance** : Chaque Traefik gère son domaine
5. **DNS local** : Résolution simple avec dnsmasq

## 🔄 Workflow GitOps

1. Modifier les fichiers d'ingress
2. Commiter dans Git
3. ArgoCD applique automatiquement
4. Vérifier le déploiement

Cette architecture est conçue pour être simple, fiable et facile à maintenir !
