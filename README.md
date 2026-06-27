# Formation Kubernetes - Slides

Support de présentation interactif pour la formation Kubernetes (2 jours).

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
> `serve:protected` recopie `dist-protected` sous ce sous-chemin — ouvrir l'URL **avec** le
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

---

## Modèle de sécurité : à comprendre avant d'utiliser

`staticrypt` fournit une **protection de classe salle de cours**, pas de confidentialité forte :

✅ **Ça protège contre** :
- Accès aléatoire via l'URL publique
- Indexation par moteurs de recherche
- Partage accidentel du lien sans passphrase

❌ **Ça ne protège PAS contre** :
- Un stagiaire qui a la passphrase et sauvegarde la page pour la rediffuser
- Extraction de contenu via devtools une fois la passphrase saisie

Pour de la confidentialité forte (contenu sensible, contrôle d'accès révocable par stagiaire), il faut passer à Cloudflare Access, Netlify Identity, ou un backend d'authentification, ce qui sort du cadre GitHub Pages.

---

## Personnalisation

### Slide de couverture

Formateur, contact, session et durée sont dans le bloc `.cover-meta` en tête de `slides.md` — éditer ces lignes pour une nouvelle session.

### Couleurs et styles

Tout est centralisé dans `style.css`. Les variables CSS à connaître :

- `--k8s-blue: #326ce5` : bleu principal
- `--k8s-blue-soft` : fond des callouts et headers de table
- `--k8s-ink` : texte principal

Modifier une variable met à jour tout le diaporama.

---

## Structure du projet

```
presentation/
├── slides.md                       # Contenu de la présentation
├── style.css                       # Palette Kubernetes (auto-chargée par Slidev)
├── components/Quiz.vue             # Composant de quiz interactif
├── package.json                    # Scripts Slidev + staticrypt
└── .github/workflows/deploy.yml    # CI vers GitHub Pages
```

Source pédagogique de référence : `../formation-kubernetes.md`.

