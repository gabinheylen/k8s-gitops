📦 Installation propre d’ArgoCD sur Talos + GitOps
1️⃣ Créer le namespace ArgoCD

On commence par isoler ArgoCD dans son namespace dédié.

kubectl create namespace argocd

2️⃣ Installer ArgoCD via manifest officiel

On applique le manifest officiel, mais de façon déclarative via GitOps plus tard :

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


Vérifier les pods :

kubectl get pods -n argocd


Tu devrais voir :

argocd-server

argocd-repo-server

argocd-application-controller

argocd-dex-server

3️⃣ Exposer le serveur ArgoCD

Pour le moment, en dev, on peut exposer en NodePort :

kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
kubectl get svc -n argocd argocd-server


Note le NodePort pour te connecter via navigateur à https://<IP_CP>:<NODEPORT>

Par la suite, on passera à un Ingress propre (cert-manager + TLS).

4️⃣ Récupérer le mot de passe initial
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d


Utilisateur : admin
Mot de passe : celui que tu récupères

5️⃣ Préparer le repo GitHub pour GitOps

Structure recommandée :

k8s-gitops/
├── argocd/
│   ├── apps.yaml             # Applications ArgoCD (Longhorn, HA, etc.)
│   └── projects.yaml         # Optional ArgoCD projects
├── longhorn/
│   └── longhorn.yaml
├── home-assistant/
│   └── ha.yaml
└── README.md


Chaque dossier contient les manifests YAML nécessaires pour chaque app

ArgoCD va synchroniser ces manifests sur ton cluster

6️⃣ Créer la première Application ArgoCD pour ton repo GitHub

Exemple minimal (apps.yaml) :

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: gitops-root
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/TON_UTILISATEUR/k8s-gitops.git'
    targetRevision: main
    path: '.'
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true


Cette application « parent » permet à ArgoCD de synchroniser automatiquement tout le repo sur le cluster.

7️⃣ Vérifier
kubectl get applications -n argocd


Tu devrais voir ton gitops-root et son état Synced ou OutOfSync si tu n’as pas encore mis de manifests dans le repo.

✅ À ce stade :

ArgoCD est installé et fonctionnel

Tu peux accéder à l’UI

Tu as la structure GitOps pour toutes tes apps

Tu n’as rien appliqué manuellement hormis ArgoCD lui-même


kubectl create clusterrolebinding argocd-admin \
  --clusterrole=cluster-admin \
  --serviceaccount=argocd:argocd-application-controller
