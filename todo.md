# TODO - Améliorations du projet k8s-gitops

## 🚨 Sécurité

### Hautement Prioritaire
- [ ] **Ajouter des secrets pour les données sensibles**
  - Les mots de passe et clés API sont actuellement en clair dans les manifests
  - Utiliser Kubernetes Secrets ou External Secrets Operator
  - Exemple: DOMAIN, WEBSOCKET_ENABLED dans vaultwarden

- [ ] **Configurer RBAC**
  - Pas de restrictions d'accès définies
  - Créer des rôles et permissions spécifiques par namespace
  - Limiter les privilèges des pods

- [ ] **Sécuriser les communications**
  - Pas de TLS/HTTPS configuré
  - Ajouter des certificats SSL (Let's Encrypt ou cert-manager)
  - Forcer HTTPS sur les ingress

### Prioritaire
- [ ] **Network Policies**
  - Isoler les namespaces entre eux
  - Restreindre le trafic réseau par défaut
  - Autoriser uniquement les communications nécessaires

- [ ] **Pod Security Policies/Standards**
  - Configurer les contextes de sécurité (securityContext)
  - Limiter les capabilities des conteneurs
  - Utiliser des utilisateurs non-root

## 📝 Configuration et Organisation

### Structure
- [ ] **Standardiser les namespaces**
  - `longhorn-ui` namespace créé mais l'application utilise `longhorn-system`
  - Incohérence dans les noms de namespaces

- [ ] **Ajouter des labels et annotations standards**
  - Manque de labels cohérents pour la gestion
  - Ajouter des labels pour le suivi, la gestion des coûts, etc.

- [ ] **Créer des valeurs configurables**
  - Utiliser ConfigMaps et Kustomize
  - Paramétrer les valeurs par environnement
  - Faciliter les déploiements multi-environnements

### Documentation
- [ ] **Ajouter un README.md**
  - Documenter l'architecture
  - Instructions d'installation
  - Prérequis et dépendances

- [ ] **Documenter les applications**
  - Qu'est-ce que Vaultwarden? Longhorn?
  - Comment accéder aux services?
  - Configuration par défaut

## 🔧 Gestion et Monitoring

### Ressources
- [ ] **Optimiser les ressources**
  - Limits/requests manquants sur plusieurs pods
  - Ajouter des quotas de ressources par namespace
  - Surveiller l'utilisation des ressources

- [ ] **Health checks**
  - Ajouter liveness et readiness probes
  - Surveiller l'état de santé des applications
  - Configurer des timeouts appropriés

### Monitoring et Logging
- [ ] **Ajouter des solutions de monitoring**
  - Prometheus + Grafana
  - AlertManager pour les notifications
  - Dashboards de surveillance

- [ ] **Centralisation des logs**
  - Fluentd/Fluent Bit
  - Elasticsearch + Kibana
  - Log retention policies

## 🚀 Déploiement et CI/CD

### GitOps
- [ ] **Valider les manifests**
  - Ajouter des tests de validation dans le pipeline
  - Utiliser kubeval ou kubeconform
  - Pré-déploiement automatisé

- [ ] **Gestion des versions**
  - Pinner les versions des images Docker
  - Utiliser des tags spécifiques au lieu de `latest`
  - Politique de mise à jour des images

- [ ] **Backup et Disaster Recovery**
  - Stratégie de backup des données
  - Backup de l'état du cluster
  - Plan de reprise d'activité

## 🌐 Réseau et Stockage

### Réseau
- [ ] **DNS interne**
  - Configurer CoreDNS ou ExternalDNS
  - Résolution de noms cohérente
  - Services discovery

- [ ] **Gestion des ingress**
  - Standardiser les annotations Traefik
  - Ajouter des middlewares (réécriture, authentification)
  - Gérer les certificats SSL

### Stockage
- [ ] **Backup des données**
  - Stratégie de backup pour Vaultwarden
  - Snapshots Longhorn réguliers
  - Stockage externe des backups

- [ ] **Optimisation du stockage**
  - Monitoring de l'utilisation des PVC
  - Nettoyage automatique des données obsolètes
  - Compression si nécessaire

## 🧪 Tests et Qualité

### Tests
- [ ] **Tests d'intégration**
  - Tests de connectivité entre services
  - Tests de montée en charge
  - Tests de failover

- [ ] **Validation continue**
  - Linting des fichiers YAML
  - Tests de syntaxe Kubernetes
  - Vérification des best practices

## 📊 Performance et Scalabilité

### Performance
- [ ] **Optimisation des performances**
  - Tuning des ressources
  - Optimisation des bases de données
  - Cache management

- [ ] **Scalabilité**
  - Horizontal Pod Autoscaler (HPA)
  - Cluster Autoscaler si nécessaire
  - Gestion de la charge

## 🔍 Divers

### Maintenance
- [ ] **Politique de rotation**
  - Rotation des secrets
  - Mise à jour des dépendances
  - Nettoyage des ressources non utilisées

- [ ] **Gestion des erreurs**
  - Stratégie de retry
  - Gestion des timeouts
  - Monitoring des erreurs

### Coûts
- [ ] **Optimisation des coûts**
  - Monitoring des ressources utilisées
  - Rightsizing des instances
  - Éteindre les services non critiques

---

## 🎯 Quick Wins (à faire en priorité)

1. **Corriger l'incohérence du namespace Longhorn**
2. **Ajouter des health checks aux déploiements**
3. **Sécuriser avec HTTPS/TLS**
4. **Créer un README.md**
5. **Ajouter des probes de monitoring**
6. **Utiliser des images versionnées (pas latest)**

## 📈 Échéancier suggéré

- **Semaine 1**: Sécurité de base + corrections critiques
- **Semaine 2-3**: Monitoring et logging
- **Semaine 4**: Documentation et tests
- **Mois 2**: Optimisation et scalabilité
