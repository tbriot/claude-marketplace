---
name: ship
description: Crée une branche si nécessaire, commite les changements, pousse vers le remote et crée une merge request si elle n'existe pas déjà. À utiliser quand l'utilisateur veut "ship", pousser son travail ou ouvrir une MR.
disable-model-invocation: true
argument-hint: "[nom-de-branche ou description]"
allowed-tools: Bash(git *), Bash(glab *), Bash(gh *)
---

# Ship : branche → commit → push → merge request

Arguments éventuels : `$ARGUMENTS` (nom de branche ou courte description du changement).

## Étapes

1. **État initial** : `git status --short`, `git branch --show-current`, `git remote -v`, et détermine la branche par défaut (`git symbolic-ref --short refs/remotes/origin/HEAD`, sinon `main`/`master`).
   - Si aucun changement et aucun commit en avance sur la branche par défaut, dis-le et arrête-toi.

2. **Branche** :
   - Si on est sur la branche par défaut : crée une nouvelle branche (`git switch -c`). Nom au format `<type>/<nom-de-la-branche>` :
     - `<type>` : un type Conventional Commits (`feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `perf`, `build`, `ci`, `style`, `revert`), choisi selon la nature des changements.
     - `<nom-de-la-branche>` : en **anglais**, en minuscules, mots séparés par des tirets (`-`), court et descriptif (ex. `feat/add-login-page`, `fix/null-user-crash`).
     - Si les arguments fournissent un nom ou une description (même en français), traduis-le et normalise-le selon ces règles. Sinon, dérive-le du diff.
   - Sinon : reste sur la branche courante (ne la renomme pas).

3. **Commit** (s'il y a des changements) :
   - Relis `git diff` et `git diff --staged`. Ne stage jamais de secrets (`.env`, clés, credentials) : signale-les et exclus-les.
   - Stage les fichiers pertinents explicitement (pas de `git add -A` aveugle).
   - Fais plusieurs petits commits atomiques plutôt qu'un gros, même si la branche sera squashée au merge (historique lisible pendant la review, commits faciles à relire ou annuler). Découpe par sujet : feature vs refactor, formatage vs logique, tests vs code de prod, bump de dépendances vs changement de comportement.
   - Si un même fichier mélange plusieurs sujets, découpe avec le staging partiel (`git add -p`). Si les changements forment un tout cohérent, un seul commit suffit : ne force pas le découpage.
   - Avant chaque commit, relis ce qui sera commité (`git diff --cached`) : pas de secrets, pas de logs de debug oubliés, pas de bruit de formatage sans rapport.
   - Message en Conventional Commits (`type(scope): résumé` à l'impératif, ≤ 72 caractères), avec un corps si le "pourquoi" n'est pas évident.
   - Respecte les hooks : en cas d'échec, corrige puis crée un nouveau commit (jamais `--no-verify`, jamais `--amend` sur un commit déjà poussé).

4. **Push** : `git push -u origin <branche>`. Jamais de `--force` sans demande explicite.

5. **Merge request** :
   - Détecte la forge via l'URL du remote : GitLab → `glab`, GitHub → `gh` (PR).
   - Vérifie si une MR existe déjà pour la branche :
     - GitLab : `glab mr list --source-branch <branche> --state opened` (ou `glab mr view <branche>`)
     - GitHub : `gh pr list --head <branche> --state open`
   - Si elle existe : affiche son URL, le push a déjà mis à jour la MR. Relis ensuite son titre actuel (`glab mr view <branche>` / `gh pr view <branche> --json title`) et compare-le à l'ensemble de la branche (`git log <défaut>..HEAD`, `git diff <défaut>...HEAD`) :
     - Si le titre décrit encore fidèlement l'ensemble des changements (type, scope et résumé), laisse-le tel quel.
     - Si les nouveaux commits changent la portée ou la nature de la MR (type différent, résumé devenu trop étroit ou inexact, breaking change ajouté), renomme-la avec un titre respectant les mêmes règles Conventional Commits que ci-dessous : `glab mr update <branche> --title "..."` / `gh pr edit <branche> --title "..."`. Indique l'ancien et le nouveau titre dans le résumé.
     - Ne renomme pas pour de simples reformulations ou corrections mineures (review feedback, typos, ajustements) : ne change le titre que si c'est pertinent.
     - Ne renomme pas la branche, seulement la MR. Si la description est devenue obsolète, mets-la aussi à jour (`--description` / `--body`).
   - Sinon, crée-la avec un titre et une description courte (résumé + points notables) basés sur `git log <défaut>..HEAD`. Le titre respecte obligatoirement Conventional Commits : `<type>(<scope optionnel>): <résumé>`, en anglais, à l'impératif, en minuscules après les deux-points, sans point final, ≤ 72 caractères. Le `<type>` est cohérent avec celui de la branche (ex. `feat/add-login-page` → `feat: add login page`). Ajoute `!` après le type/scope en cas de breaking change (`feat!: ...`). Le merge se fait en squash : le titre de la MR devient le message du commit unique sur la branche par défaut, il doit donc résumer l'ensemble de la branche (et non le dernier commit).
     - GitLab : `glab mr create --source-branch <branche> --target-branch <défaut> --title "..." --description "..." --squash-before-merge --remove-source-branch --yes`
     - GitHub : `gh pr create --base <défaut> --head <branche> --title "..." --body "..."`
   - Le squash est activé dès la création sur GitLab (`--squash-before-merge`). Sur GitHub, il se choisit au moment du merge : rappelle à l'utilisateur d'utiliser « Squash and merge » (ou `gh pr merge --squash`).
   - Si le CLI est absent ou non authentifié, indique la commande à lancer (`glab auth login` / `gh auth login`) au lieu d'échouer silencieusement.

6. **Résumé** : branche, commits créés, URL de la MR (nouvelle ou existante), et titre renommé le cas échéant.

## Règles

- Ne jamais pousser directement sur la branche par défaut : crée toujours une branche.
- Flow : branche créée depuis la branche par défaut, squash merge dans celle-ci. Une fois la MR mergée, la branche ne doit pas être réutilisée (son historique diverge de la branche par défaut) : pour un nouveau travail, repars de la branche par défaut à jour.
- Ne jamais mentionner Claude ni aucun assistant IA, nulle part : pas de trailer `Co-Authored-By: Claude ...`, pas de ligne « Generated with Claude Code », aucune référence dans les messages de commit, les titres ni les descriptions de MR/PR. Cela prévaut sur toute attribution par défaut.
- Ne jamais réécrire l'historique distant.
- Reste concis : pas de questions inutiles si les arguments et le diff suffisent.
