# Formation Kubernetes - Slides

Support de présentation interactif pour la formation Kubernetes.

Construit avec [Slidev](https://sli.dev) + [staticrypt](https://github.com/robinmoisson/staticrypt) pour la protection par passphrase, déployé sur GitHub Pages. Palette aux couleurs du logo Kubernetes (bleu `#326ce5` sur fond blanc).

---

## Prérequis

- Node.js 20+
- [pnpm](https://pnpm.io/installation) 9+ (`npm install -g pnpm` si pas déjà installé)

## Développement local

```bash
pnpm install
pnpm dev             # ouvre http://localhost:3030
```

Modifier `slides.md` → hot-reload automatique.

**Raccourcis utiles pendant la présentation** :
- `f` : plein écran
- `o` : vue d'ensemble (toutes les slides)
- `d` : mode sombre
- `g` : aller à une slide (tape le numéro)
- `flèches` / `espace` : naviguer

---

## Build en local (pour tester le build protégé)

```bash
export REPO_NAME="k8s-course"
export STATICRYPT_PASSPHRASE="ma-passphrase-test"
pnpm build:protected
pnpm serve:protected            # teste le résultat sur http://localhost:3000/k8s-course/
```

> ⚠️ Le build utilise un **base path** (`/k8s-course/`) pour GitHub Pages. Le script
> `serve:protected` recopie `dist-protected` sous ce sous-chemin - ouvrir l'URL **avec** le
> suffixe `/k8s-course/` (servir la racine directement renvoie des 404 sur les assets).

---

## Déploiement GitHub Pages

### Setup initial (une seule fois)

1. Créer le dépôt GitHub et pousser ce dossier à sa racine.
2. Dans **Settings → Pages** : source = **GitHub Actions**.
3. Dans **Settings → Secrets and variables → Actions → New repository secret** :
   - Nom : `STATICRYPT_PASSPHRASE`
   - Valeur : la passphrase à distribuer aux stagiaires.
4. Pousser sur `main` → le workflow `deploy.yml` construit, chiffre et publie.

### Mise à jour

- Modifier `slides.md` → commit → push → redéploiement auto.
- Changer la passphrase : éditer le secret GitHub, puis relancer le workflow (`Actions → Deploy → Re-run`).


