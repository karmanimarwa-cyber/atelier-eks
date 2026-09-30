# Atelier GitOps — Construire un chart Helm déployé par ArgoCD sur EKS

Repo **template** de l'atelier. Chaque participant **forke** ce dépôt, provisionne
un cluster EKS **dans la sandbox AWS Pluralsight**, installe **ArgoCD**, puis
**écrit lui-même**, pas à pas, un **chart Helm** pour le microservice **podinfo**.
Chaque commit est synchronisé automatiquement par ArgoCD : on voit l'application
se construire, composant par composant.

> 🎓 **Vous suivez l'atelier ?** Ouvrez [`guide.html`](guide.html) dans votre
> navigateur : c'est le parcours pas-à-pas. Ce README décrit le **contenu du dépôt**
> (référence technique).

---

## Objectif de l'atelier

Public : développeurs à l'aise avec Docker, débutants sur Kubernetes/AWS. Pas de
durée stricte : le guide est **autonome** et peut se terminer à la maison. À la fin,
vous saurez :

- Provisionner un cluster **EKS** avec `eksctl`.
- Installer **ArgoCD** et déclarer une **Application** en mode **Helm**.
- **Écrire un chart Helm** (Deployment, Service, valeurs, sondes, HPA).
- Vivre le **GitOps** : commit → synchronisation automatique → app mise à jour.

---

## Deux environnements

| Environnement | Rôle | Ce qu'on y fait |
|---------------|------|-----------------|
| **AWS CloudShell** | Interactions AWS / cluster | Créer l'EKS, installer ArgoCD, `kubectl`, générer le kubeconfig |
| **Poste local (IDE)** | Écriture du code | Éditer le chart Helm, `helm lint`, `git` |

> 🔒 Le poste local ne lance **jamais** de commande AWS → aucun risque de créer des
> ressources sur un compte d'entreprise. L'accès aux UIs se fait via un **kubeconfig
> par jeton** (généré dans CloudShell, utilisé en local sans AWS).

---

## Structure du dépôt

```
.
├── README.md                  # ce fichier (référence technique)
├── guide.html                 # guide pas-à-pas (à ouvrir au navigateur)
├── .gitignore                 # ignore .local/ (kubeconfig local), secrets, artefacts Helm…
├── infra/
│   └── cluster.yaml           # config eksctl (EKS minimal, us-east-1, v1.34)
├── argocd/
│   ├── application.yaml       # Application ArgoCD (source Helm → charts/podinfo)
│   ├── dashboard.yaml         # Applications ArgoCD (Kubernetes Dashboard + metrics-server) — workloads, events, logs, CPU/RAM
│   ├── monitoring.yaml        # Application ArgoCD (Prometheus, kube-prometheus-stack allégé)
│   └── dynatrace.yaml         # Applications ArgoCD (Dynatrace Operator + DynaKube) — bonus
├── dynatrace/
│   └── dynakube.yaml          # CR DynaKube (apiUrl à personnaliser) — bonus, complément de Prometheus
├── charts/
│   └── podinfo/               # chart Helm — SQUELETTE à compléter par le participant
│       ├── Chart.yaml         # métadonnées du chart
│       ├── values.yaml        # valeurs de départ (image, replicaCount)
│       └── templates/         # VIDE au départ — vous y écrivez deployment/service/hpa/servicemonitor
└── docs/
    └── prerequisites.md       # prérequis à réaliser AVANT l'atelier
```

---

## Prérequis

Détails dans [`docs/prerequisites.md`](docs/prerequisites.md).

| Outil   | Où       | Rôle                                   |
|---------|----------|----------------------------------------|
| AWS CLI | CloudShell (préinstallé) | Accès au compte sandbox   |
| kubectl | CloudShell (préinstallé) + local | Piloter Kubernetes |
| eksctl  | CloudShell (à installer) | Créer le cluster EKS      |
| helm    | **local** | Écrire / valider le chart (`helm lint`) |
| git     | local     | Forker / committer                     |

---

## Déroulé résumé

| Étape | Où | Clé |
|-------|----|-----|
| 1. Créer le cluster    | CloudShell | `eksctl create cluster -f cluster.yaml` |
| 2. Installer ArgoCD    | CloudShell | manifest officiel v3.5.3 |
| 3. Accès aux UIs       | CloudShell → local | kubeconfig par jeton + `port-forward` local |
| 4. Application ArgoCD  | CloudShell | `kubectl apply -f application.yaml` (source Helm) |
| 5. Dashboard K8s (GitOps) | local | `kubectl apply -f dashboard.yaml` (Kubernetes Dashboard + metrics-server → workloads, events, logs, CPU/RAM) |
| 6. Prometheus (GitOps) | local | `kubectl apply -f monitoring.yaml` (kube-prometheus-stack allégé) |
| 7. Construire le chart | local + GitHub | écrire `templates/*` (dont `servicemonitor.yaml`), commit → ArgoCD sync |
| 8. Dynatrace (bonus)   | CloudShell + local | Secret tokens (hors Git) + `kubectl apply -f dynatrace.yaml` |
| 9. Nettoyer            | CloudShell | `eksctl delete cluster --name atelier-argocd --region us-east-1` |

Le détail complet est dans [`guide.html`](guide.html).

---

## Points d'attention

- **Sandbox Pluralsight** : régions **`us-east-1`** (utilisée ici) ou `us-west-2` ;
  EC2 `t2/t3/t3a/t4g` en `micro/small/medium`. Sandbox détruite après ~4h (pas de
  coût), teardown **optionnel**.
- **Version EKS** : `1.34` (doit être en « standard support » ; `eksctl` liste les
  versions valides s'il refuse).
- **CloudShell — collage multi-lignes** : confirmer via « Safe Paste » (bouton Paste).
- **CloudShell — pas d'aperçu de port web** : d'où l'accès aux UIs via kubeconfig
  par jeton, utilisé **en local**.
- **`repoURL` à personnaliser** dans [`argocd/application.yaml`](argocd/application.yaml)
  (URL de votre fork).
- **Version podinfo** : image épinglée `ghcr.io/stefanprodan/podinfo:6.7.1`.

---

## Valider le chart localement

```bash
echo "=== Lint du chart ==="
helm lint charts/podinfo

echo ""
echo "=== Rendu du chart (manifests générés) ==="
helm template podinfo charts/podinfo
```

Au départ, `templates/` est vide : `helm template` ne rend rien. À mesure que vous
ajoutez les templates (étape 7 du guide, « Construire le chart pas à pas »), le rendu
se remplit.
