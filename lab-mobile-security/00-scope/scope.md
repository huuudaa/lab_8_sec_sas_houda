# Périmètre d'analyse — LAB 8

## Cible autorisée
Nom: EduVulnApp — Application Android pédagogique
Propriétaire: Département Sécurité Informatique — Établissement d'enseignement supérieur

## Autorisation
Source: TP LAB 8 — Cours Sécurité Mobile (Semestre 2 — 2025/2026)
Preuve: Email de l'enseignant référencé dans le dossier de cours (réf. SEC-LAB-2026-08)
Document: Consignes de TP distribué en séance — accès ENT étudiant

## Type d'artefact
- [x] APK pédagogique fourni par l'enseignant
- [ ] Application interne autorisée
- [ ] Domaine explicitement autorisé

## Détail de la cible
Fichier: EduVulnApp-v1.0-debug.apk
Package: com.example.eduvulnapp
Version: 1.0 (build 1)
SDK minimum: 21 (Android 5.0 Lollipop)
SDK cible: 33 (Android 13)

## Limites explicites
- Pas d'exploitation des vulnérabilités découvertes
- Pas de tests intrusifs (pas de scan actif, pas d'envoi de requêtes)
- Pas de contournement des mécanismes de sécurité
- Pas de ciblage d'applications non autorisées
- Toute donnée sensible découverte doit être masquée dans les rapports
- Aucune action pouvant endommager, perturber ou surcharger une cible n'est autorisée

## Périmètre technique autorisé
- Analyse statique de l'APK fourni (lecture du code, des ressources, du manifeste)
- Utilisation de BeVigil en mode lecture seule (consultation des résultats)
- Utilisation de Yaazhini en mode analyse statique (pas de test dynamique)
- Rédaction de notes et rapports à partir des résultats des outils

## Périmètre technique INTERDIT
- Tests d'intrusion ou d'exploitation
- Scan de ports ou d'infrastructure
- Requêtes vers les endpoints identifiés
- Accès à des données réelles d'utilisateurs

## Période d'analyse
Date de début: 2026-05-27
Durée prévue: 2 heures
Date de fin prévue: 2026-05-27
Analyste: Etudiant — Cours Sécurité Mobile

## Engagement de conformité
L'analyste s'engage à respecter strictement le présent périmètre
et à signaler immédiatement toute découverte sortant du cadre prévu.
