# IMPORTANT : ceci est une V1.0 du CdC, il est voué à évoluer dans les prochains jours. Celui-ci a été produit après un long après-midi de brainstorming et a # # comme premier rôle d'être une solide base, et fait état de notre vision du projet au début de ce dernier. La ROADMAP et d'autres sections seront sujettes à # # évolution au fur et à mesure que notre compréhension et vision du projet évoluent.

# Cahier des Charges Technique - RobLaude

**Version:** 1.1
**Date:** 2026-02-19
**Équipe:** Maxime Theophilos, Wissem Karboub
**Projet:** HETIC Web3 - Robotique

---

## 1. Vision

**RobLaude** est un robot d'assistance autonome qui navigue, saisit et transporte des objets (documents, livres) entre points d'un ERP (Établissement Recevant du Public), contrôlé via une interface web accessible, pour améliorer l'autonomie des personnes à mobilité réduite.

---

## 2. Objectifs et Périmètre

### Objectifs

1. Naviguer de manière autonome dans un espace intérieur en évitant les obstacles
2. Détecter, saisir et manipuler des objets avec le bras robotique (livres, documents, petits colis)
3. Transporter des objets entre deux points désignés
4. Fournir une interface web accessible (WCAG AA) pour le contrôle et le monitoring en temps réel
5. Communiquer l'état du robot via WebSocket (position, statut mission, alertes)

### Non-objectifs (MVP)

1. Pas de gestion multi-robots (fleet management)
2. Pas de reconnaissance faciale ou vocale
3. Pas de fonctionnement en extérieur
4. Pas de manipulation d'objets complexes ou lourds (>500g)

### Personas

**Marie**, 45 ans, agente administrative en fauteuil roulant
Doit régulièrement faire transporter des dossiers entre services, actuellement dépendante de collègues. Elle souhaite pouvoir envoyer une demande de transport depuis son poste et suivre l'avancement sans solliciter d'aide.

**Thomas**, 28 ans, responsable technique ERP
Veut monitorer les missions du robot, consulter l'historique des livraisons, et intervenir rapidement en cas de problème. Il a besoin d'une vue d'ensemble claire et d'alertes en temps réel.

---

## 3. Use Cases

### Tableau récapitulatif

| ID | Nom | Acteur | But |
|----|-----|--------|-----|
| UC-01 | Demander un transport | Utilisateur | Envoyer le robot chercher/livrer un document |
| UC-02 | Récupérer un objet | Utilisateur | Demander au robot de saisir un objet à un emplacement |
| UC-03 | Suivre la mission | Utilisateur | Voir la position et l'état du robot en temps réel |
| UC-04 | Arrêt d'urgence | Utilisateur / Système | Stopper immédiatement le robot |
| UC-05 | Configurer les points | Admin | Définir les points de collecte/livraison |
| UC-06 | Configurer les objets | Admin | Définir les objets saisissables et leurs positions |
| UC-07 | Consulter l'historique | Admin | Voir les missions passées et statistiques |

### UC-01 : Demander un transport (détaillé)

**Acteurs:** Utilisateur (via interface web accessible)

**Préconditions:**
- Robot connecté et en attente
- Points de collecte/livraison configurés

**Scénario nominal:**
1. L'utilisateur ouvre le dashboard et sélectionne "Nouvelle mission"
2. L'utilisateur choisit le point de départ (ex: "Accueil")
3. L'utilisateur choisit le point d'arrivée (ex: "Bureau 204")
4. L'utilisateur confirme la mission
5. Le système envoie l'ordre au robot
6. Le robot confirme la prise en charge
7. L'interface affiche "Mission en cours" avec position temps réel

**Extensions:**
- 1a. Robot déjà en mission → afficher "Robot occupé", proposer mise en file d'attente
- 6a. Robot ne répond pas → alerte "Robot injoignable", proposer retry

**Postconditions:** Mission créée, robot en déplacement, utilisateur peut suivre

---

### UC-02 : Récupérer un objet (détaillé)

**Acteurs:** Utilisateur (via interface web accessible)

**Préconditions:**
- Robot connecté et en attente
- Objet enregistré dans le système (position, type)
- Bras robotique calibré

**Scénario nominal:**
1. L'utilisateur ouvre le dashboard et sélectionne "Récupérer un objet"
2. L'utilisateur sélectionne l'objet souhaité (ex: "Livre - 1984")
3. L'utilisateur choisit le point de livraison (ex: "Table 3")
4. L'utilisateur confirme la mission
5. Le robot navigue vers l'emplacement de l'objet
6. Le robot détecte l'objet via caméra RealSense
7. Le robot saisit l'objet avec le bras robotique
8. Le robot confirme la saisie réussie
9. Le robot navigue vers le point de livraison
10. Le robot dépose l'objet
11. L'interface affiche "Livraison effectuée"

**Extensions:**
- 6a. Objet non détecté → alerte "Objet introuvable", proposer retry ou annulation
- 7a. Saisie échouée → 3 tentatives max, puis alerte et annulation
- 8a. Objet tombé pendant transport → alerte "Objet perdu", arrêt mission

**Postconditions:** Objet livré au point demandé, mission loguée

---

### UC-04 : Arrêt d'urgence (détaillé)

**Acteurs:** Utilisateur ou Système (détection obstacle)

**Préconditions:** Robot en mouvement

**Scénario nominal:**
1. L'utilisateur clique sur le bouton "STOP" (accessible via raccourci clavier)
2. Le système envoie l'ordre d'arrêt immédiat
3. Le robot s'immobilise
4. L'interface affiche "Robot arrêté - intervention requise"

**Extensions:**
- 1a. Déclenché automatiquement par obstacle imprévu → même flux
- 3a. Robot ne s'arrête pas → alerte critique, instruction de déconnexion manuelle

**Postconditions:** Robot immobile, mission suspendue, log enregistré

---

## 4. Architecture Technique

### Vue d'ensemble

```
┌─────────────────────────────────────────────────────────────────┐
│                         UTILISATEUR                              │
│                    (navigateur web accessible)                   │
└─────────────────────────┬───────────────────────────────────────┘
                          │ HTTPS + WebSocket
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                      SERVEUR NODE.JS                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │ API REST    │  │ WebSocket   │  │ Robot Adapter           │  │
│  │ (missions,  │  │ (temps réel │  │ (abstraction comm.      │  │
│  │  config)    │  │  position)  │  │  avec robot)            │  │
│  └─────────────┘  └─────────────┘  └───────────┬─────────────┘  │
│                                                 │                │
│  ┌─────────────────────────────────────────────┴──────────────┐ │
│  │ Base de données MySQL                                      │ │
│  │ - Missions, Points, Historique, Logs                       │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────┬───────────────────────────────────────┘
                          │ MQTT ou Serial (USB)
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                    ROBOT (Yahboom Transbot)                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │ Jetson Nano │  │ ROS Nodes   │  │ Hardware                │  │
│  │ (compute)   │  │ (nav, lidar │  │ (moteurs, lidar,        │  │
│  │             │  │  control)   │  │  caméra RealSense)      │  │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Flux de données

| Flux | Protocole | Données |
|------|-----------|---------|
| UI → Serveur | REST + WebSocket | Commandes missions, config |
| Serveur → UI | WebSocket | Position robot, statut, alertes |
| Serveur → Robot | MQTT (ou Serial USB) | Ordres navigation, stop |
| Robot → Serveur | MQTT (ou Serial USB) | Position, capteurs, état |

### Pourquoi cette architecture

- **Découplage total** : Le web fonctionne sans robot (mock pour dev/tests)
- **Testabilité** : Chaque couche testable indépendamment
- **Compétences équipe** : Node.js + React = zone de confort
- **Flexibilité** : Changer le protocole robot sans toucher au front

---

## 5. Diagrammes UML

Les diagrammes sont disponibles dans le dossier `uml/` au format PlantUML :

- [Diagramme de cas d'utilisation](../uml/use-case-diagram.puml)
- [Diagramme de séquence - Mission](../uml/sequence-mission.puml)
- [Diagramme d'états - Mission](../uml/state-mission.puml)

---

## 6. Stack Technique

### Technologies retenues

| Composant | Technologie | Justification |
|-----------|-------------|---------------|
| **Frontend** | React + TypeScript | Expertise équipe, typage fort, large écosystème |
| **UI Components** | Radix UI ou React Aria | Accessibilité WCAG AA native, pas de styles imposés |
| **Styling** | Tailwind CSS | Rapide, responsive, bon support accessibilité |
| **État client** | Zustand ou React Query | Léger, simple, adapté au temps réel |
| **Backend** | Node.js + Express | Event-driven, bon pour WebSocket, expertise équipe |
| **WebSocket** | ws | Temps réel pour position robot |
| **Base de données** | MySQL | Robuste, maîtrisé par l'équipe, bon tooling |
| **ORM** | Prisma | DX moderne, migrations faciles, typage TypeScript |
| **Robot comm** | MQTT (Mosquitto) | Standard IoT, léger, pub/sub adapté aux capteurs |
| **Robot OS** | ROS2 Humble (Ubuntu 22.04) | Standard industrie récent, meilleur support |
| **Simulation** | Gazebo | Simulateur ROS, permet de dev sans hardware |
| **Navigation** | Nav2 + SLAM Toolbox | Solution éprouvée pour navigation autonome ROS2 |
| **Manipulation** | MoveIt2 | Framework standard pour contrôle de bras robotique |
| **Vision** | OpenCV + RealSense SDK | Détection d'objets, calcul de position 3D |
| **Détection objets** | YOLOv8 (optionnel) | Détection ML si besoin de reconnaissance avancée |

### Alternatives écartées

| Technologie | Raison d'exclusion |
|-------------|-------------------|
| SQLite | Moins adapté aux accès concurrents, préférence équipe MySQL |
| Socket.io | Plus lourd que `ws`, fonctionnalités extra non nécessaires |
| Next.js | SSR non nécessaire pour un dashboard, complexité inutile |
| rosbridge | Couple trop fort web↔ROS, complique les tests sans robot |
| Vue.js | Équipe plus expérimentée en React |

---

## 7. Risques et Contraintes

### Tableau des risques

| Risque | Probabilité | Impact | Mitigation |
|--------|-------------|--------|------------|
| Courbe d'apprentissage ROS2 trop longue | **Haute** | Fort | Commencer par tutoriels Gazebo dès S1, prévoir 3-4 semaines de formation avant hardware |
| Hardware livré en retard | Moyenne | Fort | Architecture découplée permet d'avancer sur web + simulation sans robot |
| Latence MQTT trop élevée | Faible | Moyen | Tests de latence dès réception hardware, fallback sur Serial USB si besoin |
| Navigation imprécise en environnement réel | **Haute** | Moyen | Calibration SLAM progressive, commencer avec parcours simples, agrandir zone |
| Manipulation bras robotique complexe | **Haute** | **Fort** | POC manipulation dès réception hardware, commencer avec objets simples, calibration hand-eye |
| Détection objets peu fiable | Moyenne | Fort | Utiliser marqueurs/QR codes en fallback si vision pure échoue |
| Saisie d'objets échoue fréquemment | Moyenne | Fort | Design poses de saisie prédéfinies, limiter variété d'objets au début |
| Intégration ROS2 ↔ Node.js complexe | Moyenne | Moyen | Utiliser bibliothèque MQTT standard, POC d'intégration S5-S6 |
| Accessibilité WCAG insuffisante | Faible | Fort | Audit accessibilité intégré dès le début (axe-core, tests manuels clavier) |
| Un membre de l'équipe absent/bloqué | Moyenne | Fort | Cross-training, documentation continue, code reviews régulières |
| Professeurs doutent de la compréhension | Moyenne | **Critique** | Sessions d'explication mutuelle, documenter les apprentissages, éviter code copié non compris |

### Contraintes projet

| Contrainte | Description |
|------------|-------------|
| Délai | ~4-5 mois (fin juin/juillet 2025) |
| Équipe | 2 développeurs (Maxime Theophilos, Wissem Karboub) |
| Hardware | Yahboom Transbot (Jetson Nano + Lidar + Bras + RealSense) - arrivée ~S3 |
| Budget | Matériel fourni par l'école |
| Compétences | Expertise web, débutants en ROS/robotique |
| Évaluation | Professeurs vérifieront la compréhension du code, pas seulement le résultat |

---

## 8. Conventions Équipe

### Git workflow

```
main (prod)   ← code stable, déployé en production
  └── dev     ← intégration, déploie auto sur pré-prod
        ├── feature/ROB-xx-description
        ├── fix/ROB-xx-description
        └── docs/ROB-xx-description
```

### Règles Git

- Branches nommées : `type/ROB-xx-description-courte`
- Commits : [Conventional Commits](https://www.conventionalcommits.org/)
  - `feat:` nouvelle fonctionnalité
  - `fix:` correction de bug
  - `docs:` documentation
  - `refactor:` refactoring sans changement de comportement
  - `test:` ajout/modification de tests
- **AUCUNE PR sans review de l'autre** - non négociable
- Merge sur `dev` → déploiement automatique pré-prod
- **RIEN sur `main` (prod)** tant que non testé et validé end-to-end par les deux

### Ticketing

- **Une feature = un ticket**
- Outil : GitHub Issues (ou selon préférence)
- Chaque ticket a un ID unique (ROB-xx)

### Communication

- **On ne commence JAMAIS un ticket sans que l'autre soit au courant**
- Serveur Discord dédié :
  - Un salon par ticket en cours
  - Liens vers documentation utile
  - Canal général pour discussions rapides
- Daily async (message le matin)

### Organisation

- **Réunion IRL chaque week-end**
  - Brainstorm sur la semaine à venir
  - Revue de ce qui a été fait
  - Peer-coding si nécessaire
- Documentation des décisions dans `docs/decisions/`

### Nommage code

| Élément | Convention | Exemple |
|---------|------------|---------|
| Composants React | PascalCase | `MissionCard.tsx` |
| Fichiers utilitaires | camelCase | `mqttClient.ts` |
| Variables/fonctions | camelCase | `startMission()` |
| Constantes | UPPER_SNAKE | `MAX_RETRY_COUNT` |
| Tables MySQL | snake_case | `mission_logs` |
| Topics MQTT | kebab-case | `robot/position-update` |

### Code review checklist

- [ ] Le code compile sans erreur
- [ ] Tests passent
- [ ] Pas de `console.log` oubliés
- [ ] Accessibilité vérifiée (si composant UI)
- [ ] Le reviewer **comprend** le code (sinon → session d'explication)

---

## 9. Roadmap et Questions Ouvertes

### Planning macro (18 semaines ~ mi-février → fin juin)

| Phase | Semaines | Objectif | Livrables |
|-------|----------|----------|-----------|
| **CDC & Setup** | S1-S2 | CDC validé, environnement prêt | CDC mergé, repo configuré, CI/CD basique, Discord setup |
| **Formation ROS2 & Nav** | S3-S4 | Apprendre ROS2, navigation, bases web | Tutoriels ROS2/Gazebo/Nav2 complétés, squelette React + Node.js |
| **Formation Manipulation** | S5-S6 | Apprendre MoveIt2, vision | Tutoriels MoveIt2 complétés, OpenCV basics, maquettes UI |
| **POC Navigation** | S7-S8 | Valider navigation | Robot simulé navigue dans Gazebo, MQTT testé, WebSocket fonctionne |
| **POC Manipulation** | S9-S10 | Valider manipulation | Bras saisit objet en simulation, détection objet fonctionne |
| **MVP Web** | S7-S12 | Dashboard fonctionnel | Interface accessible complète, API REST, connexion MQTT mockée |
| **MVP Robot** | S11-S14 | Navigation + manipulation réelle | SLAM calibré, robot navigue et saisit objets réels |
| **Intégration** | S15-S16 | Système complet | Web + Robot communiquent, tests end-to-end, scénario complet fonctionnel |
| **Polish & Demo** | S17-S18 | Prêt pour évaluation | Bug fixes, documentation finale, démo préparée, répétitions |

**Note:** MVP Web en parallèle des phases POC et MVP Robot. Convergence en phase Intégration.

**Répartition suggérée:**
- Maxime: Dashboard web, API, intégration MQTT côté serveur
- Wissem: ROS2, navigation, manipulation, bridge MQTT côté robot
- Cross-training régulier pour que chacun comprenne l'ensemble

### Questions ouvertes

**Communication:**
- [ ] MQTT ou Serial USB pour communication robot ? → À valider en POC S7-S8
- [ ] Quel broker MQTT ? (Mosquitto local vs cloud) → S7
- [ ] Format exact des messages robot ↔ serveur ? → S7-S8

**Navigation:**
- [ ] Quelle lib pour la carte temps réel ? (Leaflet, custom canvas, Three.js 2D) → S7
- [ ] Comment simuler le robot pour les tests automatisés ? → S7

**Manipulation:**
- [ ] Quels objets exactement pour la démo ? (livres, boîtes, documents) → S5
- [ ] Vision pure ou QR codes/marqueurs pour détection ? → POC S9-S10
- [ ] Poses de saisie prédéfinies ou calcul dynamique ? → POC S9-S10
- [ ] Calibration hand-eye: méthode automatique ou manuelle ? → S11

**Fonctionnel:**
- [ ] Faut-il un système de file d'attente de missions ? → S7

---

## Annexes

### Extensions futures (hors MVP)

Si le temps le permet, les extensions suivantes sont envisagées par ordre de priorité :

1. **Commandes vocales** - Intégration Web Speech API
2. **File d'attente de missions** - Gestion de plusieurs demandes en série
3. **Historique et analytics** - Dashboard admin avec statistiques d'utilisation
4. **Reconnaissance avancée d'objets** - Modèle ML custom pour nouveaux objets

### Ressources d'apprentissage

**ROS2 & Navigation:**
- [ROS2 Humble Tutorials](https://docs.ros.org/en/humble/Tutorials.html)
- [Nav2 Documentation](https://docs.nav2.org/)
- [TurtleBot3 Simulation](https://emanual.robotis.com/docs/en/platform/turtlebot3/simulation/)
- [Gazebo Tutorials](https://gazebosim.org/docs)

**Manipulation:**
- [MoveIt2 Tutorials](https://moveit.picknik.ai/main/doc/tutorials/tutorials.html)
- [OpenCV Python Tutorials](https://docs.opencv.org/4.x/d6/d00/tutorial_py_root.html)
- [Intel RealSense ROS2](https://github.com/IntelRealSense/realsense-ros)

**Hardware:**
- [Yahboom Transbot Documentation](https://category.yahboom.net/collections/jetson)
- [Jetson Nano Developer Kit](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit)
