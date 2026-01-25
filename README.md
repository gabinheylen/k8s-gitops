# 🚀 Vaultwarden Kubernetes Production Setup

> ⚠️ **AVERTISSEMENT DE SÉCURITÉ**  
> Ce projet contient des **placeholders** pour les informations sensibles.  
> **NE JAMAIS** commiter de vrais mots de passe, tokens ou informations personnelles dans Git.  
> Utilisez toujours la section "Configuration initiale" pour générer vos propres credentials.

## 📋 Vue d'ensemble

Ce projet déploie une instance **Vaultwarden** (serveur Bitwarden auto-hébergé) en production sur Kubernetes avec :
- ✅ **TLS Let's Encrypt** automatique
- ✅ **Ingress Controller Traefik** avec LoadBalancer MetalLB
- ✅ **Authentification BasicAuth** sur la route `/admin`
- ✅ **GitOps avec ArgoCD** pour la gestion des configurations
- ✅ **WebSocket support** pour les applications mobiles/desktop

---

## 🏗️ Architecture

```
Internet
    ↓
MetalLB LoadBalancer (IP: VOTRE_IP_PUBLIQUE)
    ↓
Traefik Ingress Controller (namespace: traefik-public)
    ↓
┌─────────────────────────────────────────────────────────┐
│                   Routes Traefik                        │
│                                                         │
│ 1. /admin → BasicAuth → vaultwarden:80 (priorité 100)   │
│ 2. /      → vaultwarden:80 (priorité par défaut)        │
│                                                         │
│ TLS: Let's Encrypt (votre-domaine.com)                 │
└─────────────────────────────────────────────────────────┘
    ↓
Vaultwarden Pods (namespace: vaultwarden)
    ↓
Données persistantes (PVC)
```

---

## 🔧 Composants techniques

### 1. **Traefik** - Ingress Controller
- **Namespace** : `traefik-public`
- **Type** : LoadBalancer via MetalLB
- **Fonction** : Routing HTTP/HTTPS, terminaison TLS, middlewares
- **Configuration** : Helm chart avec certificate resolver Let's Encrypt

### 2. **Vaultwarden** - Application
- **Namespace** : `vaultwarden`
- **Service** : ClusterIP sur port 80
- **Fonctionnalités** :
  - Interface web Bitwarden
  - WebSocket `/notifications/hub` pour les clients
  - Page d'administration `/admin`

### 3. **Cert-Manager** - Gestion des certificats
- **Namespace** : `cert-manager`
- **ClusterIssuer** : Let's Encrypt Production (HTTP-01 challenge)
- **Fonction** : Génération et renouvellement automatique des certificats TLS

### 4. **ArgoCD** - GitOps
- **Namespace** : `argocd`
- **Fonction** : Déploiement continu depuis Git
- **Applications** : traefik-public, vaultwarden, cert-manager

---

## 🔐 Sécurité

### Authentification BasicAdmin
- **Route protégée** : `/admin`
- **Identifiants** : À configurer lors de l'installation
  - Username : `admin`
  - Password : `VOTRE_MOT_DE_PASSE_SECURISE`
- **Middleware** : Traefik BasicAuth avec secret Kubernetes

### TLS Let's Encrypt
- **Domaine** : `votre-domaine.com`
- **Validité** : 3 mois avec renouvellement automatique
- **Challenge** : HTTP-01 via cert-manager

---

## 📁 Structure du projet

```
k8s-gitops/
├── argocd/                    # Applications ArgoCD
│   ├── cert-manager.yaml     # Déploiement cert-manager
│   ├── traefik-public.yaml  # Déploiement Traefik Helm
│   └── vaultwarden.yaml     # Déploiement Vaultwarden
├── cert-manager/              # Configuration cert-manager
│   ├── cert-manager.yaml     # Helm chart cert-manager
│   └── cluster-issuer.yaml  # Let's Encrypt ClusterIssuer
├── traefik-public/           # Configuration Traefik
│   ├── middleware-admin-basicauth.yaml
│   └── vaultwarden-service.yaml
├── vaultwarden/              # Configuration Vaultwarden
│   ├── admin-auth-secret.yaml          # Secret BasicAuth
│   ├── certificate.yaml                # Certificat TLS
│   ├── ingressroute-admin.yaml         # Route /admin protégée
│   ├── ingress-public.yaml             # Route principale
│   ├── middleware-admin-basicauth.yaml # Middleware BasicAuth
│   ├── secret.yaml                     # Token admin Vaultwarden
│   └── vaultwarden.yaml               # Deployment + Service
└── README.md                   # Cette documentation
```

---

## 🚀 Installation complète

### Prérequis
```bash
# Cluster Kubernetes avec MetalLB configuré
# kubectl configuré et fonctionnel
# Domaine votre-domaine.com pointant vers l'IP du cluster
```

### 1. **Cloner le repository**
```bash
git clone https://github.com/VOTRE_USERNAME/k8s-gitops.git
cd k8s-gitops
```

### 1.1. **Configuration initiale (obligatoire)**
```bash
# 1. Configurer votre domaine
sed -i 's/votre-domaine.com/VOTRE_DOMAINE.com/g' **/*.yaml

# 2. Configurer votre email Let's Encrypt
sed -i 's/VOTRE_EMAIL@exemple.com/VOTRE_EMAIL@exemple.com/g' argocd/traefik-public.yaml

# 3. Générer un mot de passe BasicAuth sécurisé
htpasswd -nb admin VOTRE_MOT_DE_PASSE > /tmp/auth.txt
HASH=$(cat /tmp/auth.txt | cut -d: -f2)
echo "admin:$HASH" | base64 -w 0 > /tmp/users.txt

# 4. Mettre à jour le secret BasicAuth
sed -i "s|users: .*|users: $(cat /tmp/users.txt)|g" vaultwarden/admin-auth-secret.yaml

# 5. Générer un token admin Vaultwarden
ADMIN_TOKEN=$(openssl rand -base64 32 | tr -d "=+/" | cut -c1-32)
echo "VOTRE_TOKEN_ADMIN: $ADMIN_TOKEN"

# 6. Mettre à jour le secret admin token
echo "$ADMIN_TOKEN" | base64 -w 0 > /tmp/token.txt
sed -i "s|admin-token: .*|admin-token: $(cat /tmp/token.txt)|g" vaultwarden/secret.yaml

# 7. Commiter les changements
git add .
git commit -m "Configure domain, email and credentials"
git push origin main
```

### 3. **Déployer ArgoCD**
```bash
kubectl apply -k argocd/
# ArgoCD va automatiquement déployer les autres applications
```

### 4. **Vérifier le déploiement**
```bash
# Vérifier les applications ArgoCD
kubectl get applications -n argocd

# Vérifier les pods
kubectl get pods -n traefik-public
kubectl get pods -n vaultwarden
kubectl get pods -n cert-manager

# Vérifier les services
kubectl get svc -n traefik-public
kubectl get svc -n vaultwarden
```

### 5. **Vérifier le certificat TLS**
```bash
# Vérifier le certificat Let's Encrypt
kubectl get certificate -n vaultwarden
kubectl describe certificate vaultwarden-tls -n vaultwarden
```

---

## 🔧 Configuration détaillée

### Traefik Configuration
```yaml
# argocd/traefik-public.yaml
additionalArguments:
  - --certificatesresolvers.letsencrypt.acme.email=VOTRE_EMAIL@exemple.com
  - --certificatesresolvers.letsencrypt.acme.storage=/data/acme.json
  - --certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web
```

### BasicAuth Middleware
```yaml
# vaultwarden/middleware-admin-basicauth.yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: vaultwarden-admin-basicauth
  namespace: vaultwarden
spec:
  basicAuth:
    secret: vaultwarden-admin-auth
```

### IngressRoute Admin (protégée)
```yaml
# vaultwarden/ingressroute-admin.yaml
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: vaultwarden-admin
  namespace: vaultwarden
spec:
  entryPoints: [websecure]
  routes:
  - match: Host(`votre-domaine.com`) && PathPrefix(`/admin`)
    priority: 100
    middlewares: [vaultwarden-admin-basicauth]
    services:
    - name: vaultwarden
      port: 80
  tls:
    secretName: vaultwarden-tls
```

---

## 🌐 Accès à l'application

### URL principales
- **Application Vaultwarden** : `https://votre-domaine.com`
- **Administration** : `https://votre-domaine.com/admin`

### Identifiants
#### Accès administration
- **Username** : `admin`
- **Password** : `VOTRE_MOT_DE_PASSE_SECURISE` (à configurer)

#### Token admin Vaultwarden (interne)
- **Token** : `VOTRE_TOKEN_ADMIN_GENERE` (à générer)
- **Utilisation** : Page d'administration Vaultwarden

---

## 🔄 Gestion des mots de passe et configuration

### Changer le mot de passe BasicAuth
```bash
# 1. Générer un nouveau hash htpasswd
htpasswd -nb admin NOUVEAU_MOT_DE_PASSE

# 2. Mettre à jour le secret Kubernetes
kubectl delete secret vaultwarden-admin-auth -n vaultwarden --ignore-not-found
echo "admin:HASH_GENERE" | base64 -w 0 > /tmp/users.txt
kubectl create secret generic vaultwarden-admin-auth -n vaultwarden \
  --from-literal=users="$(cat /tmp/users.txt)"

# 3. Mettre à jour le fichier Git pour la persistance
sed -i "s|users: .*|users: $(cat /tmp/users.txt)|g" vaultwarden/admin-auth-secret.yaml
git add vaultwarden/admin-auth-secret.yaml
git commit -m "Update BasicAuth password"
git push origin main
```

### Changer le token admin Vaultwarden
```bash
# 1. Générer un nouveau token sécurisé
ADMIN_TOKEN=$(openssl rand -base64 32 | tr -d "=+/" | cut -c1-32)
echo "Nouveau token admin: $ADMIN_TOKEN"

# 2. Mettre à jour le secret Kubernetes
echo "$ADMIN_TOKEN" | base64 -w 0 > /tmp/token.txt
kubectl patch secret vaultwarden-admin-token -n vaultwarden \
  -p='{"data":{"admin-token":"'$(cat /tmp/token.txt)'"}}'

# 3. Mettre à jour le fichier Git
sed -i "s|admin-token: .*|admin-token: $(cat /tmp/token.txt)|g" vaultwarden/secret.yaml
git add vaultwarden/secret.yaml
git commit -m "Update Vaultwarden admin token"
git push origin main

# 4. Redémarrer Vaultwarden pour appliquer le changement
kubectl rollout restart deployment/vaultwarden -n vaultwarden
```

### Changer le domaine
```bash
# 1. Mettre à jour tous les fichiers YAML
find . -name "*.yaml" -type f -exec sed -i "s/ancien-domaine.com/nouveau-domaine.com/g" {} \;

# 2. Mettre à jour le certificat
kubectl delete certificate vaultwarden-tls -n vaultwarden --ignore-not-found

# 3. Commiter et pousser les changements
git add .
git commit -m "Change domain from ancien-domaine.com to nouveau-domaine.com"
git push origin main

# 4. Vérifier le nouveau certificat
sleep 60 && kubectl get certificate -n vaultwarden
```

### Changer l'email Let's Encrypt
```bash
# 1. Mettre à jour la configuration Traefik
sed -i 's/ancien.email@exemple.com/nouveau.email@exemple.com/g' argocd/traefik-public.yaml

# 2. Commiter et pousser
git add argocd/traefik-public.yaml
git commit -m "Update Let's Encrypt email"
git push origin main

# 3. Redémarrer Traefik si nécessaire
kubectl rollout restart deployment/traefik-public -n traefik-public
```

### Mettre à jour les images Docker
```bash
# 1. Mettre à jour l'image Vaultwarden
kubectl set image deployment/vaultwarden vaultwarden=vaultwarden/server:latest -n vaultwarden

# 2. Vérifier le rollout
kubectl rollout status deployment/vaultwarden -n vaultwarden

# 3. Mettre à jour le fichier Git pour la persistance
sed -i 's|image: .*|image: vaultwarden/server:latest|g' vaultwarden/vaultwarden.yaml
git add vaultwarden/vaultwarden.yaml
git commit -m "Update Vaultwarden image to latest"
git push origin main
```

### Sauvegarder et restaurer les données
```bash
# Sauvegarder les données Vaultwarden
kubectl exec -n vaultwarden deployment/vaultwarden -- tar czf - /data > backup-$(date +%Y%m%d).tar.gz

# Restaurer les données Vaultwarden
kubectl exec -i -n vaultwarden deployment/vaultwarden -- tar xzf - -C / < backup-YYYYMMDD.tar.gz

# Sauvegarder les certificats
kubectl get secret vaultwarden-tls -n vaultwarden -o yaml > cert-backup-$(date +%Y%m%d).yaml
```

### Vérifier l'état du système
```bash
# État global des applications
kubectl get applications -n argocd

# État des pods par namespace
kubectl get pods -n traefik-public
kubectl get pods -n vaultwarden
kubectl get pods -n cert-manager

# État des services
kubectl get svc -n traefik-public
kubectl get svc -n vaultwarden

# État des certificats
kubectl get certificate -n vaultwarden
kubectl describe certificate vaultwarden-tls -n vaultwarden

# Logs des composants principaux
kubectl logs -n traefik-public deployment/traefik-public --tail=50
kubectl logs -n vaultwarden deployment/vaultwarden --tail=50
kubectl logs -n cert-manager deployment/cert-manager --tail=20
```

### Debug et dépannage
```bash
# Test de connectivité HTTP
curl -I https://votre-domaine.com
curl -I https://votre-domaine.com/admin

# Test avec authentification BasicAuth
curl -I -u admin:VOTRE_MOT_DE_PASSE https://votre-domaine.com/admin

# Test des certificats TLS
openssl s_client -connect votre-domaine.com:443 -servername votre-domaine.com

# Test WebSocket (doit retourner 400 car c'est un protocole WebSocket)
curl -I https://votre-domaine.com/notifications/hub

# Vérifier la configuration Traefik
kubectl get ingressroutes -n vaultwarden
kubectl describe ingressroute vaultwarden-admin -n vaultwarden

# Vérifier les middlewares
kubectl get middlewares -n vaultwarden
kubectl describe middleware vaultwarden-admin-basicauth -n vaultwarden

# Accès aux dashboards (localement)
kubectl port-forward -n traefik-public svc/traefik-public 9000:9000 &
echo "Dashboard Traefik: http://localhost:9000/dashboard/"

kubectl port-forward -n argocd svc/argocd-server 8080:443 &
echo "Dashboard ArgoCD: https://localhost:8080 (admin:password à configurer)"
```

### Maintenance régulière
```bash
# Script de maintenance mensuelle
#!/bin/bash
echo "=== Maintenance Vaultwarden ==="

# 1. Vérifier les certificats
echo "Vérification des certificats..."
kubectl get certificate -n vaultwarden
kubectl get orders -n vaultwarden

# 2. Vérifier les pods
echo "Vérification des pods..."
kubectl get pods -n vaultwarden
kubectl get pods -n traefik-public

# 3. Nettoyer les anciens secrets
echo "Nettoyage des anciens secrets..."
kubectl get secrets -n vaultwarden | grep "vaultwarden-tls-" | tail -n +2 | awk '{print $1}' | xargs -r kubectl delete secret -n vaultwarden

# 4. Backup des données
echo "Backup des données..."
kubectl exec -n vaultwarden deployment/vaultwarden -- tar czf - /data > backup-$(date +%Y%m%d).tar.gz

echo "Maintenance terminée!"
```

### Sécurité - Audit des permissions
```bash
# Vérifier les permissions des pods
kubectl auth can-i create pods --namespace=vaultwarden
kubectl auth can-i get secrets --namespace=vaultwarden

# Scanner les vulnérabilités des images
kubectl get pods -n vaultwarden -o jsonpath='{.items[*].spec.containers[*].image}' | tr ' ' '\n' | sort | uniq

# Vérifier les NetworkPolicies (si configurées)
kubectl get networkpolicies -n vaultwarden
```

---

## 📊 Monitoring et dépannage

### Vérifier l'état de l'application
```bash
# Statut des applications ArgoCD
kubectl get applications -n argocd

# Logs Vaultwarden
kubectl logs -n vaultwarden deployment/vaultwarden

# Logs Traefik
kubectl logs -n traefik-public deployment/traefik-public

# Certificats
kubectl get certificate -n vaultwarden
kubectl describe certificate vaultwarden-tls -n vaultwarden
```

### Problèmes courants

#### Certificat Let's Encrypt ne se renouvelle pas
```bash
# Vérifier les challenges ACME
kubectl get order -n vaultwarden
kubectl describe order <order-name> -n vaultwarden

# Forcer la recréation du certificat
kubectl delete certificate vaultwarden-tls -n vaultwarden
kubectl apply -f vaultwarden/certificate.yaml
```

#### BasicAuth ne fonctionne pas
```bash
# Vérifier le middleware
kubectl get middleware -n vaultwarden
kubectl describe middleware vaultwarden-admin-basicauth -n vaultwarden

# Vérifier le secret
kubectl get secret vaultwarden-admin-auth -n vaultwarden -o yaml
```

#### WebSocket ne fonctionne pas
```bash
# Vérifier la configuration WebSocket
curl -I https://vaultwarden.ga-nan.ovh/notifications/hub
# Doit retourner 400 (WebSocket handshake requis)
```

---

## 🔄 GitOps Workflow

### Structure GitOps
1. **ArgoCD** surveille le repository Git
2. **Détection des changements** → synchronisation automatique
3. **Déploiement** des manifests Kubernetes
4. **Health checks** et monitoring

### Cycle de modification
```bash
# 1. Modifier les fichiers manifests
# 2. Commiter les changements
git add .
git commit -m "Description du changement"
git push origin main

# 3. ArgoCD applique automatiquement les changements
# 4. Vérifier le statut
kubectl get applications -n argocd
```

---

## 🛡️ Considérations de sécurité

### Production recommendations
- ✅ **TLS Let's Encrypt** automatique
- ✅ **BasicAuth** sur l'administration
- ✅ **NetworkPolicies** (à implémenter)
- ✅ **PodSecurityPolicies** (à implémenter)
- ✅ **Backup régulier** des données Vaultwarden
- ✅ **Monitoring** des certificats et services

### Sécurité du mot de passe
- Le mot de passe BasicAuth est stocké en base64 dans Git
- Pour plus de sécurité, utiliser **SealedSecrets** ou **External Secrets Operator**
- Le token admin Vaultwarden est un secret Kubernetes

---

## 📚 Ressources utiles

### Documentation officielle
- [Vaultwarden](https://github.com/dani-garcia/vaultwarden)
- [Traefik](https://doc.traefik.io/traefik/)
- [Cert-Manager](https://cert-manager.io/docs/)
- [ArgoCD](https://argoproj.github.io/cd/)

### Commandes utiles
```bash
# Accès au dashboard Traefik
kubectl port-forward -n traefik-public svc/traefik-public 9000:9000

# Accès au dashboard ArgoCD
kubectl port-forward -n argocd svc/argocd-server 8080:443

# Debug des certificats
openssl s_client -connect votre-domaine.com:443 -servername votre-domaine.com

# Test WebSocket
wscat -c wss://votre-domaine.com/notifications/hub
```

---

## 📝 Maintenance

### Tâches régulières
- **Mensuel** : Vérifier les renouvellements de certificats
- **Trimestriel** : Mise à jour des images Docker
- **Semestriel** : Rotation des mots de passe
- **Annuel** : Audit de sécurité

### Backup des données
```bash
# Backup des données Vaultwarden
kubectl exec -n vaultwarden deployment/vaultwarden -- tar czf - /data > vaultwarden-backup.tar.gz

# Restore des données
kubectl exec -i -n vaultwarden deployment/vaultwarden -- tar xzf - -C / < vaultwarden-backup.tar.gz
```

---

## 🎉 Conclusion

Ce setup fournit une instance **Vaultwarden production-ready** avec :
- 🔐 **Sécurité** TLS et authentification
- 🚀 **Performance** Traefik et HTTP/2
- 🔄 **Fiabilité** GitOps et monitoring
- 🛠️ **Maintenabilité** Infrastructure as Code

L'architecture est conçue pour être **scalable**, **sécurisée** et **facile à maintenir** grâce à GitOps.
