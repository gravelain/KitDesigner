# Roadmap détaillée — Devenir dev backend/frontend Python
### Projets directeurs : KitConfect → Application de réservation de bus

---

## Comment utiliser ce document

Chaque étape est découpée en **blocs**. Un bloc = une session de travail avec :
- 🎯 **Objectif** : ce qu'on cherche à obtenir concrètement
- 🛠️ **Ce qu'on fait** : les actions/étapes de code
- 📚 **Points d'apprentissage** : les notions à comprendre (pas juste "faire marcher")
- ⚠️ **À ne pas oublier / pièges fréquents**
- ✅ **Critère de validation** : comment on sait que le bloc est terminé avant de passer au suivant

Règle d'or : **on ne passe au bloc suivant que si le critère de validation est atteint ET que tu peux réexpliquer ce que fait le code avec tes mots.**

---

## Étape 0 — Fondations Python Web (avant tout code métier)

### Bloc 0.1 — Environnement de travail propre
- 🎯 Avoir un environnement Python isolé et reproductible.
- 🛠️
  - Créer un environnement virtuel (`python -m venv venv`)
  - Activer/désactiver l'environnement
  - Créer un `.gitignore` adapté Python (venv, `__pycache__`, `.env`, etc.)
  - Initialiser le dépôt Git + premier commit + repo GitHub distant
- 📚 Points d'apprentissage :
  - Pourquoi isoler les dépendances par projet (pas d'install globale qui pollue tout)
  - Différence `pip install` vs `pip freeze > requirements.txt`
  - Convention `.gitignore` pour un projet Python
- ⚠️ À ne pas oublier :
  - Ne **jamais** committer le dossier `venv/` ni les fichiers `.env` (secrets, mots de passe SMTP, clés API)
  - Vérifier que `git status` est propre avant de continuer
- ✅ Validation : tu peux cloner ton propre repo dans un nouveau dossier, recréer le venv, réinstaller les dépendances via `requirements.txt` et ça fonctionne.

### Bloc 0.2 — Bases HTTP / API (rappel utile même si tu connais Symfony)
- 🎯 Être au clair avec le vocabulaire HTTP avant de coder une API.
- 🛠️ Revoir : méthodes HTTP (GET/POST/PUT/PATCH/DELETE), codes de statut (200, 201, 400, 404, 422, 500), structure d'une requête/réponse JSON.
- 📚 Points d'apprentissage :
  - Différence API REST "propre" vs endpoints RPC-like
  - Rôle des headers (`Content-Type`, `Authorization`)
- ⚠️ À ne pas oublier : un mauvais code de retour (ex: renvoyer 200 sur une erreur) casse la lisibilité de l'API pour un futur consommateur (React, Postman, autre service).
- ✅ Validation : tu sais dire quel code HTTP renvoyer pour "ressource créée", "donnée invalide", "ressource introuvable".

### Bloc 0.3 — Premiers pas FastAPI
- 🎯 Faire tourner une API minimale et comprendre Pydantic.
- 🛠️
  - Installer FastAPI + Uvicorn
  - Créer une route `GET /health` qui renvoie un statut
  - Créer un modèle Pydantic simple et une route `POST` qui le valide
  - Explorer la doc auto-générée (`/docs`)
- 📚 Points d'apprentissage :
  - Rôle de Pydantic (validation + sérialisation automatique)
  - Différence *path parameter* / *query parameter* / *body*
  - Ce qu'apporte le typage Python ici (tu connais déjà ce confort via TypeScript/Angular)
- ⚠️ À ne pas oublier : FastAPI génère la doc Swagger automatiquement à partir de tes types — ne la néglige pas, elle te sert de test manuel.
- ✅ Validation : tu peux créer une nouvelle route de zéro en moins de 10 minutes sans copier-coller un tuto.

---

## Étape 1 — Projet "KitConfect" (FastAPI + React)

### Bloc 1.1 — Modélisation des données
- 🎯 Définir la structure de la commande joueur.
- 🛠️ Modèle Pydantic (`nom`, `dossard`, `numero`, `taille`, `email`, `telephone`) + choix de la base de données (SQLite pour démarrer).
- 📚 Points d'apprentissage :
  - Différence schéma Pydantic (validation entrée/sortie API) vs modèle de base de données (persistance)
  - Introduction à un ORM léger (SQLAlchemy) ou SQLModel (fait par l'auteur de FastAPI, plus simple pour débuter)
- ⚠️ À ne pas oublier :
  - Valider le format email et téléphone côté serveur (jamais faire confiance au frontend)
  - Prévoir une contrainte de taille cohérente (enum : XS/S/M/L/XL...)
- ✅ Validation : tu peux dessiner (sur papier ou en dictant) la structure de la table `commandes` sans regarder ton code.

### Bloc 1.2 — Endpoint d'inscription + persistance
- 🎯 Recevoir une inscription et la stocker en base.
- 🛠️ Route `POST /commandes`, connexion à la base, insertion, retour de l'objet créé avec un identifiant.
- 📚 Points d'apprentissage :
  - Cycle de vie d'une requête : validation → traitement métier → persistance → réponse
  - Gestion des erreurs (que se passe-t-il si l'email est déjà utilisé ?)
- ⚠️ À ne pas oublier : gérer les erreurs de validation avec des messages clairs (status 422 avec détail), pas juste un crash.
- ✅ Validation : test réussi dans Postman/Insomnia avec un cas valide ET un cas invalide (champ manquant, email mal formé).

### Bloc 1.3 — Envoi de l'email de confirmation
- 🎯 Envoyer un récapitulatif de commande par email après inscription.
- 🛠️ Intégration SMTP (Mailtrap ou équivalent en dev), template simple du mail.
- 📚 Points d'apprentissage :
  - Ne jamais bloquer la réponse HTTP sur l'envoi d'email si possible (notion d'opération asynchrone/tâche de fond — FastAPI propose `BackgroundTasks`)
  - Gestion des secrets (identifiants SMTP dans `.env`, jamais en dur dans le code)
- ⚠️ À ne pas oublier : que se passe-t-il si l'envoi d'email échoue ? La commande doit rester enregistrée quand même (ne pas faire dépendre la réussite de l'inscription de l'envoi du mail).
- ✅ Validation : tu reçois bien le mail récapitulatif en environnement de test, et une commande reste enregistrée même si tu coupes volontairement l'envoi de mail pour tester.

### Bloc 1.4 — Frontend React : le formulaire
- 🎯 Formulaire connecté à l'API pour l'inscription.
- 🛠️ Setup React (Vite recommandé), formulaire contrôlé, appel API (fetch/axios), affichage de confirmation ou des erreurs.
- 📚 Points d'apprentissage :
  - Gestion d'état d'un formulaire en React (tu as déjà les réflexes Angular, il faut juste "traduire")
  - Gestion des erreurs API côté frontend (afficher les messages de validation renvoyés par FastAPI)
  - Notion de CORS (pourquoi le navigateur bloque parfois l'appel entre React (port 5173) et FastAPI (port 8000))
- ⚠️ À ne pas oublier : configurer CORS côté FastAPI dès le début pour éviter des heures de debug inutiles.
- ✅ Validation : un utilisateur peut remplir le formulaire, avoir un retour d'erreur clair si besoin, et voir une confirmation à la fin, avec réception du mail.

### Bloc 1.5 — Tests manuels structurés
- 🎯 Vérifier l'ensemble du parcours avant de packager.
- 🛠️ Collection Postman/Insomnia avec tous les cas (succès, erreurs de validation, doublon éventuel).
- 📚 Points d'apprentissage : intérêt de garder une collection de tests réutilisable (base pour automatiser plus tard).
- ✅ Validation : la collection couvre au moins 5 scénarios différents et tous passent comme attendu.

---

## Étape 2 — Introduction à l'infra (Docker + Jenkins) sur KitConfect

### Bloc 2.1 — Dockerisation du backend
- 🎯 Faire tourner l'API FastAPI dans un conteneur.
- 🛠️ Écrire un `Dockerfile`, builder l'image, lancer le conteneur.
- 📚 Points d'apprentissage : différence image/conteneur, notion de couches Docker, `.dockerignore`.
- ⚠️ À ne pas oublier : ne pas copier `venv/` dans l'image (inutile et lourd).
- ✅ Validation : l'API répond correctement via `curl` alors qu'elle tourne uniquement dans le conteneur (pas en local).

### Bloc 2.2 — Docker Compose (API + base de données)
- 🎯 Orchestrer plusieurs services ensemble.
- 🛠️ `docker-compose.yml` avec le service API et un service PostgreSQL (bon moment pour migrer de SQLite à PostgreSQL).
- 📚 Points d'apprentissage : réseau interne Docker, variables d'environnement partagées, volumes pour la persistance des données.
- ⚠️ À ne pas oublier : les identifiants de connexion à la base ne doivent pas être en dur, utiliser un fichier `.env`.
- ✅ Validation : `docker compose up` démarre tout le projet en une seule commande, données persistées après redémarrage.

### Bloc 2.3 — Premier pipeline Jenkins
- 🎯 Automatiser un minimum : build + tests à chaque push.
- 🛠️ Installer Jenkins (en conteneur Docker, pratique pour ta machine), créer un job simple relié à ton repo GitHub, définir un `Jenkinsfile` basique (checkout → install deps → run tests).
- 📚 Points d'apprentissage : notion de pipeline CI, déclenchement automatique (webhook GitHub → Jenkins).
- ⚠️ À ne pas oublier : commencer simple (juste lancer les tests), on complexifiera au projet 2.
- ✅ Validation : un push sur GitHub déclenche automatiquement un build visible dans Jenkins.

---

## Étape 3 — Projet "Réservation de bus" (Django + DRF) — le projet vitrine

⚠️ Ici on change de logique : ce projet sera **valorisé en entretien**, donc l'architecture compte autant que le fonctionnel.

### Bloc 3.1 — Cadrage fonctionnel avant le code
- 🎯 Clarifier le périmètre avant d'ouvrir VS Code.
- 🛠️ Lister les entités (utilisateur, trajet, bus, réservation, paiement éventuel), les rôles (client, admin), les règles métier (ex : pas de surbooking).
- 📚 Points d'apprentissage : l'intérêt de modéliser sur papier avant de coder évite de tout refaire à mi-parcours.
- ⚠️ À ne pas oublier : définir le MVP (version minimale) clairement pour ne pas se disperser.
- ✅ Validation : tu as un schéma (même à la main) des entités et de leurs relations.

### Bloc 3.2 — Setup Django + structuration en apps
- 🎯 Poser une architecture propre dès le départ.
- 🛠️ Projet Django, découpage en apps (`users`, `trips`, `reservations`, ...), configuration des settings par environnement (dev/prod).
- 📚 Points d'apprentissage :
  - Philosophie Django "batteries included" vs FastAPI minimaliste
  - Bonnes pratiques de découpage en apps (une app = un domaine métier cohérent)
- ⚠️ À ne pas oublier : séparer la config sensible (`SECRET_KEY`, base de données) via variables d'environnement dès le premier commit.
- ✅ Validation : le projet démarre, structure des apps cohérente et explicable.

### Bloc 3.3 — Modèles Django et migrations
- 🎯 Construire le modèle de données réel.
- 🛠️ Modèles (Trajet, Bus, Réservation, Utilisateur si custom), relations (ForeignKey, ManyToMany si sièges), migrations.
- 📚 Points d'apprentissage : ORM Django, gestion des migrations (jamais modifier une migration déjà appliquée en prod), relations entre modèles.
- ⚠️ À ne pas oublier : réfléchir aux contraintes d'intégrité (un bus ne peut pas avoir plus de réservations que de sièges).
- ✅ Validation : tu peux créer des données de test via l'admin Django et via le shell, les relations sont correctes.

### Bloc 3.4 — API avec Django REST Framework
- 🎯 Exposer les données via une API propre.
- 🛠️ Serializers, ViewSets/Views, routing DRF, pagination, filtres (trajets par date/ville).
- 📚 Points d'apprentissage :
  - Différence Serializer (DRF) vs Schéma Pydantic (FastAPI) — même rôle, philosophie différente
  - Permissions et authentification (JWT recommandé pour une API consommée par React)
- ⚠️ À ne pas oublier : ne pas exposer de données sensibles dans les serializers (mots de passe, infos internes).
- ✅ Validation : Swagger/OpenAPI de DRF généré et fonctionnel, tests via Postman couvrant les cas d'usage principaux.

### Bloc 3.5 — Logique métier des réservations
- 🎯 Implémenter les règles concrètes (dispo des sièges, annulation, etc.).
- 🛠️ Logique de vérification de disponibilité, gestion des statuts de réservation.
- 📚 Points d'apprentissage : où placer la logique métier (pas dans les vues, plutôt dans des services/managers dédiés) — bonne pratique d'architecture.
- ⚠️ À ne pas oublier : gérer les cas de concurrence (deux personnes réservent le même siège en même temps).
- ✅ Validation : scénarios de test couvrant les cas limites (dernier siège, double réservation).

### Bloc 3.6 — Frontend React connecté à l'API DRF
- 🎯 Interface de recherche/réservation de trajet.
- 🛠️ Pages de recherche, liste de résultats, formulaire de réservation, gestion de session utilisateur (token JWT stocké côté client).
- 📚 Points d'apprentissage : gestion de l'authentification côté frontend (stockage sécurisé du token, appels authentifiés).
- ⚠️ À ne pas oublier : ne jamais stocker de token sensible de façon négligente (attention aux failles XSS si stockage en `localStorage`).
- ✅ Validation : parcours complet fonctionnel de bout en bout (recherche → réservation → confirmation).

### Bloc 3.7 — Tests automatisés
- 🎯 Sécuriser le projet avec des tests.
- 🛠️ Tests unitaires Django (modèles, logique métier), tests d'API DRF (pytest-django ou APITestCase).
- 📚 Points d'apprentissage : pyramide de tests, intérêt de tester la logique métier isolément.
- ✅ Validation : suite de tests qui passe en local et qu'on pourra brancher sur Jenkins.

---

## Étape 4 — Mise en production du projet bus

### Bloc 4.1 — Dockerisation complète
- 🎯 Conteneuriser backend, frontend et base de données.
- 🛠️ `Dockerfile` pour Django (avec Gunicorn/Uvicorn selon config), `Dockerfile` pour le build React (servi via Nginx), `docker-compose.yml` global.
- 📚 Points d'apprentissage : différence serveur de dev vs serveur de prod (jamais `runserver` Django en prod).
- ⚠️ À ne pas oublier : `DEBUG=False` en prod, gestion des fichiers statiques Django (`collectstatic`).
- ✅ Validation : l'application complète tourne via `docker compose up` en mode "quasi-prod".

### Bloc 4.2 — Pipeline CI/CD complet avec Jenkins
- 🎯 Automatiser tests → build → déploiement.
- 🛠️ `Jenkinsfile` avec étapes : lint, tests, build des images Docker, push vers un registre, déploiement.
- 📚 Points d'apprentissage : notion de pipeline multi-étapes, gestion des secrets dans Jenkins (credentials), stratégie de déploiement (manuel validé vs automatique).
- ⚠️ À ne pas oublier : ne jamais mettre de secrets en clair dans le `Jenkinsfile`.
- ✅ Validation : un push sur la branche principale déclenche automatiquement tests + build, déploiement au moins semi-automatique.

### Bloc 4.3 — Hébergement et mise en ligne
- 🎯 Rendre l'application accessible publiquement.
- 🛠️ Choix d'un hébergement (VPS ou service cloud), configuration domaine/sous-domaine, HTTPS (Let's Encrypt), variables d'environnement de prod.
- 📚 Points d'apprentissage : bases de la sécurisation d'un serveur exposé (pare-feu, mises à jour, accès SSH sécurisé).
- ⚠️ À ne pas oublier : sauvegardes de la base de données en production.
- ✅ Validation : l'application est accessible via une URL publique, en HTTPS, fonctionnelle de bout en bout.

### Bloc 4.4 — Monitoring minimal
- 🎯 Savoir si quelque chose casse en prod.
- 🛠️ Logs applicatifs centralisés, alerte basique en cas d'erreur serveur.
- ✅ Validation : tu peux retrouver une erreur survenue en prod dans les logs sans deviner.

---

## Étape 5 — Valorisation pour la recherche d'emploi

### Bloc 5.1 — Documentation GitHub
- 🎯 Un repo qui donne envie de te recruter.
- 🛠️ README clair (contexte, stack, architecture, captures d'écran, instructions de lancement), schéma d'architecture simple.
- ✅ Validation : une personne qui ne te connaît pas comprend le projet en 2 minutes de lecture.

### Bloc 5.2 — Discours d'entretien
- 🎯 Pouvoir présenter le projet avec assurance.
- 🛠️ Préparer une explication orale de 2-3 minutes : problème résolu, choix techniques et pourquoi, difficultés rencontrées et comment tu les as surmontées.
- ✅ Validation : tu peux présenter le projet sans notes.

---

## Repères transverses à garder en tête sur tout le parcours

- **Sécurité** : jamais de secret en dur dans le code, valider systématiquement les entrées utilisateur côté serveur.
- **Git** : commits réguliers et lisibles, une branche par fonctionnalité si possible.
- **Progression** : mieux vaut un bloc bien compris qu'trois blocs bâclés — le but est la maîtrise, pas la vitesse.
- **FastAPI vs Django** : garder en tête que ce sont deux philosophies (minimaliste vs "batteries included") — comprendre les deux te rend plus fort qu'un profil mono-techno.
- **React** : profite de ta expérience Angular pour aller plus vite sur les concepts (composants, state, routing), la syntaxe change mais les concepts sont proches.

---

*Document évolutif — à ajuster au fil de l'avancement du projet.*
