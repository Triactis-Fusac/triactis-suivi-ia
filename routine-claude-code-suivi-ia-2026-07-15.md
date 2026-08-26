# Routine Claude Code, complément de la passe Cowork « MAJ avancement suivi IA »

Version 2026-07-15. But : couvrir l'angle mort de la passe Cowork, qui ne voit ni les sessions Claude Code ni l'activité des dépôts de code. Cette routine tourne dans Claude Code, récolte l'avancement côté code, et dépose des candidats dans le cerveau. Elle n'écrit jamais le board : la passe Cowork reste le seul écrivain sur Firestore et consomme ces candidats.

Architecture retenue : producteur (cette routine, Claude Code) vers consommateur (la passe Cowork). Un seul écrivain sur le board, pas de conflit, pas de double comptage.

---

## A. Le prompt (corps de la routine, à coller tel quel)

Tâche : passe quotidienne de récolte de l'avancement du chantier d'implémentation IA de Triactis, côté Claude Code, en complément de la passe Cowork « MAJ avancement suivi IA ». Rédige tout en français professionnel, sans jamais utiliser de tiret cadratin. Exécution autonome (run planifié, Oscar absent) : fais des choix raisonnables et note-les dans la sortie. N'exécute aucune action d'écriture externe (pas de board, pas de push, pas d'envoi).

RÔLE ET COMPLÉMENTARITÉ
- La passe Cowork couvre les sessions Cowork et écrit le board (Firestore) via le navigateur. Elle ne voit pas les sessions Claude Code ni l'historique git des dépôts d'outils.
- Cette routine tourne dans Claude Code, là où sont visibles les transcripts Claude Code et les dépôts de code. Elle ne touche pas le board : principe d'un seul écrivain sur Firestore. Elle produit un dépôt de candidats dans le cerveau, que la passe Cowork consommera pour écrire le board (réconciliation live et anti-doublon faits par elle).

CONTEXTE
- Utilisateur : Oscar Grima (Triactis). Un libellé « PG Constructions » ou « Pierre Grima » est le nom du compte Claude, jamais une entité réelle.
- Cerveau Obsidian : vault Cerveau-claude, dossier C:\Users\OscarGRIMA\Claude\Projects\obsidian-cerveau-master\Cerveau-claude. Lire en premier _system/claude-operating-contract.md, puis _log/CHANGELOG.md. Accès fichier direct (lecture et écriture), c'est un dépôt local.
- Board (pour information seulement, ne pas écrire) : 3 projets parents P1 Architecture, P2 Modélisation, P3 Info-Mémos ; convention : statut Bloqué = tag Blocage, item terminé = progress 100, tâche créée par une passe = tag auto.
- Périmètre strict : uniquement la construction des outils IA (P1/P2/P3). L'enrichissement des dossiers M&A (fichiers mvXXXX, valorisations par dossier, corrections Zoho de deal) est du savoir métier, hors board.

SOURCES À BALAYER (uniquement depuis la dernière passe Claude Code)
- Repère la dernière passe grâce au dernier marqueur « MAJ Claude Code suivi IA <date> » dans _log/CHANGELOG.md et ne traite que ce qui est postérieur. Si aucun marqueur n'existe encore, prends les 7 derniers jours et signale-le.
1. Historique git des dépôts d'outils IA (source primaire, la plus fiable et horodatée). Pour chaque dépôt : `git log --since="<derniere passe>" --stat`, `git branch -a --sort=-committerdate`, tags récents, et l'état de travail non commité (`git status`, `git stash list`). Dépôts connus, mapping vers projet à confirmer par Oscar :
   - cerveau-mcp vers P1 (items a16 cerveau-mcp, a25 branchement MCP dans Claude Desktop)
   - triactis-pipeline (chantier im-automation) vers P3 Info-Mémos
   - triactis-Partie-Financiere (webapp modèle financier) vers P2 Modélisation
   - triactis-suivi-ia (l'appli board elle-même) : méta, à ne porter au board que si Oscar le décide explicitement
   - le cas échéant, un dépôt de skills.
2. Transcripts Claude Code : C:\Users\OscarGRIMA\.claude\projects\**\*.jsonl modifiés depuis la dernière passe (source secondaire, plus bruitée). En extraire les décisions d'architecture et les livraisons d'outils, pas le détail des échanges.
Ne pas balayer Leexi ni les sessions Cowork (couvertes par l'autre passe).

ÉTAPES
1. Établir la fenêtre de traitement depuis le dernier marqueur.
2. Collecter les preuves (commits, tags, fichiers livrés, décisions de session), par dépôt, chacune rattachée à un projet P1/P2/P3.
3. Pour chaque chantier avec preuve d'avancement, préparer un candidat : item de board (id si connu, sinon titre exact probable), proposition d'avant vers après en pourcentage justifiée par la preuve (hash de commit, date, nom de session), statut proposé. Proposer une tâche auto (tag auto) seulement si un chantier réalisé ou clairement engagé n'a pas d'équivalent au board. En cas de doute, lister sans trancher.
4. Blocages : signaler tout blocage matérialisé côté code (dépendance manquante, interdiction IT, geste manuel hors sandbox requis, build cassé).
5. Déposer la sortie dans le cerveau, sans écrire le board :
   - créer 00-inbox/claude-code-avancement-<date>.md avec frontmatter (type: session, tags: [inbox, claude-code, avancement, auto], created, updated) et des candidats structurés : projet parent, item visé, avant vers après, justification et source, tâches auto proposées, blocages ;
   - journaliser dans _log/CHANGELOG.md une entrée datée (kebab-case, aucune perte d'information, aucun cadratin, marqueur en fin), en respectant les conventions du cerveau ; mettre à jour le champ updated des notes touchées ;
   - terminer l'entrée par le marqueur « MAJ Claude Code suivi IA <date du jour> » pour la prochaine passe.
6. Résumé final (notification de fin) : dépôts balayés, candidats proposés (avant vers après), tâches auto proposées, blocages détectés, en rappelant que l'écriture du board revient à la passe Cowork.

GARDE-FOUS
- N'écris jamais le board (ni Firestore direct, ni via navigateur). Rôle de producteur uniquement.
- Ne propose un relèvement de pourcentage que si la preuve dépasse la valeur probable au board ; jamais de baisse ; une édition humaine prime. La passe Cowork fera la réconciliation finale contre le board live et l'anti-doublon.
- Ne propose que des avancements, des statuts et des tâches taguées auto. Ne renomme ni ne supprime aucun élément.
- Aucune action sensible : aucun secret manipulé, aucun `git push`, aucun `git commit` (lecture seule de l'historique), aucun déploiement, aucune dépense.
- Confidentialité M&A : le suivi ne contient que des intitulés de chantier, jamais de données de deal (montants, noms de cédants, passifs).
- Français, aucun tiret cadratin. Note toute hypothèse retenue, en particulier le mapping dépôt vers projet tant qu'Oscar ne l'a pas confirmé.

---

## B. Câbler la routine dans Claude Code

Deux options, non exclusives.

1. Commande réutilisable. Enregistrer le prompt ci-dessus dans un fichier de commande Claude Code, par exemple `~/.claude/commands/maj-cc-suivi-ia.md` (commande globale) ou `.claude/commands/maj-cc-suivi-ia.md` dans un dépôt. Invocation manuelle : `/maj-cc-suivi-ia`. Utile pour un lancement à la demande et pour tester avant d'automatiser.

2. Run planifié sans surveillance. Planificateur de tâches Windows, une fois par jour vers 18h30 (juste avant la passe Cowork de 19h, pour que les candidats soient frais dans 00-inbox). Action : un `.bat` qui lance Claude Code en mode headless depuis un répertoire qui donne accès au vault et aux dépôts, par exemple :

   `claude -p "/maj-cc-suivi-ia" --permission-mode acceptEdits`

   Points de vigilance : le run headless doit avoir le droit d'écrire dans le vault (pré-accorder les permissions d'édition de fichiers), et le répertoire de travail doit couvrir le vault, les dépôts d'outils et C:\Users\OscarGRIMA\.claude\projects. Ne pas utiliser de contournement global de permissions ; borner aux écritures fichiers nécessaires.

Choix du créneau : la routine Claude Code produit à 18h30, la passe Cowork consomme à 19h le même jour. Si tu préfères, l'inverse marche aussi (CC après Cowork), la consommation se fera au run Cowork suivant.

---

## C. Modification à faire côté tâche Cowork (pour fermer la boucle)

Sans cette ligne, les dépôts de la routine risquent de ne pas être appliqués au board. Ajouter aux sources de la tâche Cowork « MAJ avancement suivi IA » :

« 3. Dépôts de la routine Claude Code : lire les fichiers 00-inbox/claude-code-avancement-*.md non encore consommés. Les traiter comme preuve d'avancement in-scope au même titre qu'une session (mêmes garde-fous : relever seulement au-dessus du live, anti-doublon contre les titres réels, jamais de baisse). Après application, marquer le fichier consommé (renommer en .consomme ou noter dans le CHANGELOG) pour ne pas le retraiter. »

Point ouvert à trancher par toi : le dépôt triactis-suivi-ia (l'appli board elle-même) compte-t-il comme un chantier à tracer au board, ou reste-t-il méta et hors périmètre. Et confirmer le mapping des trois autres dépôts vers P1/P2/P3.
