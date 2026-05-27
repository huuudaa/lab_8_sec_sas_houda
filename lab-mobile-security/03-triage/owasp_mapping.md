# Mapping OWASP MASVS — LAB 8
**Cible :** EduVulnApp v1.0  
**Référentiel :** OWASP MASVS v2.0 + MASTG  
**Date :** 2026-05-27  

---

## FIND-001 : Clé API Google Maps hardcodée

- **Catégorie OWASP :** MASVS-STORAGE
- **Référence spécifique :** MASVS-STORAGE-2 — *"L'application ne doit pas stocker de données sensibles en dehors du stockage sécurisé de l'application."*
- **Test MASTG associé :** MASTG-TEST-0001 — Testing Local Data Storage for Sensitive Data
- **Justification :** Une clé API est une donnée d'authentification sensible. La stocker dans `ApiClient.java` la rend accessible à quiconque décompile l'APK, violant directement le principe de confidentialité des credentials d'application défini dans MASVS-STORAGE-2. L'objectif du contrôle est précisément d'éviter que des secrets puissent être extraits de l'application.
- **Niveau de conformité MASVS :** ❌ Non conforme

---

## FIND-002 : Communication HTTP non chiffrée

- **Catégorie OWASP :** MASVS-NETWORK
- **Référence spécifique :** MASVS-NETWORK-1 — *"L'application doit utiliser TLS pour toutes les communications réseau."*
- **Test MASTG associé :** MASTG-TEST-0019 — Testing Network Communication
- **Justification :** Les 5 endpoints HTTP identifiés transmettent des données (dont des identifiants via `/v1/login`) sans chiffrement. La configuration `android:usesCleartextTraffic="true"` désactive activement la protection par défaut d'Android 9+ — violation directe de MASVS-NETWORK-1 qui exige TLS pour toutes les communications sans exception. La corroboration BeVigil+Yaazhini renforce la certitude de ce constat.
- **Niveau de conformité MASVS :** ❌ Non conforme

---

## FIND-003 : Application déboguable en production

- **Catégorie OWASP :** MASVS-RESILIENCE
- **Référence spécifique :** MASVS-RESILIENCE-2 — *"L'application doit implémenter des mécanismes de prévention du débogage et répondre aux tentatives de débogage."*
- **Test MASTG associé :** MASTG-TEST-0039 — Testing Anti-Debugging Detection
- **Justification :** L'attribut `android:debuggable="true"` est explicitement listé comme vecteur d'attaque dans le MASTG. Il compromet tous les autres contrôles de sécurité car un attaquant peut inspecter et modifier la mémoire en temps réel, extrayant des clés de chiffrement et contournant les vérifications d'intégrité. MASVS-RESILIENCE-2 interdit explicitement cet état en production.
- **Niveau de conformité MASVS :** ❌ Non conforme

---

## FIND-004 : Backup automatique non désactivé

- **Catégorie OWASP :** MASVS-STORAGE
- **Référence spécifique :** MASVS-STORAGE-1 — *"L'application doit stocker les données sensibles correctement en utilisant les mécanismes de stockage sécurisé de la plateforme."*
- **Test MASTG associé :** MASTG-TEST-0002 — Testing for Sensitive Data in Backups
- **Justification :** `android:allowBackup="true"` expose les données applicatives (base de données, SharedPreferences) à une extraction non autorisée via `adb backup`. Les données ne peuvent pas être considérées comme "stockées correctement" si elles peuvent être extraites sans authentification par n'importe quel utilisateur ayant accès USB à l'appareil. MASVS-STORAGE-1 exige que le stockage soit protégé contre les accès non autorisés.
- **Niveau de conformité MASVS :** ⚠️ Partiellement non conforme (SQLCipher utilisé mais backup non restreint)

---

## FIND-005 : Permissions dangereuses excessives

- **Catégorie OWASP :** MASVS-PLATFORM
- **Référence spécifique :** MASVS-PLATFORM-1 — *"L'application ne doit demander que les permissions Android minimales nécessaires."*
- **Test MASTG associé :** MASTG-TEST-0022 — Testing for App Permissions
- **Justification :** Les permissions `READ_SMS` et `READ_CONTACTS` n'ont aucune justification fonctionnelle visible dans l'application pédagogique analysée. Leur présence constitue une violation directe du principe de moindre privilège tel que défini dans MASVS-PLATFORM-1, qui exige que chaque permission soit nécessaire ET justifiée par la fonctionnalité de l'application.
- **Niveau de conformité MASVS :** ❌ Non conforme

---

## FIND-006 : LoginActivity exportée sans protection

- **Catégorie OWASP :** MASVS-PLATFORM
- **Référence spécifique :** MASVS-PLATFORM-2 — *"L'application doit restreindre l'accès aux composants exportés à l'aide de permissions Android personnalisées."*
- **Test MASTG associé :** MASTG-TEST-0024 — Testing for Exposed IPC Components
- **Justification :** L'exportation d'une Activity de connexion sans permission constitue une interface IPC non protégée exploitable par des applications tierces malveillantes. MASVS-PLATFORM-2 requiert explicitement que les composants exportés soient protégés par des permissions appropriées, notamment pour les composants sensibles comme un écran de connexion.
- **Niveau de conformité MASVS :** ❌ Non conforme

---

## FIND-007 : Token JWT hardcodé dans les ressources

- **Catégorie OWASP :** MASVS-STORAGE
- **Référence spécifique :** MASVS-STORAGE-2 — *"L'application ne doit pas stocker de données sensibles en dehors du stockage sécurisé de l'application."*
- **Test MASTG associé :** MASTG-TEST-0001 — Testing Local Data Storage for Sensitive Data
- **Justification :** Un token JWT est un credential d'authentification. Sa présence dans `strings.xml` viole le même principe que pour la clé API (FIND-001), avec un impact potentiellement plus grave : un JWT valide permet une authentification directe aux APIs sans connaître les identifiants utilisateur. MASVS-STORAGE-2 couvre explicitement les tokens d'authentification comme données sensibles à protéger.
- **Niveau de conformité MASVS :** ❌ Non conforme

---

## Tableau de synthèse MASVS

| ID | Vulnérabilité | Catégorie | Référence MASVS | Test MASTG | Sévérité | Conformité |
|----|---------------|-----------|-----------------|------------|----------|------------|
| FIND-001 | Clé API hardcodée | MASVS-STORAGE | MASVS-STORAGE-2 | MASTG-TEST-0001 | High | ❌ |
| FIND-002 | Communication HTTP non chiffrée | MASVS-NETWORK | MASVS-NETWORK-1 | MASTG-TEST-0019 | High | ❌ |
| FIND-003 | Application déboguable | MASVS-RESILIENCE | MASVS-RESILIENCE-2 | MASTG-TEST-0039 | High | ❌ |
| FIND-004 | Backup non désactivé | MASVS-STORAGE | MASVS-STORAGE-1 | MASTG-TEST-0002 | Medium | ⚠️ |
| FIND-005 | Permissions excessives | MASVS-PLATFORM | MASVS-PLATFORM-1 | MASTG-TEST-0022 | Medium | ❌ |
| FIND-006 | LoginActivity exportée | MASVS-PLATFORM | MASVS-PLATFORM-2 | MASTG-TEST-0024 | High | ❌ |
| FIND-007 | Token JWT hardcodé | MASVS-STORAGE | MASVS-STORAGE-2 | MASTG-TEST-0001 | High | ❌ |

### Répartition par catégorie MASVS

| Catégorie | Constats | Observations |
|-----------|----------|-------------|
| **MASVS-STORAGE** | 3 (FIND-001, 004, 007) | Catégorie la plus impactée — gestion des secrets défaillante |
| **MASVS-PLATFORM** | 2 (FIND-005, 006) | Contrôle d'accès aux composants à renforcer |
| **MASVS-NETWORK** | 1 (FIND-002) | Communications non sécurisées |
| **MASVS-RESILIENCE** | 1 (FIND-003) | Absence de protection contre le débogage |

> **Note :** Les mappings OWASP ont été mis à jour dans `triage.csv` (colonne "Référence OWASP") pour les 7 constats ci-dessus.
