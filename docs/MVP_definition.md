  MVP Definition - Draft

  Problème + utilisateur (3 lignes):

  Dans les ERP (mairies, hôpitaux), les personnes à mobilité réduite dépendent d'un tiers pour récupérer ou se faire livrer des documents et objets. 
  Cette dépendance quotidienne limite leur autonomie et surcharge le personnel. RobLaude permet à ces usagers de demander un transport ou une récupération d'objet via une interface web 
  accessible, sans assistance humaine.

  ---
  MVP en 5 bullets:

  1. Navigation autonome — Le robot se déplace d'un point A à un point B en évitant les obstacles (simulation Gazebo, hardware si disponible)
  2. Transport de document (UC-01) — L'utilisateur demande un transport → le robot va au point de collecte → l'utilisateur pose le document et confirme → le robot livre à destination
  3. Récupération d'objet (UC-02) — L'utilisateur demande un objet pré-enregistré → le robot navigue, détecte, saisit avec le bras, transporte et dépose
  4. Interface web accessible (PWA) — Dashboard responsive WCAG AA avec création de mission, suivi temps réel sur carte, et bouton d'arrêt d'urgence toujours visible
  5. Arrêt d'urgence (UC-04) — Stopper le robot immédiatement depuis l'interface (bouton + raccourci clavier), avec possibilité de reprendre ou annuler

  ---
  3 critères de succès mesurables:
  ┌─────────────────────────────┬──────────────────────────────────────────────────────────┬─────────────────────────────────────────────────┐
  │           Critère           │                          Cible                           │                Comment on mesure                │
  ├─────────────────────────────┼──────────────────────────────────────────────────────────┼─────────────────────────────────────────────────┤
  │ Taux de réussite navigation │ ≥ 90% des missions atteignent leur destination           │ Logs des missions en simulation (sur 20 tests)  │
  ├─────────────────────────────┼──────────────────────────────────────────────────────────┼─────────────────────────────────────────────────┤
  │ Taux de réussite saisie     │ ≥ 70% des tentatives de saisie réussies                  │ Logs MoveIt2 (sur 10 tests avec objet standard) │
  ├─────────────────────────────┼──────────────────────────────────────────────────────────┼─────────────────────────────────────────────────┤
  │ Accessibilité WCAG AA       │ 0 erreur critique axe-core + navigation clavier complète │ Audit Playwright + test manuel                  │
  └─────────────────────────────┴──────────────────────────────────────────────────────────┴─────────────────────────────────────────────────┘
  ---
  Contraintes:
  ┌─────────────────────┬────────────────────────────────────────────────────────────────────────────────┐
  │     Contrainte      │                                     Détail                                     │
  ├─────────────────────┼────────────────────────────────────────────────────────────────────────────────┤
  │ Environnement       │ Intérieur uniquement, sol plat, éclairage correct                              │
  ├─────────────────────┼────────────────────────────────────────────────────────────────────────────────┤
  │ Objets saisissables │ Légers (<500g), forme simple (cube, livre), position connue                    │
  ├─────────────────────┼────────────────────────────────────────────────────────────────────────────────┤
  │ Latence WebSocket   │ Position mise à jour toutes les 1-2 secondes (pas du vrai temps réel)          │
  ├─────────────────────┼────────────────────────────────────────────────────────────────────────────────┤
  │ Autonomie robot     │ Non mesurable tant qu'on n'a pas le hardware (batterie Transbot ~2h théorique) │
  ├─────────────────────┼────────────────────────────────────────────────────────────────────────────────┤
  │ Réseau              │ WiFi local requis, pas de fonctionnement offline                               │
  ├─────────────────────┼────────────────────────────────────────────────────────────────────────────────┤
  │ Équipe              │ 2 personnes, ~7h/semaine chacun                                                │
  ├─────────────────────┼────────────────────────────────────────────────────────────────────────────────┤
  │ Budget              │ 0€ (hardware fourni par l'école)                                               │
  └─────────────────────┴────────────────────────────────────────────────────────────────────────────────┘
  ---
  Top 3 risques + plan B:
  ┌───────────────────────────────────────┬─────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
  │                Risque                 │                                                   Plan B                                                    │
  ├───────────────────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
  │ MoveIt2 trop complexe / saisie échoue │ Poses de saisie prédéfinies codées en dur + objets très simples (gros cube) + réduire à 1 seul type d'objet │
  ├───────────────────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
  │ Détection objet non fiable            │ QR code ou marqueur ArUco collé sur l'objet au lieu de vision pure                                          │
  ├───────────────────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
  │ Hardware jamais livré                 │ Démo 100% Gazebo avec vidéo pré-enregistrée de qualité + scénarios scriptés                                 │
  └───────────────────────────────────────┴─────────────────────────────────────────────────────────────────────────────────────────────────────────────┘