---
name: run-pipeline
description: Enchaine les 5 sous-agents du pipeline portfolio-changelog dans l'ordre, avec le filtrage de donnees correct entre chaque etape.
---

1. `link-scanner` sur le fichier markdown d'entree.
2. Filtrer sa sortie JSON sur `status: "valid"` → `changelog-coder`.
3. `changelog-coder` → recuperer les chemins `output/<owner>-<repo>/CHANGELOG.md` produits.
4. `tester` : lui donner les memes entrees + sorties reelles pour verification (voir tester.md pour le protocole phase 1/2).
5. `reviewer` : lui donner le rapport de `tester` + les fichiers generes.
6. Si `reviewer` approuve → `publisher`, un appel par depot cible, avec le verdict + chemin du changelog.
