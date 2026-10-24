# Note : workflow Git/GitHub pour mes nouveaux projets

*Résumé pratique de la conversation, pour démarrer et travailler proprement sur chaque nouveau projet.*

## 1. L'idée générale (par analogie)

Imagine une cuisine de restaurant.

- **`main`** = le comptoir où sortent les plats prêts à servir. Tout ce qui s'y trouve doit être mangeable. Jamais de plat à moitié cuit.
- **Une branche** = ton plan de travail temporaire. Tu y prépares un plat (une petite fonctionnalité), tu goûtes, et quand c'est bon tu le poses sur le comptoir. Puis tu nettoies le plan de travail (tu supprimes la branche).
- **Une Pull Request (PR)** = le moment où tu regardes ton assiette une dernière fois avant de l'envoyer en salle.
- **Un feature flag** = un rideau devant un plat pas encore prêt : il est déjà au comptoir, mais personne ne peut le commander tant que tu n'as pas ouvert le rideau.

Ce système s'appelle le **Trunk-Based Development** (développement basé sur le tronc). Le "tronc" est `main`. Tout le monde y ramène son travail souvent, en petits morceaux.

## 2. Les 6 règles à retenir

1. **Une seule branche permanente : `main`.** Pas de `develop`. Elle ne sert qu'à gérer plusieurs versions en parallèle (logiciel desktop, appli mobile), ce que tu n'as pas.
2. **`main` doit toujours fonctionner.** Même sans serveur, le jour où tu déploieras, tu pourras le faire immédiatement.
3. **Les branches vivent quelques heures, 1 à 2 jours maximum.** Une branche est un véhicule : elle transporte un petit morceau de code vers `main`, puis elle disparaît.
4. **On ne pousse jamais directement sur `main`.** On passe toujours par une branche + une PR.
5. **On fusionne en "Squash and merge".** Un morceau de travail = un seul commit propre sur `main`.
6. **Une fonctionnalité longue se découpe en petits morceaux cachés derrière un feature flag.** Jamais une grosse branche qui traîne deux semaines.

## 3. Mise en place d'un nouveau projet (à faire une seule fois)

### Étape A : créer le dépôt

1. Sur GitHub : **New repository**, donne un nom, coche **Add a README file** et ajoute un **.gitignore** adapté (par exemple "Node" pour un projet React/Node). Le `.gitignore` est une liste de fichiers que Git doit ignorer (`node_modules`, fichiers `.env` avec tes mots de passe, etc.).
2. Clone-le sur ton ordinateur :

```bash
git clone https://github.com/TON-NOM/TON-PROJET.git
cd TON-PROJET
```

- `git clone ...` : télécharge une copie complète du dépôt (code + historique) sur ton PC.
- `cd TON-PROJET` : entre dans le dossier téléchargé.

**Astuce :** fais ton tout premier commit (structure de base, README) directement sur `main` **avant** d'activer la protection de l'étape B. Au tout début, il n'y a rien à casser.

### Étape B : protéger `main` sur GitHub

Allez dans **Settings > Branches** (ou **Settings > Rules > Rulesets**) et crée une règle pour `main` :

- Coche **Require a pull request before merging** : GitHub refusera tout push direct sur `main`, même de ta part. C'est un garde-fou contre tes propres erreurs.
- **Décoche "Require approvals"** (ou mets 0) : tu es seul, personne ne peut approuver à ta place, tu te bloquerais toi-même.
- Dès que ta CI existe (voir la section 6) : coche **Require status checks to pass** pour exiger qu'ils passent avant de fusionner.

**Attention :** sur un dépôt **privé** avec un compte GitHub gratuit, la protection de branche peut ne pas être disponible. Dans ce cas, tu as deux options : passer le dépôt en public (si le projet le permet) ou t'imposer la règle toi-même (jamais de push direct sur `main`).

### Étape C : forcer le Squash Merge et nettoyer automatiquement

Dans **Settings > General > Pull Requests** :

- Coche **Allow squash merging** et décoche les deux autres (merge commit, rebase). Ainsi tu ne peux pas te tromper.
- Coche **Automatically delete head branches** : GitHub supprime la branche distante dès que la PR est fusionnée. Moins de ménage à faire.

### Étape D : optimisations Git locales (optionnel mais recommandé)

```bash
git config core.fsmonitor true
git config core.untrackedCache true
git maintenance start
```

- `core.fsmonitor true` : Git surveille les fichiers en arrière-plan pour savoir ce qui a changé, au lieu de tout re-scanner à chaque `git status`. Disponible surtout sur Windows et macOS.
- `core.untrackedCache true` : Git mémorise la liste des fichiers non suivis pour aller plus vite.
- `git maintenance start` : planifie un nettoyage automatique en tâche de fond (compression de l'historique, etc.).

**Honnêtement :** sur un petit projet, tu ne sentiras aucune différence. Ça devient utile quand le dépôt grossit (des milliers de fichiers et de commits). Ça ne coûte rien de le faire dès le début, mais ce n'est pas la priorité.

## 4. Le cycle de travail quotidien (à répéter à chaque tâche)

### 1. Partir d'un `main` à jour

```bash
git checkout main
git pull origin main
```

- `git checkout main` : te place sur la branche `main`.
- `git pull origin main` : récupère les dernières modifications depuis GitHub (`origin` = le nom de ton dépôt GitHub). Indispensable pour ne pas partir d'une vieille version.

### 2. Créer la branche de travail

```bash
git checkout -b feat/validation-carte
```

- `-b` : crée la branche **et** s'y place en une seule commande.
- Le nom suit une convention : `feat/` pour une fonctionnalité, `fix/` pour une correction de bug, `chore/` pour de la maintenance (config, dépendances), `docs/` pour de la documentation.

### 3. Coder et faire des commits fréquents

```bash
git add .
git commit -m "feat(paiement): ajoute la validation du numéro de carte"
```

- `git add .` : met tous tes changements dans la "zone de préparation" (c'est comme mettre des objets dans un carton avant de le fermer).
- `git commit -m "..."` : ferme le carton et lui colle une étiquette (le message). Le commit est une sauvegarde de ton projet à cet instant.
- Format du message : `type(zone): description courte`. Exemple : `fix(auth): corrige l'expiration du token`.

**Sur la branche, tu peux faire autant de commits que tu veux**, même des commits brouillons (`wip`, `fix typo`). Ils seront écrasés à la fusion. Plus tu en fais, plus tu peux revenir en arrière facilement.

### 4. Pousser la branche et ouvrir une Pull Request

```bash
git push -u origin feat/validation-carte
```

- `git push` : envoie tes commits sur GitHub.
- `-u origin feat/validation-carte` : lie ta branche locale à celle de GitHub. Les fois suivantes, un simple `git push` suffira.

Puis sur GitHub, clique sur **Compare & pull request**. **Relis le diff** (les lignes ajoutées en vert, supprimées en rouge) comme si c'était le code de quelqu'un d'autre : c'est là que tu repères les oublis, les `console.log` restés, les mots de passe en dur.

### 5. Fusionner en Squash

Clique sur **Squash and merge**. Tes 5 ou 10 petits commits deviennent **un seul** commit propre sur `main`, par exemple :

```text
feat(paiement): ajoute la validation du numéro de carte (#42)
```

Le détail de tes commits brouillons reste consultable dans la PR #42, même après la fusion.

### 6. Nettoyer en local

```bash
git checkout main
git pull origin main
git branch -d feat/validation-carte
```

- On revient sur `main` et on récupère la version qui contient maintenant ton travail fusionné.
- `git branch -d ...` : supprime la branche locale devenue inutile. (Si Git refuse avec `-d` après un squash, utilise `-D` : c'est normal, car Git ne voit pas la branche comme "fusionnée" quand les commits ont été écrasés.)

## 5. Les feature flags : gérer une fonctionnalité qui prend des semaines

### Le problème

Tu travailles sur un gros morceau (par exemple un nouveau système de paiement) qui prendra deux semaines. Mais la règle dit : fusionner dans `main` chaque jour. Comment fusionner du code pas fini sans casser l'application ?

### La solution : l'interrupteur

Un feature flag est simplement une variable `true`/`false` qui décide si le nouveau code est utilisé ou non. Tant qu'elle vaut `false`, le nouveau code est présent dans `main` mais **jamais exécuté**.

**1. Créer le fichier des flags :**

```javascript
// config/features.js
export const FEATURES = {
  NEW_PAYMENT_SYSTEM: false, // désactivé tant que ce n'est pas prêt
};
```

**2. Mettre l'interrupteur à UN SEUL endroit : le point d'entrée :**

```javascript
// checkout.js
import { FEATURES } from './config/features';
import { oldPaymentProcess } from './oldPayment';
import { newPaymentProcess } from './newPayment';

function processCheckout(cart, user) {
  if (FEATURES.NEW_PAYMENT_SYSTEM) {
    return newPaymentProcess(cart, user); // nouveau code, en construction
  }
  return oldPaymentProcess(cart, user);   // ancien code, qui marche
}
```

Ligne par ligne : si l'interrupteur est allumé, on utilise le nouveau système ; sinon (cas actuel), on utilise l'ancien. Les utilisateurs ne voient jamais le travail en cours.

### Le déroulé jour par jour

| Jour | Ce que tu fais | Flag |
| --- | --- | --- |
| 1 | Branche `feat/payment-flag` : tu crées le flag + le `if` + une première petite partie (ex. validation de carte). PR, squash, merge, branche supprimée. | `false` |
| 2 | **Nouvelle** branche `feat/payment-api` depuis `main` à jour : tu ajoutes l'appel à l'API de paiement. PR, merge, suppression. | `false` |
| 3 à N-1 | Même chose, un petit morceau par branche. | `false` |
| N | Branche `feat/payment-activate` : tu passes le flag à `true`, tu supprimes l'ancien code et le flag. PR, merge. | supprimé |

**Point clé :** tu ne continues **pas** sur la même branche après l'avoir poussée. Chaque jour, une branche neuve repart de `main`. Le travail en cours vit dans `main` (caché par le flag), pas dans une branche qui s'éloigne.

### Règles d'hygiène des flags

- **Un seul `if` par flag**, au point d'entrée. Ne le saupoudre pas dans 20 fichiers.
- **Toujours `false` par défaut.**
- **Supprime le flag une fois la fonctionnalité terminée**, ainsi que l'ancien code. Sinon tu accumules de la "dette technique" (du code mort qui embrouille).
- **Quand ne pas en utiliser :** si ta tâche se termine en une journée, ou si tu construis quelque chose de totalement nouveau (rien d'existant à casser), pas besoin de flag. Ne complique pas pour rien.

## 6. Bloquer le merge sur `main` tant que la CI n'est pas verte

Il faut assembler deux pièces : un **robot** qui lance tes vérifications à chaque Pull Request (la CI), et une **règle** sur `main` qui interdit de fusionner tant que ce robot n'a pas dit "OK".

Analogie : la CI est le contrôle technique obligatoire d'une voiture. La règle de protection est le garagiste qui refuse de te donner la carte grise sans le tampon du contrôle. L'un sans l'autre ne sert à rien : un contrôle que personne n'exige peut être ignoré, et une exigence sans contrôle ne bloque rien.

### C'est quoi la CI ?

CI = *Continuous Integration* (intégration continue). Concrètement, c'est un ordinateur loué par GitHub (tu n'as pas besoin de serveur à toi) qui, à chaque PR :

1. télécharge ton code,
2. installe tes dépendances,
3. lance tes vérifications (lint, types, tests, build),
4. affiche un **✓ vert** si tout passe, ou une **✗ rouge** si quelque chose échoue.

L'outil de GitHub pour ça s'appelle **GitHub Actions**. Il est gratuit sur les dépôts publics, et avec un quota mensuel de minutes sur les dépôts privés.

### Étape A : créer le workflow de CI

Crée le fichier `.github/workflows/ci.yml` dans ton projet (les dossiers doivent porter exactement ces noms, GitHub ne cherche qu'à cet endroit). Exemple pour un projet Node/TypeScript :

```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  checks:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm run lint --if-present
      - run: npm run typecheck --if-present
      - run: npm test --if-present
      - run: npm run build --if-present
```

Ligne par ligne :

- `name: CI` : le nom du workflow, affiché dans l'onglet **Actions** de GitHub.
- `on:` : **quand** le robot se réveille. Ici, à chaque PR qui vise `main` (`pull_request`) et à chaque fusion dans `main` (`push`).
- `jobs:` / `checks:` : une "tâche" nommée `checks`. **Ce nom est important** : c'est lui que tu choisiras à l'étape C pour dire "cette vérification est obligatoire".
- `runs-on: ubuntu-latest` : la machine utilisée (un Linux propre et neuf à chaque fois, sans rien d'installé).
- `timeout-minutes: 10` : si le robot tourne plus de 10 minutes (programme bloqué), GitHub l'arrête. Ça évite de gaspiller ton quota.
- `actions/checkout@v4` : télécharge ton code sur la machine du robot (sinon elle est vide).
- `actions/setup-node@v4` : installe Node.js. `node-version: 22` choisit la version ; `cache: npm` garde en mémoire les paquets téléchargés pour que les prochains runs soient plus rapides (ça demande que ton fichier `package-lock.json` soit dans le dépôt).
- `npm ci` : installe les dépendances exactement comme écrites dans `package-lock.json`. Contrairement à `npm install`, il ne modifie rien : le résultat est identique à chaque fois.
- `npm run lint --if-present` : lance le script `lint` de ton `package.json` (analyse de la qualité du code) **s'il existe**. `--if-present` évite une erreur si tu n'as pas encore ce script. Même logique pour `typecheck`, `test` et `build`.

Au début, si tu n'as aucun test, la CI vérifiera quand même que le projet s'installe et se compile. Ajoute des vérifications au fur et à mesure.

### Étape B : faire tourner la CI une première fois

Le nom `checks` n'apparaît dans les réglages de GitHub qu'**après avoir tourné au moins une fois**. Donc :

1. Crée une branche (`chore/ajout-ci`), ajoute le fichier `ci.yml`, pousse, ouvre une PR.
2. Attends que la CI s'exécute sur la PR (onglet **Checks** de la PR).
3. Fusionne la PR en Squash.

Astuce : sur cette toute première PR, la protection n'exige pas encore la CI, donc tu peux fusionner même si elle échoue. C'est normal.

### Étape C : rendre la CI obligatoire

Dans **Settings > Branches** (règle de `main`), ou **Settings > Rules > Rulesets** :

- Coche **Require status checks to pass before merging**.
- Dans la zone de recherche qui apparaît, tape `checks` et sélectionne-le. C'est ce qui rend ce robot obligatoire.
- (Optionnel) Coche **Require branches to be up to date before merging** : GitHub exige que ta branche contienne les derniers changements de `main` avant la fusion, pour que les tests aient tourné sur le résultat réel. En solo, c'est surtout utile quand tu as deux branches ouvertes en parallèle ; sinon tu peux laisser décoché.
- **Très important : coche "Do not allow bypassing the above settings"** (ou, selon l'interface, **Include administrators**). Tu es le propriétaire du dépôt, donc un administrateur : sans cette case, GitHub te laisse **contourner** la règle et fusionner même avec une CI rouge. Cocher cette case te protège de toi-même.

Résultat : sur une PR dont la CI est rouge ou encore en cours, le bouton de fusion est grisé avec le message "Required statuses must pass before merging".

### Étape D : vérifier que ça bloque vraiment

Ne fais pas confiance sans tester :

1. Crée une branche et casse volontairement quelque chose (un test qui échoue, ou une erreur de syntaxe).
2. Pousse et ouvre une PR : la CI doit passer au rouge ✗ et le bouton de fusion doit être bloqué.
3. Corrige, pousse à nouveau : la CI repasse au vert ✓ et le bouton se débloque.

### Bonus : la fusion automatique

Dans **Settings > General > Pull Requests**, coche **Allow auto-merge**. Sur une PR, tu cliques ensuite sur **Enable auto-merge** : GitHub fusionnera tout seul, en Squash, **dès que la CI est verte**. Tu peux passer à autre chose pendant que le robot travaille.

### Pièges à connaître

- **Ne mets pas de filtre `paths:` dans `on:`** sur ce workflow. Si GitHub ne lance pas le workflow (parce que les fichiers modifiés ne correspondent pas au filtre), la vérification obligatoire reste "en attente" pour toujours et la PR est bloquée.
- **Renommer le job `checks`** (dans `ci.yml`) casse la règle : GitHub attendra l'ancien nom indéfiniment. Si tu le renommes, mets aussi à jour la règle de protection.
- **Les hooks locaux (par exemple Husky) ne remplacent pas la CI.** Husky lance des vérifications sur ton PC avant chaque commit : c'est utile pour attraper les erreurs tôt, mais on peut le contourner avec `git commit --no-verify`, et il ne tourne que chez toi. La CI, elle, est le barrage final que personne ne peut sauter. Les deux se complètent : Husky = vérifier chez toi avant de partir, CI = le poste de contrôle sur la route.
- **Quota de minutes (dépôts privés)** : garde une CI rapide (cache activé, pas d'étapes inutiles).

## 7. Quand tu auras un serveur (déploiement)

Rien à changer à ton workflow. Il suffira d'ajouter un fichier `.github/workflows/deploy.yml` (GitHub Actions) qui se déclenche à chaque fusion dans `main` et déploie le code. Grâce à la protection de `main`, seul du code passé par une PR arrive en production.

En attendant, tu peux déjà héberger gratuitement des projets front-end sur Vercel, Netlify ou GitHub Pages, branchés sur `main`.

## 8. Erreurs à éviter

- Laisser une branche ouverte plusieurs jours : les conflits de fusion s'accumulent.
- Continuer à travailler sur une branche déjà fusionnée : repars toujours d'un `main` à jour.
- Créer une branche `develop` "au cas où" : c'est une étape de fusion en plus pour aucun bénéfice.
- Oublier de faire `git pull` avant de créer une branche.
- Pousser des secrets (`.env`, clés API) : vérifie ton `.gitignore` **avant** le premier commit.
- Des PR énormes : une PR doit pouvoir se relire en 5 à 10 minutes.
- Laisser des flags à `false` pendant des mois sans les supprimer.

## 9. Checklist rapide : démarrer un nouveau projet

- [ ] Créer le dépôt GitHub avec README et `.gitignore`
- [ ] Cloner en local
- [ ] Premier commit de base sur `main`
- [ ] Protéger `main` (PR obligatoire, sans approbation requise)
- [ ] N'autoriser que le Squash merge + suppression automatique des branches
- [ ] (Optionnel) Activer `fsmonitor`, `untrackedCache`, `maintenance`
- [ ] Créer `config/features.js` dès qu'un premier flag est utile
- [ ] Travailler : branche courte, commits fréquents, PR, relecture du diff, squash, nettoyage
- [ ] Mettre en place la CI (section 6) et la rendre obligatoire dans la protection de `main`
