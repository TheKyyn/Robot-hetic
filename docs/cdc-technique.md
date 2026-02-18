# IMPORTANT : ceci est une V1.0 du CdC, il est voué à évoluer dans les prochains jours. Celui-ci a été produit après un long après-midi de brainstorming et a # # comme premier rôle d'être une solide base, et fait état de notre vision du projet au début de ce dernier. La ROADMAP et d'autres sections seront sujettes à # # évolution au fur et à mesure que notre compréhension et vision du projet évoluent.

# Cahier des Charges Technique - RobLaude

**Version:** 1.0
**Date:** 2025-02-18
**Équipe:** Maxime Theophilos, Wissem Karboub
**Projet:** HETIC Web3 - Robotique

---

## 1. Vision

**RobLaude** est un robot d'assistance autonome qui transporte des documents entre points d'un ERP (Établissement Recevant du Public), contrôlé via une interface web accessible, pour améliorer l'autonomie des personnes à mobilité réduite.

---

## 2. Objectifs et Périmètre

### Objectifs

1. Naviguer de manière autonome dans un espace intérieur en évitant les obstacles
2. Transporter des objets légers (documents, petits colis) entre deux points désignés
3. Fournir une interface web accessible (WCAG AA) pour le contrôle et le monitoring en temps réel
4. Communiquer l'état du robot via WebSocket (position, statut mission, alertes)

### Non-objectifs (MVP)

1. Pas de manipulation d'objets avec le bras robotique (extension future)
2. Pas de gestion multi-robots (fleet management)
3. Pas de reconnaissance faciale ou vocale
4. Pas de fonctionnement en extérieur

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
| UC-02 | Suivre la mission | Utilisateur | Voir la position et l'état du robot en temps réel |
| UC-03 | Arrêt d'urgence | Utilisateur / Système | Stopper immédiatement le robot |
| UC-04 | Configurer les points | Admin | Définir les points de collecte/livraison |
| UC-05 | Consulter l'historique | Admin | Voir les missions passées et statistiques |

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

### UC-03 : Arrêt d'urgence (détaillé)

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
| **Robot OS** | ROS Noetic (Ubuntu 20.04) | Standard industrie, compatible Jetson Nano |
| **Simulation** | Gazebo | Simulateur ROS, permet de dev sans hardware |
| **Navigation** | ROS Navigation Stack + SLAM | Solution éprouvée pour navigation autonome |

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
| Courbe d'apprentissage ROS trop longue | **Haute** | Fort | Commencer par tutoriels Gazebo dès S1, prévoir 2-3 semaines de formation avant hardware |
| Hardware livré en retard | Moyenne | Fort | Architecture découplée permet d'avancer sur web + simulation sans robot |
| Latence MQTT trop élevée | Faible | Moyen | Tests de latence dès réception hardware, fallback sur Serial USB si besoin |
| Navigation imprécise en environnement réel | **Haute** | Moyen | Calibration SLAM progressive, commencer avec parcours simples, agrandir zone |
| Intégration ROS ↔ Node.js complexe | Moyenne | Moyen | Utiliser bibliothèque MQTT standard, POC d'intégration S3-S4 |
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
| **Formation & Fondations** | S3-S4 | Apprendre ROS, poser bases web | Tutoriels ROS/Gazebo complétés, squelette React + Node.js, maquettes UI |
| **POC** | S5-S6 | Valider choix techniques | Robot simulé navigue dans Gazebo, WebSocket fonctionne, MQTT testé |
| **MVP Web** | S7-S10 | Dashboard fonctionnel | Interface accessible complète, API REST, connexion MQTT mockée |
| **MVP Robot** | S8-S12 | Navigation réelle | SLAM calibré, robot navigue entre 2 points réels, intégration serveur |
| **Intégration** | S13-S15 | Système complet | Web + Robot communiquent, tests end-to-end, scénario complet fonctionnel |
| **Polish & Demo** | S16-S18 | Prêt pour évaluation | Bug fixes, documentation finale, démo préparée, répétitions |

**Note:** MVP Web et MVP Robot en parallèle (Maxime / Wissem), convergence en phase Intégration.

### Questions ouvertes

- [ ] MQTT ou Serial USB pour communication robot ? → À valider en POC S5-S6
- [ ] Quel broker MQTT ? (Mosquitto local vs cloud) → S5
- [ ] Format exact des messages robot ↔ serveur ? → S5-S6
- [ ] Quelle lib pour la carte temps réel ? (Leaflet, custom canvas, Three.js 2D) → S7
- [ ] Comment simuler le robot pour les tests automatisés ? → S5
- [ ] Faut-il un système de file d'attente de missions ? → S7
- [ ] Extension bras robotique réaliste dans le temps restant ? → Revue S12

---

## Annexes

### Extensions futures (hors MVP)

Si le temps le permet, les extensions suivantes sont envisagées par ordre de priorité :

1. **Manipulation avec bras robotique** - Saisir un livre sur une étagère (UC bibliothèque)
2. **Commandes vocales** - Intégration Web Speech API
3. **File d'attente de missions** - Gestion de plusieurs demandes en série
4. **Historique et analytics** - Dashboard admin avec statistiques d'utilisation

### Ressources d'apprentissage ROS

- [ROS Noetic Tutorials](http://wiki.ros.org/noetic/Tutorials)
- [Gazebo Tutorials](http://gazebosim.org/tutorials)
- [Yahboom Transbot Documentation](https://category.yahboom.net/collections/jetson)
- [Navigation Stack Guide](http://wiki.ros.org/navigation)
