# 03-portfolio-changelog-crew

## Contexte

Projet pedagogique (voir le Project Cowork "Apprentissage du domaine de l'IA agentique avec
Claude" pour l'historique complet — non accessible depuis cette session Claude Code, qui n'a
acces qu'aux fichiers de ce dossier).

Objectif : premier projet d'orchestration multi-agents construit avec Claude Code. Un
orchestrateur delegue a des sous-agents specialises (definis dans `.claude/agents/`) :

- `link-scanner` : extrait les liens GitHub d'un fichier markdown donne en entree et verifie
  qu'ils repondent (HTTP 200).
- `changelog-coder` : clone chaque depot verifie et genere un `CHANGELOG.md` a partir de son
  `git log`.
- `tester` : ecrit et lance des tests sur les cas limites (lien mort, depot vide, URL
  malformee, mention de projet sans lien).
- `reviewer` : relit le code produit par `changelog-coder` avant de le considerer termine.
- `publisher` : commit, push, et ouvre une pull request avec le resultat.

Entree d'exemple pour le developpement : `sample-input/portfolio-snapshot.md` (extrait manuel
d'un document externe a ce depot — voir ce fichier pour le detail). L'outil doit rester
generique : il prend un chemin de fichier markdown en parametre, il ne doit jamais etre code en
dur pour ce fichier precis specifiquement.

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
  l'utilisateur avant le push.
