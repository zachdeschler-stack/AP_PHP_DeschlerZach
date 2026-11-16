# Synthèse de conception – Plateforme de QCM en ligne (E6)

Nov 16, 2026 · @Nolix

## 1. Contexte et objectifs

Le projet est une plateforme de QCM en ligne destinée à un établissement de formation : les formateurs y créent et configurent des QCM, les étudiants y répondent, et un administrateur gère le référentiel (comptes, classes, matières).

Le projet s'inscrit dans l'Atelier de professionnalisation 3 du BTS SIO option SLAM (Lycée René Cassin, Strasbourg). Il est mené en gestion de projet en cascade, avec quatre rendus : planification, environnement de travail, conception, application en ligne.

Objectifs de la plateforme :

- permettre aux formateurs de constituer une banque de questions et de générer des QCM à partir de tirages aléatoires ;
- permettre aux étudiants de consulter les QCM de leur classe, d'y répondre et de revoir leurs résultats ;
- donner à l'administrateur une vue d'ensemble et la gestion complète du référentiel.

## 2. Acteurs et diagramme de contexte

La plateforme est utilisée par trois acteurs : l'administrateur, le formateur et l'étudiant.

&#91;embedded content: diagramme de contexte · 3 acteurs, 1 système\]

L'administrateur gère tout le référentiel, le formateur s'occupe des questions, des QCM et des résultats, et l'étudiant répond aux QCM.

## 3. Analyse fonctionnelle

Le diagramme de cas d'utilisation détaille dix fonctions principales réparties entre les trois acteurs, plus trois fonctions associées.

| Acteur | Cas d'utilisation | Relation |
| --- | --- | --- |
| Administrateur | Créer et modifier les comptes utilisateur |  |
| Administrateur | Créer et modifier les classes (affecter matières et formateurs) |  |
| Administrateur | Gérer les matières |  |
| Administrateur | Gérer les mots-clés |  |
| Formateur | Gérer les questions | Dupliquer une question « extend » |
| Formateur | Configurer un QCM | Générer les questions du QCM par tirage aléatoire « include » |
| Formateur | Consulter les résultats des étudiants |  |
| Étudiant | Consulter la liste des QCM de sa classe | Filtrer les QCM |
| Étudiant | Répondre à un QCM |  |
| Étudiant | Consulter ses résultats |  |

Deux mécanismes méritent d'être soulignés. La configuration d'un QCM inclut toujours la génération des questions par tirage aléatoire (relation « include »). La duplication d'une question est une extension facultative de la gestion des questions (relation « extend »).

## 4. Modèle conceptuel de données

Le MCD, réalisé sous Looping, comporte dix entités. Chacune a un identifiant propre (préfixe `id`).

| Entité | Attributs | Rôle |
| --- | --- | --- |
| Utilisateur | idUtilisateur, nom, prénom, email, mdp, rôle | Comptes des trois profils ; le rôle détermine l'espace accessible |
| Classe | idClasse, libellé, année | Groupe d'étudiants |
| Matière | idMatière, libellé | Discipline enseignée |
| Enseignement | idEnseignement | À préciser (affectation des matières et des formateurs aux classes) |
| QCM | idQCM, titre, dateDébut, dateFin, nbQuestion | Questionnaire configuré par un formateur |
| Question | idQuestion, énoncé, correction, commentaire | Banque de questions |
| TypeQuestion | idType, libellé | Type de la question |
| RéponsePossible | idRéponse, libellé, estCorrecte, estFausse, ordreAttendu | Réponses proposées pour une question |
| MotClé | idMotClé, libellé | Mots-clés gérés par l'administrateur |
| Passage | idPassage, dateValidation, note | Tentative d'un étudiant sur un QCM |

Les associations du modèle portent les noms suivants : cibler, caractériser, composer, estDeType, porteSur, concerner, assurer, appartenir, proposer, concerner\_QCM, choisir, réaliser et pour.

*Points ouverts : exporte ton MCD de Looping en image (PNG) et ajoute-le ici, avec les cardinalités. Je n'ai pas pu les lire de façon fiable dans le fichier .loo, donc je ne décris pas quelle entité relie quelle autre. Dans le fichier, certains noms comportent des fautes de frappe (`estCorecte`, `idEnsgeignement`, `compeser`) : je les ai corrigés ici, pense à les corriger aussi dans Looping et dans la base.*

## 5. Maquettes

Les maquettes ont été réalisées sous draw.io. Elles couvrent l'authentification et le tableau de bord de l'administrateur.

**Page de connexion.** Elle comporte un champ « nom d'utilisateur », un champ « mot de passe » et un bouton de validation. En cas d'échec, la même page réaffiche le message « nom d'utilisateur ou mot de passe incorrect », sans préciser lequel des deux est faux.

**Tableau de bord administrateur.** Un menu latéral est associé à une page de synthèse qui affiche sept compteurs :

- nombre d'utilisateurs ;
- nombre de formateurs ;
- nombre d'étudiants ;
- nombre de classes ;
- nombre de matières ;
- nombre de questions disponibles ;
- nombre de QCM créés.

**Navigation.** Après la connexion, l'utilisateur est dirigé vers l'espace qui correspond à son rôle (administrateur, formateur ou étudiant).

*Point ouvert : les maquettes des espaces formateur et étudiant ne sont pas encore détaillées dans le fichier draw.io fourni. Ajoute-les ici, ou envoie-moi une capture, pour que je les décrive.*

## 6. Architecture applicative

L'application suit le modèle MVC en PHP, avec une base MariaDB.

- **Modèle** : accès à la base de données et règles métier, avec une classe par entité principale du MCD (utilisateur, classe, matière, question, QCM, passage).
- **Vue** : pages affichées à l'utilisateur, conformes aux maquettes, avec JavaScript pour les interactions.
- **Contrôleur** : reçoit la requête, vérifie le rôle de l'utilisateur, appelle le modèle puis choisit la vue à renvoyer.

*Point ouvert : ajoute l'arborescence réelle de ton projet (dossiers et fichiers principaux) et le nom des contrôleurs. Je pourrai alors détailler cette section et y ajouter un schéma.*

## 7. Sécurité

Le modèle de données prévoit un champ « rôle » sur chaque utilisateur et une authentification par adresse e-mail et mot de passe. Trois rôles (administrateur, formateur, étudiant) déterminent les fonctions accessibles.

Points à confirmer avant de les écrire comme acquis, selon ce que ton code fait réellement :

- [ ] mots de passe stockés hachés (par exemple avec `password_hash`) et non en clair ;
- [ ] requêtes SQL préparées (PDO) contre l'injection SQL ;
- [ ] échappement des données affichées contre le XSS ;
- [ ] contrôle du rôle sur chaque page ou action réservée ;
- [ ] gestion des sessions (déconnexion, régénération de l'identifiant) ;
- [ ] message d'erreur de connexion générique, comme sur la maquette.

## 8. Choix techniques

| Besoin | Choix | Justification |
| --- | --- | --- |
| Langage serveur | PHP, architecture MVC | Sépare l'accès aux données, la logique et l'affichage ; fait partie du programme SLAM |
| Base de données | MariaDB | SGBD relationnel adapté au schéma du MCD |
| Interface dynamique | JavaScript | Interactions côté navigateur (filtrage des QCM, saisie des réponses) |
| Modélisation | Looping (MCD), draw.io (contexte, cas d'utilisation, maquettes) | Outils utilisés pour les livrables de conception |
| Gestion de projet | Méthode en cascade, GitHub (milestones et tickets), diagramme de Gantt | Un rendu par phase, suivi des tâches sur GitHub |

*À compléter : serveur web utilisé (Apache, Nginx), bibliothèques ou framework éventuels, version de PHP.*

## 9. Conclusion

La conception couvre le contexte, les trois profils, treize fonctions, un modèle de données de dix entités et les maquettes de connexion et d'administration. Elle sert de base à l'application en ligne attendue au rendu 4 (4 janvier 2027), avant l'épreuve E6 blanche du 8 janvier 2027.

*À compléter : bilan personnel, difficultés rencontrées et choix qui ont évolué pendant le projet.*
