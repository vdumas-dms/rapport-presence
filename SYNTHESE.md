# Synthèse — Projet Rapport de Présence

## Contexte

Outil de téléphonie utilisé comme badge digital chez YouSchool. L'export CSV (`AGENTS_STATE_PER_DAY.csv`) contient les temps de connexion, disponibilité, travail, pause et absence par collaborateur et par journée.

## Ce qui a été construit

Application web monopage (`index.html`) — sans backend, entièrement dans le navigateur — permettant d'analyser automatiquement le fichier CSV et de générer un rapport texte copiable.

### Fonctionnement

1. L'utilisateur glisse-dépose (ou sélectionne) le fichier CSV exporté depuis l'outil téléphonie.
2. L'app parse le CSV (délimiteur `;`, gestion du BOM Excel).
3. Elle compare les temps réels aux horaires configurés et génère un rapport texte aligné, prêt à copier-coller.

### Paramétrage des horaires (panneau de configuration)

| Jour | Début | Fin | Durée | Pause |
|------|-------|-----|-------|-------|
| Lundi | 09:00 | 18:00 | 9h00 | 80 min |
| Mardi | 09:00 | 17:00 | 8h00 | 80 min |
| Mercredi | 08:30 | 14:00 | 5h30 | 20 min |
| Jeudi | 09:00 | 18:00 | 9h00 | 80 min |
| Vendredi | 08:30 | 16:00 | 7h30 | 80 min |

Tous les champs sont éditables directement dans l'interface (heures de début/fin + durée de pause). La durée nette se recalcule en temps réel.

### Rapport généré

Le rapport texte produit par jour :
- Un tableau par collaborateur avec : connexion réelle vs attendue, écart, pause réelle vs autorisée, écart pause
- Un indicateur `[!]` si l'écart de connexion dépasse 30 min ou si la pause est dépassée de plus de 5 min
- Une section de synthèse listant uniquement les anomalies

### Cas gérés

- **Jours fériés / non travaillés** : si aucune donnée n'existe pour un jour, ce jour est ignoré
- **Journée partielle** : si la connexion moyenne du groupe est < 40 % du temps attendu, le jour est signalé comme partiel et exclu de l'analyse (évite les faux positifs en fin de journée)
- **Lignes "Moyenne" et "Total"** du CSV : filtrées automatiquement (pas d'`@` dans le nom)

## Décisions techniques

| Sujet | Choix |
|-------|-------|
| Architecture | HTML/JS monopage, aucun serveur |
| Déploiement | GitHub Pages — `https://vdumas-dms.github.io/rapport-presence/` |
| Parsing CSV | Natif JS, split `;`, gestion BOM `0xFEFF` |
| Copie rapport | `navigator.clipboard.writeText()` + fallback `execCommand` |
| Horaires | Configurables via `<input type="time">` et `<input type="number">` |
| Format noms | Email `prenom.nom@youschool.fr` → `Prénom NOM` (majuscules, tirets conservés) |

## Correction apportée en cours d'échange

Le lundi avait été initialement saisi avec une fin à 17h (480 min). Corrigé à 18h (540 min) avant la création de l'app — les valeurs par défaut dans le code sont donc déjà correctes.

## Déploiement GitHub Pages

L'app est hébergée sur GitHub Pages à l'adresse :
**`https://vdumas-dms.github.io/rapport-presence/`**

### Mise en place du dépôt git

Le dépôt git local a été initialisé directement dans le dossier `rapport-presence/` (pas à la racine de l'espace de travail, sinon VS Code y inclut tous les fichiers parents).

Commandes utilisées :
```bash
cd rapport-presence
git init
git remote add origin https://github.com/vdumas-dms/rapport-presence.git
git add index.html SYNTHESE.md
git commit -m "feat: ..."
git push -u origin main
```

### Authentification automatique

Git utilise le gestionnaire de credentials macOS (keychain) — aucun token à saisir manuellement. Lors du premier `git push`, VS Code a ouvert une fenêtre d'authentification GitHub avec le compte `vdumas-dms`. Les pushes suivants se font sans aucune interaction.

### Pousser une mise à jour

Depuis le terminal dans le dossier `rapport-presence/` :
```bash
git add index.html
git commit -m "description de la modification"
git push
```

GitHub Pages se met à jour automatiquement en 1–2 minutes.

### Points de vigilance

- Initialiser le dépôt git **dans** `rapport-presence/`, jamais dans le dossier parent (risque d'inclure des PDFs confidentiels et données personnelles dans un repo public)
- Le fichier `.gitignore` à la racine du dossier exclut les fichiers indésirables

---

## Évolutions apportées

### Synthèse visuelle (section au-dessus du rapport texte)

Trois sections avec barres de progression colorées, triées par ordre décroissant d'anomalie :
- **Dépassements de pause** (orange) — cumulé par collaborateur sur la semaine
- **Surplus de connexion** (vert) — temps travaillé au-delà du prévu
- **Déficit de connexion** (rouge) — temps manquant à rattraper

### Flèches déroulantes par collaborateur

Chaque ligne de la synthèse dispose d'un bouton **▶** qui ouvre un panneau détaillé :
- Jours concernés (nom + date)
- Valeur réelle vs valeur attendue
- Écart coloré (+/-)

### Formatage des noms

La fonction `emailToName()` transforme l'email en nom affiché :
- `prenom.nom@youschool.fr` → `Prénom NOM`
- `jean-pierre.dupont-martin@youschool.fr` → `Jean-Pierre DUPONT-MARTIN`
- Règle : premier segment capitalisé (prénom), segments suivants en majuscules (nom)

## Incident : fichier corrompu (2026-05-29)

Le fichier `index.html` s'est retrouvé tronqué — seule la fin du JavaScript était présente, tout le HTML/CSS avait disparu. Cause probable : une synchronisation OneDrive ou une édition accidentelle dans l'IDE.

Le fichier a été entièrement reconstruit à partir du contexte de session Claude Code, puis poussé sur GitHub. La page GitHub Pages a été restaurée sans perte de fonctionnalité.

**Bonne pratique :** pousser sur GitHub régulièrement après chaque modification — le dépôt sert de sauvegarde fiable contre ce type de corruption.

---

## Fichiers

| Fichier | Rôle |
|---------|------|
| `index.html` | Application complète (HTML + CSS + JS inline) |
| `SYNTHESE.md` | Ce fichier — documentation du projet |
| `.gitignore` | Exclut PDFs, CSV et fichiers système du dépôt git |
| `AGENTS_STATE_PER_DAY.csv` | Fichier source d'exemple (dans le dossier parent) |
