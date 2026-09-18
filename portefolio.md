# Kamil Cassam-Chenaï - Réalisations professionnelles

---

## ✈️ NG TRAVEL — Senior Frontend Engineer (06/2026 – En cours)

---

### ⭐ Optimisation des appels vers le BFF (cache serveur cross-visiteurs)

**Contexte**
Le header et le footer généraient un flood d'appels au BFF côté serveur — jusqu'à 50 appels/seconde en pic — car React Query recrée un `QueryClient` neuf à chaque requête serveur, donc sans mémoire d'une requête à l'autre.

**Rôle**
Diagnostiquer la cause racine et concevoir un mécanisme de cache serveur partagé entre visiteurs.

**Actions**
- Helper générique combinant `unstable_cache` (Next.js, cache cross-requête) et `cache()` React (dédup intra-requête), mutualisé sur les 4 services concernés
- Durées de cache nommées et documentées, calées sur le cycle réel de mise à jour de la donnée source après recherche technique
- Architecture prête pour une invalidation à la demande (tags par ressource, `revalidateTag`, webhook CMS)
- Correction de deux bugs préexistants découverts au passage (config React Query dupliquée test/prod, cache inutilement désactivé sur l'autocomplete)
- Benchmark réel avant/après sur build de production

**Résultats**
- 8 appels BFF par visite → 0 appel sur visite répétée
- Temps de réponse serveur (Lighthouse) : 560 ms → 30–60 ms
- Bug de flood identique corrigé au passage sur un endpoint non couvert initialement
- Architecture réutilisable pour tout futur service BFF

---

### ⭐ Carousel d'images sur les cartes produit (performance + UX)

**Contexte**
Les cartes produit des résultats de recherche n'affichaient qu'une image statique ; besoin d'un carousel swipable sans dégrader la performance d'une liste de plusieurs dizaines de cartes.

**Rôle**
Concevoir et développer le composant (Design System) et son intégration front, en garantissant la performance sur liste longue.

**Actions**
- Carousel swipable avec dots + flèches de navigation (desktop/mobile)
- Priorisation LCP : premier slide de la première carte en chargement prioritaire, tout le reste en lazy
- Montage différé du carousel complet via `IntersectionObserver` mutualisé — une carte hors champ ne charge qu'une image simple, sans monter le carousel
- `content-visibility`/`contain-intrinsic-size` sur chaque carte de la liste pour éviter le recalcul de mise en page hors écran, hauteurs mesurées en conditions réelles
- Fusion avec un hook `useIntersection` existant plutôt que dupliqué, durcissement de 3 comportements latents validés chacun par test négatif

**Résultats**
- Coût de rendu réduit sur les pages de résultats
- LCP mieux maîtrisé (image prioritaire systématique)
- Composant livré en Design System + front (deux PR jumelles)

---

### ⭐ Réduction du CLS sur la fiche produit (priorisation image + skeleton)

**Contexte**
Le LCP de la fiche produit n'était pas optimisé, et l'affichage progressif des blocs créait un effet de cascade visuel le temps que les données arrivent.

**Rôle**
Prioriser le chargement de l'image principale et mettre en place un skeleton fidèle à la mise en page finale.

**Actions**
- Attributs de priorisation image (chargement prioritaire) sur la première image desktop et mobile de la fiche produit, chargement différé sans priorité sur les suivantes
- Skeleton de page produit affiché pendant le chargement des données, mobile et desktop

**Résultats**
- Image principale de la fiche produit priorisée pour le LCP
- Skeleton fonctionnel, finalisation en cours

---

### ⭐ Widget de réservation (BookingWidget) — composant réutilisable desktop/mobile

**Contexte**
La fiche produit avait besoin d'un widget de réservation complet (ville de départ, dates, voyageurs, agence), avec des contraintes d'accessibilité fortes et une intégration poussée de composants tiers.

**Rôle**
Concevoir et développer le composant "from scratch" en Design System, réutilisable desktop/mobile, en résolvant les problèmes d'intégration complexes rencontrés en cours de route.

**Actions**
- Développement du widget de réservation (DS) : sélecteurs ville de départ / dates / voyageurs / agence, prix, paiement, call center
- Déclinaison desktop (Dialog) et mobile (Sheet), avec un pattern commun de step modal réutilisé sur les 4 étapes
- Résolution d'une limitation non documentée d'une librairie tierce (composant `Select` imbriqué dans une modale) empêchant la fermeture de la modale lors d'une sélection au clavier — diagnostic par lecture du code source de la librairie, validé empiriquement (souris, clavier, tactile) avant d'écarter plusieurs alternatives plus fragiles
- Choix techniques documentés et sourcés (doc officielle, code source des libs) directement dans les PR, pour rester traçables par l'équipe

**Résultats**
- Composant réutilisable et extensible, décliné desktop/mobile
- Robustesse d'accessibilité clavier vérifiée sur un cas limite non couvert par l'implémentation "évidente"
- Base technique documentée pour les prochaines étapes du funnel de réservation

---

### ⭐ Architecture des états de chargement des filtres (skeleton vs désactivation)

**Contexte**
Sur la page de résultats, relancer une recherche différente (destination, dates...) laissait la colonne filtres bloquée sur les anciennes valeurs — grisée, sans skeleton — au lieu de repartir sur un état de chargement clair. Un simple changement de tri ne devait en revanche jamais déclencher ce comportement.

**Rôle**
Diagnostiquer la cause racine (un seul signal de chargement pilotant tout l'affichage) et redessiner la logique de chargement/désactivation.

**Actions**
- Distinction explicite de deux signaux : chargement (skeleton, uniquement au tout premier accès) et désactivation temporaire (interface grisée/inerte, à chaque nouveau résultat)
- Conservation conditionnelle des filtres précédents : gardés seulement si c'est la même recherche affinée (hors tri), sinon reset complet vers l'état de chargement initial
- Contournement d'un piège d'implémentation découvert en test réel : la désactivation ne traverse pas les portals (le composant mobile est monté hors de l'arbre DOM visible), nécessitant un ciblage plus précis qu'un seul wrapper commun
- Vérification en conditions réelles (tests navigateur automatisés, appels BFF interceptés) et comparaison avec un concurrent du secteur pour challenger certains choix UX

**Résultats**
- Skeleton affiché uniquement au premier chargement réel, jamais sur un simple changement de tri
- Filtres réellement désactivés (pas seulement figés visuellement) pendant le chargement d'une nouvelle recherche
- Changement minimal et ciblé, sans régression

---

## 🎮 FDJ UNITED — Senior Frontend Engineer (01/2025 – 12/2025)

---

### ⭐ Refonte du parcours Favoris - BottomSheet + refacto UI

**Contexte**
Le parcours Favoris devait être modernisé pour améliorer l'UX, réduire la dette technique et permettre une évolution rapide. La solution existante reposait sur une librairie tierce difficile à adapter aux besoins FDJ.

**Rôle**
Défendre une approche "from scratch" pour créer un composant réutilisable, performant et totalement maîtrisé. Préparer le terrain via une refacto préalable des composants UI.

**Actions**
- Refacto des composants UI pour les découpler de Formik et hooks API
- Conception d'un BottomSheet générique, modulaire et extensible
- Développement complet du parcours Favoris (multi-APIs, gestion fine des états)
- Optimisation des performances (réduction du JS inutile, rendu conditionnel)
- Collaboration PO/Design
- Architecture permettant d'ajouter de nouvelles règles métier sans réécriture

**Résultats**
- Composant réutilisable dans plusieurs parcours FDJ
- Performance améliorée
- Dette technique réduite
- Parcours plus fluide et maintenable
- Base technique saine pour les futures évolutions

---

### ⭐ Stabilisation mobile/desktop et correction des problèmes d'hydratation Next.js

**Contexte**
Certaines pages présentaient des divergences SSR/CSR entraînant warnings d'hydratation et décalages visuels.

**Rôle**
Identifier les causes et stabiliser l'affichage sur mobile et desktop.

**Actions**
- Audit des composants impactés
- Séparation client/server pour éviter les divergences
- Correction des dépendances non déterministes
- Garde-fous pour un rendu identique SSR/CSR
- Tests sur devices réels

**Résultats**
- Disparition des warnings d'hydratation
- Affichage stable sur tous les devices
- Réduction des bugs visuels

---

### ⭐ Playlist vidéo modulaire (HLS/YouTube + React Query)

**Contexte**
Besoin d'une expérience vidéo fluide, fiable et compatible multi-sources.

**Rôle**
Concevoir un module vidéo robuste, modulaire et performant.

**Actions**
- Playlist modulaire HLS/YouTube
- Préchargement intelligent via React Query
- Gestion fine des fallback
- Responsive + UX fluide
- Tests unitaires

**Résultats**
- Expérience vidéo plus rapide et fiable
- Réduction des erreurs de lecture
- Module réutilisable

---

### ⭐ Refonte du bandeau promotionnel (UX + barre de progression)

**Contexte**
Le bandeau manquait de clarté et d'impact.

**Rôle**
Moderniser le composant pour améliorer l'engagement.

**Actions**
- Refonte UI/UX
- Barre de progression animée
- Optimisation du rendu
- Accessibilité + responsive

**Résultats**
- Composant plus clair et engageant
- Réutilisation facilitée

---

### ⭐ Accessibilité (ARIA, navigation clavier)

**Contexte**
Besoin de renforcer l'accessibilité des parcours FDJ.

**Rôle**
Identifier les points bloquants et améliorer l'expérience clavier/lecteurs d'écran.

**Actions**
- Audit accessibilité
- Correction ARIA
- Gestion du focus
- Tests manuels

**Résultats**
- Parcours plus accessibles
- Navigation clavier fluide

---

## EKWATEUR — Senior Frontend Engineer (03/2023 – 09/2024)

---

### ⭐ Refonte complète du blog Next.js (SEO, performances, CMS headless)

**Contexte**
Le blog devait être modernisé pour améliorer SEO, performance et autonomie contenu.

**Rôle**
Concevoir une nouvelle version performante et SEO-friendly.

**Actions**
- Refonte complète en Next.js (App Router)
- Intégration Prismic + SSG
- Optimisations SEO
- Responsive + UX
- Storybook + tests snapshot

**Résultats**
- Blog plus rapide et mieux référencé
- Autonomie renforcée
- Architecture modernisée

---

### ⭐ Section Auteurs (pages dynamiques + SSG)

**Contexte**
Besoin de valoriser les auteurs et améliorer la navigation éditoriale.

**Rôle**
Créer une section Auteurs complète et performante.

**Actions**
- Pages dynamiques + SSG
- Intégration Prismic
- Responsive + SEO
- Composants réutilisables

**Résultats**
- Pages auteurs performantes
- Navigation enrichie
- Autonomie accrue

---

### ⭐ Migration Pages Router → App Router

**Contexte**
Le site utilisait encore l'ancien Pages Router.

**Rôle**
Piloter la migration vers App Router.

**Actions**
- Migration progressive
- Mise en place des Server Components
- Refonte routing + data fetching
- Nettoyage de la dette technique

**Résultats**
- Architecture modernisée
- Réduction du JS client
- Migration fluide sans perte SEO

---

### ⭐ Optimisations SEO

**Contexte**
Besoin d'améliorer le référencement naturel.

**Rôle**
Identifier et implémenter les optimisations techniques.

**Actions**
- Données structurées
- Maillage interne
- Balisage HTML
- Optimisation images

**Résultats**
- SEO renforcé
- Pages mieux structurées
- Temps de chargement réduits

---

### ⭐ Storybook + tests snapshot

**Contexte**
Manque de documentation UI et de stabilité visuelle.

**Rôle**
Structurer la documentation et sécuriser les évolutions.

**Actions**
- Mise en place de Storybook
- Création de stories
- Tests snapshot
- Harmonisation des composants

**Résultats**
- Documentation claire
- Réduction des régressions
- Adoption facilitée

---

## Projets internes CBTW (2023 – 2025)

---

### ⭐ Questionnaire interactif DevFest Lille — React Hook Form + scoring + API

**Contexte**
CBTW souhaitait une expérience interactive sur son stand au DevFest Lille 2022 : un questionnaire ludique avec scoring et enregistrement des résultats.

**Rôle**
Concevoir et développer un questionnaire complet, fiable et responsive.

**Actions**
- Développement React + React Hook Form
- Calcul du score + envoi sécurisé vers API interne
- Hash des réponses pour éviter l'exposition directe dans le bundle
- UI responsive pour tablette/borne
- Gestion des validations et transitions UX

**Résultats**
- Expérience fluide et engageante
- Scores enregistrés pour analyse interne
- Protection des réponses sensibles
- Projet utilisé en production lors du DevFest

---

### ⭐ Outil interne de préparation aux entretiens clients — IA Grok

**Contexte**
Les commerciaux remontaient des questions techniques bloquantes selon les clients. Besoin d'un outil pour préparer les candidats avec évaluation IA.

**Rôle**
Concevoir un outil interne combinant base de questions, interface de préparation et scoring IA.

**Actions**
- Interface React pour sélectionner client/techno
- Base interne de questions techniques
- Évaluation automatique via API Grok
- Scoring + feedback qualitatif
- Architecture extensible
- POC validé et utilisé par les équipes commerciales/RH

**Résultats**
- Gain de temps pour commerciaux et RH
- Préparation plus efficace des candidats
- Outil interne réutilisable
- Usage concret de l'IA dans les workflows internes

---

## MATCHBOX — Frontend Engineer (04/2019 – 01/2023)

---

### ⭐ Migration Rolex & Tudor vers React

**Contexte**
Les plateformes legacy limitaient les évolutions et la performance.

**Rôle**
Participer à la migration vers React.

**Actions**
- Migration progressive vers React 15→17
- Refonte UI avec Styled Components
- Intégration AEM SPA
- Nettoyage de la dette technique

**Résultats**
- Plateformes modernisées
- UI plus stable
- Base technique durable

---

### ⭐ Référent front Tudor

**Contexte**
Tudor nécessitait un référent pour garantir qualité et stabilité.

**Rôle**
Assurer maintenance, évolutions et qualité UI.

**Actions**
- Suivi des évolutions
- Correction des bugs
- Harmonisation des composants
- Documentation

**Résultats**
- Plateforme plus stable
- Réduction des bugs
- Qualité premium maintenue

---

### ⭐ Store Locator Tudor → [Rolex.com](https://www.rolex.com)

**Contexte**
Le Store Locator Tudor devait être modernisé et réutilisable sur Rolex.com.

**Rôle**
Concevoir un composant modulaire et portable.

**Actions**
- Refonte complète en React
- Multi-APIs
- Optimisation performance
- Documentation
- Portage Rolex.com

**Résultats**
- Réutilisation réussie
- Composant plus rapide et fiable
- Transmission fluide

---

### ⭐ Optimisation des performances

**Contexte**
Les pages premium nécessitaient une performance irréprochable.

**Rôle**
Optimiser le rendu et réduire le JS.

**Actions**
- Lazy-loading
- Mémoïsation
- Découpage logique
- Audits Lighthouse

**Résultats**
- Temps de chargement réduits
- Core Web Vitals améliorés

---

### ⭐ Tracking analytics (Adobe DTM)

**Contexte**
Besoin d'un tracking fiable et cohérent.

**Rôle**
Mettre en place un tracking robuste.

**Actions**
- Data layers
- Configuration Adobe DTM
- Tests data
- Documentation

**Résultats**
- Tracking fiable
- Données marketing de meilleure qualité

---

## SELOGER — Frontend Engineer (01/2018 – 04/2019)

---

### ⭐ Migration progressive Vue.js → React via Vuera

**Contexte**
Besoin d'évoluer vers React sans freeze produit.

**Rôle**
Mettre en place une migration progressive.

**Actions**
- Intégration Vuera
- Migration progressive
- Nettoyage dette technique
- Collaboration front/back

**Résultats**
- Migration fluide
- Réduction dette technique
- Adoption progressive de React

---

### ⭐ Parcours utilisateurs complexes

**Contexte**
Besoin de parcours robustes pour calculs immobiliers et formulaires.

**Rôle**
Concevoir des interfaces fiables et performantes.

**Actions**
- Développement calculettes + formulaires
- Intégration APIs REST
- Gestion fine des erreurs
- Optimisation responsive

**Résultats**
- Parcours plus fiables
- Réduction des bugs
- UX améliorée
