# Templates

Worked examples for branch names, commit messages, and pull request bodies.

## Branch names

Format: `feature/<TICKET>-<kebab-description>` or `bugfix/<TICKET>-<kebab-description>`, in English. Omit the ticket segment when the repository has none.

| Change | Branch |
|---|---|
| Add CSV export to the reports page | `feature/IGIA-290-add-csv-export` |
| Fix the empty cart total at checkout | `bugfix/IGIA-312-fix-empty-cart-total` |
| Add retry and backoff to the HTTP client | `feature/IGIA-301-add-http-retry` |
| Correct the deploy steps in the README | `bugfix/IGIA-288-fix-deploy-steps` |
| Bump CI to Node 22 | `feature/IGIA-275-upgrade-ci-node-22` |

Rules:
- `bugfix/` when the change primarily fixes bugs or corrects behavior.
- `feature/` when the change primarily adds functionality or enhancements.
- Refactors, docs, and chores use `feature/` unless they correct behavior, which uses `bugfix/`.
- Ticket in uppercase right after the prefix, for example `IGIA-290`.
- An override named with the branch request applies to that branch only; the stored default is unchanged.
- Lowercase, hyphens, no issue numbers in the description.

## Gitmoji

The title is `<gitmoji> <type>: <subject>`. Pick the emoji that matches the change.

| Type | Gitmoji | Meaning |
|---|---|---|
| feat | ✨ | new feature |
| fix | 🐛 | bug fix |
| docs | 📝 | documentation |
| refactor | ♻️ | refactor |
| test | ✅ | tests |
| chore | 🔧 | config or tooling |
| perf | ⚡️ | performance |
| ci | 👷 | continuous integration |
| build | 📦 | build or dependencies |
| style | 💄 | formatting |
| remove | 🔥 | remove code or files |
| security | 🔒 | security |
| wip | 🚧 | work in progress |

Count the emoji as one character toward the 50-character title limit. A few emoji are multi-codepoint, so check the rendered length.

## Commit messages

Format: gitmoji, semantic prefix, present tense, French with technical terms in English. Title at or under 50 characters, no trailing period. Body wrapped at 72, explaining what changed and why. No line starts with `#`.

### Feature

```text
✨ feat: ajoute l'export CSV aux rapports

Ajoute un bouton Export sur la page des rapports qui télécharge le
résultat filtré au format CSV. L'export est exécuté côté serveur pour
garder les gros résultats hors du navigateur.

Closes #412
```

### Bug fix

```text
🐛 fix: empêche le double submit au checkout

Le handler désactivait le bouton après la résolution de la requête, ce qui
permettait un second clic pendant que la première requête était en cours.
Désactive le bouton au clic et le réactive seulement en cas d'erreur.
```

### Refactor

```text
♻️ refactor: extrait la retry policy dans un client partagé

Trois call sites dupliquaient la même logique de backoff. Déplace-la dans
http-client pour garder le timeout et le nombre de retries cohérents.
```

### Documentation

```text
📝 docs: documente le processus de déploiement

Ajoute les variables d'environnement et les étapes de rollback qui
n'étaient que dans le chat de l'équipe.
```

### Subject rules

- Present tense: "ajoute", not "ajouté" or "ajout".
- French sentence, English technical terms: `timeout`, `retry`, `call site`, `rollback`, `token`.
- The gitmoji and type prefix stay in the title only.
- No emoji beyond the leading gitmoji.

## Pull request bodies

Sections: Résumé, Motivation, Tests. Same language as the commit messages.

### Example: bug fix

```markdown
## Résumé

Corrige le bouton du checkout qui acceptait un second clic pendant que la
première requête était toujours en cours. Le bouton est maintenant
désactivé au clic et réactivé seulement si la requête échoue.

## Motivation

Les utilisateurs sur une connexion lente pouvaient soumettre la même
commande deux fois, créant une facturation en double. Voir IGIA-312.

## Tests

- `npm test -- checkout`
- Manuel : limiter à Slow 3G, cliquer deux fois sur Pay, vérifier une
  seule commande.
```

### Example: feature

```markdown
## Résumé

Ajoute l'export CSV côté serveur à la page des rapports, en respectant les
filtres courants.

## Motivation

L'équipe finance exporte les données à la main chaque semaine. Ceci
supprime le copier-coller et garde l'export cohérent avec les filtres
affichés. Voir IGIA-290.

## Tests

- `npm test -- reports`
- Manuel : appliquer un filtre de dates, exporter, vérifier que le nombre
  de lignes correspond.
```

### Title rules

- Reuse the single commit subject when the branch has one commit.
- Otherwise summarize the branch in one line, same style as a commit subject.
- Reference the Jira ticket in the body.
