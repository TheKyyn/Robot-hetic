# Roadmap RobLaude v1.4 — Février → Juin 2026

## Paramètres du projet

| Paramètre | Valeur |
|-----------|--------|
| Équipe | 2 personnes (Wissem + Maxime) |
| Durée | 18 semaines — 17 fév → 26 juin 2026 |
| Organisation | 9 sprints de 2 semaines |
| Granularité | Mini-tickets de 1-2h avec test obligatoire |
| Hardware | Yahboom Transbot non arrivé → simulation Gazebo |
| Zone de confort | Web (React, Node.js) ✅ — ROS2 débutants ⚠️ |
| Frontend | PWA (React + Vite) |

---

## Timeline macro

```
         Fév              Mars              Avril             Mai               Juin
    ┌─────────┐     ┌─────────┐     ┌─────────────┐  ┌──────────────┐  ┌──────────────┐
    │ PHASE 1 │     │ PHASE 2 │     │   PHASE 3   │  │   PHASE 4    │  │   PHASE 5    │
    │Fondation│     │Robot +  │     │ Intégration │  │ Manipulation │  │Polish +      │
    │ + Web   │     │Communic.│     │  UC-01 E2E  │  │ Bras UC-02   │  │ Soutenance   │
    │ S1  S2  │     │ S3  S4  │     │    S5       │  │  S6    S7    │  │  S8    S9    │
    └─────────┘     └─────────┘     └─────────────┘  └──────────────┘  └──────────────┘
    🟢 Facile       🟡 Moyen        🟢 Critique       🔴 Difficile      🟡 Moyen
    
 JALONS:
    J0             J1         J2                J3                 J4              J5
    Env prêt       Web MVP    Robot navigue     UC-01 complet      UC-02 complet   Soutenance
    2 mars         16 mars    30 mars           27 avril           25 mai          26 juin
```

---

## Phase 1 — Fondations (S1 → S2 : 17 fév → 16 mars)

Objectif : avoir un dashboard web fonctionnel et un robot qui bouge en simulation. Les deux indépendants, on peut bosser en parallèle.

### Sprint 1 — Fondations & Environnement (17 fév → 2 mars)

| Donnée | Valeur |
|--------|--------|
| Objectif | Tout le monde peut coder, compiler, simuler |
| Jalon J0 | Environnements prêts, CI passe, Gazebo tourne |
| Tickets | 15 |
| Heures estimées | 22h |
| Buffer | 4h |
| Risque | 🟢 Facile (sauf install ROS2 qui peut être galère) |

**Epics :**

| Epic | Tickets | Contenu |
|------|---------|---------|
| 1.1 Infrastructure Dev | 8 | Monorepo GitHub, projet React+Vite en PWA, projet Node+Express, base de données Prisma+MySQL, config tests (Vitest + Playwright), CI GitHub Actions, Docker Compose pour les services locaux |
| 1.2 Env Robot Simulation | 7 | Installation Ubuntu 22.04 + ROS2 Humble, premier package ROS2, installation Gazebo, import du modèle robot, création du monde simulé (couloirs ERP), contrôle clavier du robot |

### Sprint 2 — Web MVP (3 mars → 16 mars)

| Donnée | Valeur |
|--------|--------|
| Objectif | Dashboard CRUD missions fonctionnel, PWA installable |
| Jalon J1 | Web MVP fonctionnel |
| Tickets | 23 |
| Heures estimées | 35.5h |
| Buffer | 3h |
| Risque | 🟢 Facile (zone de confort web) |

**Epics :**

| Epic | Tickets | Contenu |
|------|---------|---------|
| 2.1 Backend API REST | 10 | Schéma base de données complet, CRUD missions, CRUD points (lecture + admin), état robot, validation des entrées, authentification JWT |
| 2.2 Frontend Dashboard | 11 | Layout responsive (sidebar desktop / bottom nav mobile), page dashboard avec stats, page missions avec filtres, formulaire de création, page détail mission, composants de statut, gestion d'état, pages login/register, navigation clavier WCAG, bouton STOP accessible, config PWA (manifest + icônes + service worker) |
| 2.3 Admin basique | 2 | Page admin gestion des points, page historique missions |

---

## Phase 2 — Robot & Communication (S3 → S4 : 17 mars → 13 avril)

Objectif : le robot navigue tout seul en simulation ET il communique avec le serveur web. C'est la phase où on apprend vraiment ROS2.

### Sprint 3 — Navigation Autonome (17 mars → 30 mars)

| Donnée | Valeur |
|--------|--------|
| Objectif | Robot navigue d'un point A à un point B en simulation |
| Jalon J2 | Navigation autonome fonctionnelle |
| Tickets | 15 |
| Heures estimées | 26h |
| Buffer | 5h |
| Risque | 🟡 Moyen (Nav2 peut être difficile à configurer) |

**Epics :**

| Epic | Tickets | Contenu |
|------|---------|---------|
| 3.1 SLAM & Cartographie | 5 | Installation Nav2 + SLAM Toolbox, configuration LiDAR simulé, cartographie du monde Gazebo, localisation sur la carte |
| 3.2 Navigation Nav2 | 8 | Configuration des paramètres Nav2, envoi d'objectifs de navigation (via RViz puis via code), points nommés du bâtiment, orchestrateur de missions côté robot, évitement d'obstacles, gestion des timeouts, publication de la position du robot |
| 3.3 Formation ROS2 | 2 | Documentation interne sur les concepts ROS2 qu'on découvre (transforms, lifecycle Nav2) |

### Sprint 4 — Communication Robot ↔ Serveur (31 mars → 13 avril)

| Donnée | Valeur |
|--------|--------|
| Objectif | Le robot et le serveur communiquent en temps réel, le dashboard affiche la position live |
| Jalon | Communication bidirectionnelle fonctionnelle |
| Tickets | 22 |
| Heures estimées | 30h |
| Buffer | 4h |
| Risque | 🟡 Moyen |

C'est à ce sprint qu'on définira l'architecture MQTT exacte (topics, format des messages, niveaux de fiabilité) en fonction de ce qu'on aura appris sur ROS2 au Sprint 3.

**Epics :**

| Epic | Tickets | Contenu |
|------|---------|---------|
| 4.1 Bridge ROS2 ↔ MQTT | 9 | Noeud Python qui fait le pont entre ROS2 et MQTT : transmettre les ordres du serveur au robot, remonter la position et les statuts du robot vers le serveur, gérer l'arrêt d'urgence et la reprise, gérer la confirmation de chargement |
| 4.2 Adaptateur MQTT côté serveur | 8 | Client MQTT dans le backend Node.js : envoyer les ordres au robot, recevoir et traiter les retours (accusés, position, fin de mission), mettre à jour la base de données, transmettre au frontend via WebSocket |
| 4.3 Temps réel Frontend | 5 | Connexion WebSocket, mise à jour de la position sur la carte 2D, notifications de changement de statut, carte tactile (zoom/pan sur mobile), bouton STOP connecté au backend |

---

## Phase 3 — Intégration UC-01 E2E (14 avril → 27 avril)

### Sprint 5 — UC-01 de bout en bout

| Donnée | Valeur |
|--------|--------|
| Objectif | Le transport de document fonctionne de bout en bout |
| Jalon J3 | UC-01 complet et testé |
| Tickets | 14 |
| Heures estimées | 24h |
| Buffer | 3h |
| Risque | 🟢 Les composants sont déjà prêts, il faut les assembler |

**Epics :**

| Epic | Tickets | Contenu |
|------|---------|---------|
| 5.1 Flux UC-01 complet | 7 | Création de mission → robot navigue vers collecte → notification d'arrivée → confirmation utilisateur → transport → livraison, plus les cas d'erreur : arrêt d'urgence en cours de route (reprendre/annuler), annulation, timeout |
| 5.2 Tests E2E & Accessibilité | 5 | Tests Playwright du parcours complet, test arrêt d'urgence, audit accessibilité clavier + mobile, audit axe-core, test d'intégration MQTT sans Gazebo |
| 5.3 Documentation | 2 | README à jour, documentation API |

---

## Phase 4 — Bras Robotique (28 avril → 25 mai)

C'est la phase la plus risquée du projet. On découvre MoveIt2 et la vision par ordinateur. Le CDC v1.4 est honnête là-dessus : on ne sait pas encore ce qui va marcher et ce qui ne marchera pas. Le nombre de tentatives de saisie, la stratégie exacte, les limites du bras — tout ça sera déterminé pendant ces sprints.

### Sprint 6 — Découverte MoveIt2 & Vision (28 avril → 11 mai)

| Donnée | Valeur |
|--------|--------|
| Objectif | Le bras bouge en simulation + on détecte un objet avec la caméra |
| Jalon | Bras fonctionnel + objet détecté |
| Tickets | 12 |
| Heures estimées | 24h |
| Buffer | 5h |
| Risque | 🔴 Difficile (MoveIt2 = partie la plus risquée du projet) |

**Epics :**

| Epic | Tickets | Contenu |
|------|---------|---------|
| 6.1 MoveIt2 Setup | 5 | Installation, configuration pour le bras du Transbot, premiers mouvements simples, contrôle du gripper, définition de quelques poses de base |
| 6.2 Vision & Détection | 5 | Caméra de profondeur simulée dans Gazebo, détection d'un objet simple par couleur (OpenCV), calcul de la position 3D de l'objet, transformation de coordonnées caméra → robot, objet saisissable dans Gazebo |
| 6.3 Formation | 2 | Documentation interne MoveIt2, documentation calibration caméra-bras |

### Sprint 7 — Saisie & UC-02 complet (12 mai → 25 mai)

| Donnée | Valeur |
|--------|--------|
| Objectif | Le robot saisit un objet, le transporte et le dépose en simulation |
| Jalon J4 | UC-02 fonctionne en simulation |
| Tickets | 17 |
| Heures estimées | 31h |
| Buffer | 5h |
| Risque | 🔴 Difficile |

**Epics :**

| Epic | Tickets | Contenu |
|------|---------|---------|
| 7.1 Saisie automatique | 5 | Planification de trajectoire de saisie, exécution, logique de réessai en cas d'échec (nombre de tentatives à déterminer), détection de chute pendant transport, séquence de dépôt |
| 7.2 Intégration UC-02 Robot | 3 | Ajout du flow pick & place dans l'orchestrateur de missions, communication des sous-états vers le serveur, communication des résultats de saisie |
| 7.3 Intégration UC-02 Web | 6 | Extension de l'API pour les missions de récupération, gestion des objets (CRUD admin), traitement des retours du robot côté serveur, formulaire "Récupérer un objet" dans le frontend, affichage de la progression, page admin des objets |
| 7.4 Tests UC-02 | 3 | Tests E2E du parcours réussi, test du cas d'échec, test d'intégration |

---

## Phase 5 — Polish, Hardware & Soutenance (26 mai → 26 juin)

### Sprint 8 — Stabilisation & Hardware (26 mai → 8 juin)

| Donnée | Valeur |
|--------|--------|
| Objectif | Couverture tests max, accessibilité validée, migration hardware si possible |
| Jalon | Projet stable |
| Tickets | 17 |
| Heures estimées | 28h |
| Buffer | 4h |
| Risque | 🟡 Moyen |

**Epics :**

| Epic | Tickets | Contenu |
|------|---------|---------|
| 8.1 Tests & Qualité | 6 | Couverture tests backend >70%, frontend >60%, suite E2E desktop + mobile, audit accessibilité complet, tests de robustesse (charge WebSocket, reconnexion MQTT) |
| 8.2 Hardware (si reçu) | 6 | Installation sur Jetson Nano, branchement LiDAR et caméra, cartographie réelle, test navigation réelle, test saisie réelle |
| 8.3 Fallback Gazebo (si pas de hardware) | 5 | Amélioration visuelle du monde simulé, scénarios de démo scriptés, vidéo de démo enregistrée, enregistrement ROS2 pour replay |

### Sprint 9 — Documentation & Soutenance (9 juin → 26 juin)

| Donnée | Valeur |
|--------|--------|
| Objectif | Tout est prêt pour le jour J |
| Jalon J5 | Soutenance prête |
| Tickets | 13 |
| Heures estimées | 24h |
| Buffer | 2h |
| Risque | 🟢 Facile |

**Epics :**

| Epic | Tickets | Contenu |
|------|---------|---------|
| 9.1 Documentation technique | 5 | README final, documentation API complète, diagrammes UML, schéma d'architecture, guide de déploiement |
| 9.2 Présentation & Démo | 5 | Slides (~20), script de démo avec rôles, vidéo de secours, répétition complète, Q&A anticipées |
| 9.3 Retro & Business | 3 | Retrospective, business plan simplifié, analyse des coûts |

---

## Résumé des jalons

| Jalon | Sprint | Date | On considère que c'est OK si... |
|-------|--------|------|-------------------------------|
| J0 | Sprint 1 | 2 mars | La CI est verte, Gazebo tourne, on peut contrôler le robot au clavier |
| J1 | Sprint 2 | 16 mars | Le dashboard fonctionne (CRUD missions), la PWA est installable |
| J2 | Sprint 3 | 30 mars | Le robot navigue seul d'un point A à un point B en simulation |
| J3 | Sprint 5 | 27 avril | UC-01 complet : création → collecte → transport → livraison |
| J4 | Sprint 7 | 25 mai | UC-02 complet : le robot saisit, transporte et dépose un objet en simulation |
| J5 | Sprint 9 | 26 juin | Démo prête, docs complètes, slides prêtes |

---

## Matrice de risques par sprint

| Sprint | Risque principal | Buffer | Mitigation |
|--------|-----------------|--------|------------|
| S1 | Installation ROS2 qui galère | 4h | Tuto officiel + Docker en plan B |
| S2 | Aucun (zone de confort) | 3h | — |
| S3 | Nav2 difficile à configurer | 5h | Partir des exemples officiels |
| S4 | Problèmes de communication | 4h | Tout tester en local avec Docker |
| S5 | Imprévus d'intégration | 3h | Mocks robustes + tests indépendants |
| S6 | **MoveIt2 trop complexe** | 5h | Poses prédéfinies en plan B |
| S7 | **Saisie qui ne marche pas** | 5h | Simplifier l'objet (gros cube) |
| S8 | Hardware pas arrivé | 4h | Fallback Gazebo (Epic 8.3) |
| S9 | Stress de dernière minute | 2h | Vidéo de secours filmée dès le Sprint 8 |

---

## Compteurs globaux

| Métrique | Valeur |
|----------|--------|
| Total tickets | ~148 |
| Total heures estimées | ~245h |
| Total buffer | ~35h (14%) |
| Charge par personne | ~122h sur 18 semaines ≈ 6.8h/semaine |