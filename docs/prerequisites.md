# Prérequis de l'atelier — sandbox AWS Pluralsight

> ⏱️ **5 minutes.** À lire **avant** l'atelier. L'accès AWS est fourni par la
> plateforme — pas de compte perso ni de carte bancaire.

L'atelier se déroule dans la **sandbox AWS de Pluralsight**
([Hands-on Playground](https://app.pluralsight.com/hands-on)) et utilise **deux
environnements aux rôles distincts**.

---

## 1. Deux environnements

| Environnement | Rôle | Outils |
|---------------|------|--------|
| **AWS CloudShell** (dans le navigateur) | Tout ce qui touche **AWS / le cluster** : créer l'EKS, installer ArgoCD, `kubectl`, générer le kubeconfig | `aws`, `kubectl`, `git` **préinstallés** ; `eksctl` à installer (1 commande) |
| **Poste local (IDE)** | **Écrire le code** : le chart Helm, `helm lint`, `git` | `helm`, `git`, un IDE |

> 🔒 **Sécurité.** Toutes les commandes AWS se font **dans CloudShell** (authentifié
> sur la sandbox → impossible de toucher un compte d'entreprise). Le poste local ne
> lance **jamais** de commande AWS : il sert à éditer le chart et à `git push`.
> L'accès aux interfaces web (ArgoCD, podinfo) se fait via un **kubeconfig par jeton**
> généré dans CloudShell puis utilisé en local (voir le guide, étape 5) — sans AWS.

---

## 2. Contraintes de la sandbox (à connaître)

Déjà respectées par les fichiers de l'atelier — juste pour comprendre :

| Contrainte | Valeur |
|------------|--------|
| Régions autorisées | **us-east-1** ou us-west-2 uniquement |
| Types EC2 autorisés | t2 / t3 / t3a / t4g en micro, small, medium |
| Version EKS | « standard support » (ex. **1.34**) |
| Durée de la sandbox | ~4 h, puis destruction automatique |
| Facturation | aucune (la plateforme gère) |

> ⚠️ **Inactivité / collage** : CloudShell peut se fermer après quelques minutes
> d'inactivité, et demande de confirmer les collages multi-lignes (« Safe Paste »
> → bouton **Paste**). Ces points sont rappelés dans le guide.

---

## 3. Côté CloudShell

Rien à installer à l'avance : `aws`, `kubectl`, `git` sont préinstallés. Vous
installerez `eksctl` en début d'atelier (commande fournie dans le guide). Vérifiez
juste que vous savez **ouvrir CloudShell** depuis la console de la sandbox (icône
terminal, en haut).

---

## 4. Côté poste local

Installez (ou vérifiez) :

| Outil | Version min. | Rôle |
|-------|--------------|------|
| helm  | 3.x          | Écrire et valider le chart (`helm lint`, `helm template`) |
| git   | 2.x          | Committer / pousser vers votre fork |
| IDE   | —            | Éditer les fichiers du chart (VS Code, etc.) |

```bash
echo "=== Vérification des outils locaux ==="
helm version
git --version
```

> 💡 Pas besoin d'`aws` ni d'`eksctl` en local : ces commandes se font uniquement
> dans CloudShell.

---

## 5. Checklist « prêt pour l'atelier »

- [ ] J'ai accès à la sandbox AWS Pluralsight (Hands-on Playground)
- [ ] Je sais ouvrir **AWS CloudShell** depuis la console
- [ ] `helm version` et `git --version` fonctionnent sur mon poste
- [ ] J'ai un **IDE** pour éditer des fichiers YAML
- [ ] J'ai un compte **GitHub** (pour forker le dépôt de l'atelier)

Si toutes les cases sont cochées : **vous êtes prêt·e.** 🎉
