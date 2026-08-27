# 03-portfolio-changelog-crew

## Contexte

Objectif : premier projet d'orchestration multi-agents construit avec Claude Code. Un
orchestrateur delegue a des sous-agents specialises (definis dans `.claude/agents/`) :

- `link-scanner` : extrait les liens GitHub d'un fichier markdown donne en entree et verifie
  qu'ils repondent (HTTP 200).
- `changelog-coder` : clone chaque depot verifie et genere un `CHANGELOG.md` a partir de son
  `git log`.
- `tester` : ecrit et lance des tests sur les cas limites (lien mort, depot vide, URL
  malformee, mention de projet sans lien).
- `reviewer` : relit le code produit par `changelog-coder` avant de le considerer termine.
- `publisher` : commit, push, et ouvre une pull request avec le resultat **sur le depot cible
  scanne** (celui dont le lien a ete verifie par `link-scanner`), **jamais sur ce depot pipeline
  lui-meme** (`03-portfolio-changelog-crew`).

Les fichiers generes dans `output/` (les `CHANGELOG.md` intermediaires produits par
`changelog-coder`) sont ephemeres : ils ne sont jamais commites dans ce depot (`.gitignore`), et
leur seule destination est d'etre pousses vers leur depot cible par `publisher`. Ne pas les
supprimer avant la fin complete d'un run (le `reviewer` et le `publisher` en ont besoin sur
disque) — les nettoyer seulement une fois le run termine.

Entree d'exemple pour le developpement : `sample-input/portfolio-snapshot.md` (extrait manuel
d'un document externe a ce depot — voir ce fichier pour le detail). L'outil doit rester
generique : il prend un chemin de fichier markdown en parametre, il ne doit jamais etre code en
dur pour ce fichier precis specifiquement.

## Workflow git (regle explicite, pour ne pas dependre d'une correction humaine a chaque fois)

- Ne jamais committer ni pousser directement sur `main`. Toute modification passe par une
  branche dediee (`git checkout -b <nom-de-branche>`), poussee sur `origin`, puis une pull
  request — meme pour un petit changement.
- Ouvrir la pull request tot (des qu'un morceau coherent est pret, ex: les definitions de
  sous-agents) plutot que d'attendre l'implementation complete — des PR incrementales, pas une
  seule PR geante en fin de projet.
- Le merge reste une action humaine, jamais automatique (voir `.claude/agents/publisher.md` —
  le publisher s'arrete a l'ouverture de la PR).

## Prerequis environnement

- `gh` CLI doit etre installe et authentifie avant toute operation GitHub. Si absent :
  installer via `winget install --id GitHub.cli --silent --accept-package-agreements
  --accept-source-agreements` sans demander confirmation (prerequis d'outillage, pas une
  decision produit), puis lancer `gh auth login --hostname github.com --git-protocol https
  --web` — la validation du code affiche reste necessairement une etape humaine (device flow),
  impossible a automatiser davantage.
- Le token `gh` par defaut (`repo`, `read:org`, `gist`) ne couvre pas `delete_repo` — c'est
  voulu, ne pas elargir ce scope pour un besoin ponctuel de nettoyage. Un depot jetable cree
  pour tester `publisher` (ou autre) doit etre supprime manuellement par l'utilisateur (lien
  vers Settings > Danger Zone), pas via une elevation de scope du token. Le risque (suppression
  accidentelle ou non autorisee d'un depot quelconque du compte) depasse largement le gain
  (eviter un clic manuel occasionnel).

## Principes d'architecture (deduits des decisions prises pendant le developpement)

- **Separation trouver / corriger** : un agent qui detecte un probleme (ex. `tester`) ne le
  corrige jamais lui-meme, meme s'il en a techniquement les moyens. Il rapporte ; un autre agent
  (ou l'orchestrateur) decide de la correction. Objectif : eviter qu'un agent se convainque
  lui-meme que son propre travail est correct.
- **Seul l'orchestrateur invoque des sous-agents** : aucun sous-agent ne recoit l'outil
  `Agent`/`Task` pour en invoquer un autre. Garde le cout, la latence et le determinisme de la
  boucle sous le controle de l'orchestrateur plutot que de les laisser decider entre eux
  d'enchainer des appels LLM.
- **Choix du modele par sous-agent** : haiku pour une tache mecanique (extraction,
  classification, verification HTTP — pas de jugement ambigu requis) ; sonnet pour une tache qui
  implique du jugement, de la synthese ou de la composition de texte coherent (revue de code,
  redaction de description de PR, conception de fixtures de test). Trancher ce choix a la
  conception initiale de l'agent, pas apres coup.

## Modification des definitions de sous-agents en cours de session

Editer un fichier `.claude/agents/*.md` en cours de session ne garantit pas que les invocations
suivantes de ce sous-agent refletent la nouvelle version (comportement de cache du harness
Claude Code observe empiriquement, non documente officiellement a ce jour). Consequence
pratique apres une modification significative (pas une simple typo) :

- Signe revelateur : le comportement reel de l'agent ne change pas malgre l'edition (ex. il
  paraphrase encore l'ancienne version de ses instructions).
- En cas de doute, demander a l'utilisateur de relancer la session (fermer/rouvrir Claude Code,
  ou `/resume`) avant de re-tester — seule methode confirmee qui fonctionne. Ne pas perdre de
  temps a renforcer le prompt de tache pour contourner le probleme : ca ne corrige pas la cause
  (le system prompt lui-meme est perime).
- Envisager de signaler ce comportement via `/feedback` si ca se reproduit.

## Conventions (heritees du projet global, resume pertinent pour ce depot)

- Code (identifiants, chaines, messages de log) en anglais des le depart, quel que soit le
  stade.
- Commentaires pedagogiques (explication d'une notion agentique, destines a disparaitre avant
  la version showcase) en francais.
- Commentaires destines a rester (intention d'un bloc, choix d'architecture non triviaux) en
  anglais.
- Avant de pousser en version showcase : README complet (ce que fait le projet, competences
  agentiques demontrees, architecture, comment le lancer) + nettoyage des traces pedagogiques.
- Depot GitHub prive par defaut ; passage en public au moment showcase, a confirmer avec
  l'utilisateur avant le push. Attention a ne pas confondre "public" (reglage de visibilite
  GitHub, binaire) et "showcase" (readiness du contenu : README complet + nettoyage des traces
  pedagogiques, cf. ligne precedente) — un depot peut etre public sans etre showcase, les deux
  ne sont pas correles.
