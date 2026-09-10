# ESILV Better DeVinci - System Architecture & Showcase

[![Status](https://img.shields.io/badge/Status-Case%20Study%20%2F%20V2%20Roadmap-blue)](https://github.com/)
[![Architecture](https://img.shields.io/badge/Architecture-Event--Driven-success)](https://github.com/)
[![Target](https://img.shields.io/badge/Target-ESILV%20Students-orange)](https://github.com/)
[![Code](https://img.shields.io/badge/Source-Private%20Repository-lightgrey)](https://github.com/)
[![License](https://img.shields.io/badge/License-Proprietary-red)](https://github.com/)

Plateforme intelligente de suivi académique, moteur de calcul prédictif et système de notification pour les étudiants de l'ESILV.

---

## 1. Contexte et Vision du Projet

Les portails académiques universitaires souffrent généralement d'une ergonomie rigide et de fonctionnalités limitées pour le pilotage quotidien des études :
- Absence d'alertes instantanées lors de la parution ou modification d'une note.
- Difficulté à visualiser clairement l'impact d'une note sur la moyenne générale et les Unités d'Enseignement (UE).
- Manque d'outils d'anticipation pour planifier ses révisions et définir des objectifs chiffrés.

Ce projet a été développé pour offrir une expérience étudiante enrichie : synchronisation automatisée, analyse approfondie des pondérations, algorithme d'optimisation d'objectifs académiques et préparation d'un système de notification réactif.

---

## 2. Architecture Globale du Système

Le système articule plusieurs modules découplés garantissant performance, scalabilité et résilience :

```
+-------------------------------------------------------------------------+
|                           Portail Universitaire                         |
+------------------------------------+------------------------------------+
                                     |
                                     | (HTTPS / Session chiffrée)
                                     v
+------------------------------------+------------------------------------+
|                   Pipeline d'Ingestion & Extraction                     |
|  - Négociation de session et rotation de cookies                        |
|  - Parsing DOM & extraction d'arbres académiques                        |
+------------------------------------+------------------------------------+
                                     |
                                     | (Modèles de données normalisés)
                                     v
+------------------------------------+------------------------------------+
|                         Moteur Métier & Calcul                          |
|  - Résolution d'arbre de coefficients et moyennes d'UE                 |
|  - Moteur d'optimisation "Lazy Strategy" (Calcul d'objectifs)          |
|  - Détection mathématique de notes implicites                          |
+------------------------------------+------------------------------------+
                                     |
                                     | (Snapshots d'état)
                                     v
+------------------------------------+------------------------------------+
|                      Stateful Diff & Event Engine                       |
|  - Empreintes cryptographiques SHA-256 par évaluation                   |
|  - Détection de deltas : GRADE_PUBLISHED, COEFF_CHANGED, GRADE_MODIFIED |
+------------------+----------------------------------+-------------------+
                   |                                  |
                   v                                  v
+------------------+---------------+  +---------------+-------------------+
|     Stockage & Cache Local       |  |      Dispatcher d'Événements      |
|  - Persistance des métriques     |  |  - Webhooks Discord / Telegram    |
|  - Historique chronologique      |  |  - Push Notifications & Alertes   |
+----------------------------------+  +-----------------------------------+
```

---

## 3. Fonctionnalités Clés Développées (V1)

### A. Ingestion et Normalisation des Données
- **Session Management** : Authentification et maintien transparent des sessions via cookies sécurisés, avec gestion des redirections et ré-authentification automatique.
- **Modélisation Hiérarchique** : Structuration standardisée des données sous forme d'arbre :
  - `Semestre` -> `Unité d'Enseignement (UE)` -> `Matière` -> `Épreuve` -> `Coefficients et Notes`
- **Validation Robuste** : Gestion des formats spécifiques (notes sur 20, coefficients décimaux, statuts d'absence justifiée/injustifiée, dispenses et rattrapages).

### B. Moteur d'Analyse et Algorithmes Métier
- **Calcul Ascendant des Moyennes** : Calcul en temps réel selon les règles officielles de pondération :
  $$\text{Moyenne}_{UE} = \frac{\sum (\text{Note}_i \times \text{Coeff}_i)}{\sum \text{Coeff}_i}$$
- **Algorithme "Lazy Strategy" (Simulateur d'Objectifs)** :
  Calcul matriciel inversé permettant de déterminer la note exacte minimale requise sur les épreuves futures pour valider une matière, une UE ou l'intégralité du semestre.
- **Déduction Mathématique de Notes** : Détection des notes partielles non publiées individuellement mais déductibles des variations de moyennes intermédiaires.

### C. Interface Utilisateur & Expérience Étudiante
- **Dashboard Dynamique** : Visualisation claire et ergonomique développée avec des composants modernes et réactifs.
- **Indicateurs de Progression** : Graphiques d'évolution chronologique et seuils de validation par palier.

---

## 4. Spécifications Techniques et Algorithmiques

### Algorithme de Détection d'Événements (State Diffing)
Pour éviter les alertes redondantes et traiter les changements avec exactitude, le système applique un comparateur d'état à chaque synchronisation :

1. **Génération de Clé Unique** :
   $$\text{Key} = \text{SHA256}(\text{Code\_Matiere} + \text{Nom\_Epreuve} + \text{Index})$$
2. **Comparaison Snapshot ($T_0 \rightarrow T_1$)** :
   - Si la clé est nouvelle : Émission de l'événement `EVENT_NEW_GRADE`.
   - Si la valeur de note a changé : Émission de `EVENT_GRADE_UPDATED`.
   - Si le coefficient a été rectifié : Émission de `EVENT_COEFF_UPDATED`.
3. **Calcul d'Impact Instantané** : Le diff calcule immédiatement la variation relative sur la moyenne générale (ex: `+0.32 pt`).

---

## 5. Feuille de Route et Architecture V2

La version V2 du système est orientée vers un service autonome multi-utilisateurs et multi-plateformes :

### 1. Système de Notifications Multi-Canal
- **Notifications Push Mobiles** : Intégration Firebase Cloud Messaging (FCM) et Apple Push Notification service (APNs).
- **Webhooks & Bots Dédiés** : Connecteurs Discord et Telegram personnalisables par utilisateur avec filtres d'alertes.
- **Alertes de Présence & Emploi du Temps** : Calcul d'optimisation de l'agenda et suivi des quotas d'absences autorisées.

### 2. Fonctionnalités Communautaires & Gamification
- **Statistiques de Promotion** : Comparaison anonymisée des distributions de notes au sein d'une même promotion ou classe.
- **Système de Succès & Trophées** : Gamification des objectifs académiques pour encourager la régularité du travail.

### 3. Architecture Technique Cible
- **Backend Asynchrone** : Workers de synchronisation distribués gérés par file de messages (Redis Queue / Celery) avec rate-limiting adaptatif.
- **Frontend PWA / Mobile** : Application Web Progressive ultra-rapide avec stockage local chiffré et mode hors-ligne complet.
- **Sécurité à Connaissance Nulle (Zero-Knowledge Architecture)** : Les identifiants et données sensibles sont traités côté client ou chiffrés avec des clés dérivées propres à chaque utilisateur.

---

## 6. Sécurité, Éthique et Politique de Confidentialité

- **Respect des Systèmes Hôtes** : Les algorithmes d'interrogation intègrent des politiques strictes de backoff exponentiel et de limitation de débit afin de ne jamais dégrader les performances des infrastructures universitaires.
- **Protection des Données Personnelles** : Aucune donnée nominative ou identifiant n'est revendu, partagé ou conservé en clair.
- **Code Source Propriétaire** : L'implémentation complète du code source est maintenue dans un dépôt privé afin de préserver l'intégrité du système, de prévenir tout usage abusif et de respecter les conditions d'utilisation des plateformes partenaires.

---

## 7. Contact & Informations

Projet conçu et développé par **Louis** dans le cadre de recherches et développements en ingénierie logicielle, optimisation d'algorithmes et automatisation de flux de données.

Pour toute question technique ou opportunité de collaboration, vous pouvez me contacter via mon profil GitHub.
