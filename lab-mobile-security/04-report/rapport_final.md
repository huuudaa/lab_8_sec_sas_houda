# Rapport d'analyse de sécurité mobile
**LAB 8 — BeVigil & Yaazhini — Usage strictement pédagogique**

---

## A. Informations générales

| Champ | Valeur |
|-------|--------|
| **Date d'analyse** | 27 mai 2026 |
| **Analyste** | Etudiant |
| **Cible** | EduVulnApp — Application Android pédagogique |
| **Package** | com.example.eduvulnapp |
| **Version** | 1.0 (build 1) |
| **Hash SHA-256** | `7B2C4F9E1A3D5B8C0E2F4A6D8B0C2E4F6A8D0B2C4E6F8A0D2B4C6E8F0A2D4B6` |
| **Outils utilisés** | BeVigil v2.1.0 + Yaazhini v1.3.2 |
| **Périmètre** | APK pédagogique fourni par l'enseignant — TP LAB 8 (réf. SEC-LAB-2026-08) |
| **Score BeVigil** | 67/100 — Risque modéré à élevé |
| **Score Yaazhini** | 38/100 — Risque élevé |

---

## B. Résumé exécutif

L'analyse combinée de l'application **EduVulnApp** avec BeVigil (analyse OSINT externe) et Yaazhini (analyse statique interne) a mis en évidence **12 constats de sécurité**, dont **5 de sévérité High** et **4 de sévérité Medium**. Les problèmes identifiés se concentrent sur trois axes : la gestion insécurisée des secrets d'application (clé API et token JWT hardcodés dans le code), la faiblesse des communications réseau (5 endpoints HTTP non chiffrés dont l'endpoint de connexion), et la configuration risquée de la plateforme Android (mode debug activé, backup non restreint, composant de connexion exporté sans protection). Un constat (FIND-002) est corroboré par les deux outils, renforçant sa certitude. Le niveau de risque global est évalué à **Élevé** — une remédiation prioritaire des 5 constats High est impérative avant tout déploiement.

---

## C. Top 5 constats

### 1. Clé API Google Maps hardcodée — FIND-001

| | |
|--|--|
| **Sévérité** | 🔴 High |
| **Source** | Yaazhini — règle "Hardcoded API Key" |
| **Preuve** | `com.example.eduvulnapp.network.ApiClient`, ligne 52 |
| **Référence OWASP** | MASVS-STORAGE-2 |

**Impact :** La clé API est accessible à quiconque décompile l'APK avec des outils publics (apktool, jadx) en moins de 2 minutes sans compétence particulière. Elle peut être utilisée pour des requêtes frauduleuses au nom de l'application, générant des coûts excessifs ou exposant les données cartographiques associées.

**Remédiation :** Supprimer la clé du code source immédiatement. Pour les clés Google Maps Android, restreindre leur usage à l'empreinte SHA-1 du certificat de signature via la console Google Cloud. Pour les services côté serveur, utiliser des variables d'environnement et ne jamais embarquer de secrets dans l'APK.

---

### 2. Communication HTTP non chiffrée — FIND-002

| | |
|--|--|
| **Sévérité** | 🔴 High |
| **Source** | BeVigil + Yaazhini (constat corroboré) |
| **Preuve** | `AndroidManifest.xml` `android:usesCleartextTraffic="true"` + 5 endpoints HTTP (BeVigil) dont `http://api.eduvulnapp.example.com/v1/login` |
| **Référence OWASP** | MASVS-NETWORK-1 |

**Impact :** Les identifiants de connexion, données de profil utilisateur et exports de données sont transmis sans chiffrement. Sur un réseau Wi-Fi partagé (café, campus, hôtel), un attaquant peut les capturer avec Wireshark en quelques secondes sans nécessiter de compétences avancées.

**Remédiation :** Supprimer `android:usesCleartextTraffic="true"` du manifeste. Configurer `res/xml/network_security_config.xml` avec `cleartextTrafficPermitted="false"` en base-config. Migrer l'intégralité des endpoints v1 vers HTTPS ou les désactiver au profit de la v2.

---

### 3. Application déboguable en production — FIND-003

| | |
|--|--|
| **Sévérité** | 🔴 High |
| **Source** | Yaazhini — règle "Application is Debuggable" |
| **Preuve** | `AndroidManifest.xml` — `android:debuggable="true"` dans l'élément `<application>` |
| **Référence OWASP** | MASVS-RESILIENCE-2 |

**Impact :** Un attaquant disposant d'un accès ADB à l'appareil (physique ou réseau) peut attacher un débogueur à l'application sans nécessiter de root. Cela permet l'inspection de la mémoire en temps réel, l'extraction de clés de chiffrement et le contournement de tous les mécanismes de sécurité applicatifs.

**Remédiation :** Retirer l'attribut `android:debuggable` du manifeste (valeur par défaut : `false`). Configurer explicitement `debuggable false` dans le bloc `buildTypes { release { ... } }` du fichier `build.gradle` et vérifier avec `aapt dump badging app.apk | grep debuggable` avant toute livraison.

---

### 4. LoginActivity exportée sans protection — FIND-006

| | |
|--|--|
| **Sévérité** | 🔴 High |
| **Source** | Yaazhini — règle "Exported Activity without Permission" |
| **Preuve** | `AndroidManifest.xml` — `<activity android:name=".LoginActivity" android:exported="true" />` sans `android:permission` |
| **Référence OWASP** | MASVS-PLATFORM-2 |

**Impact :** N'importe quelle application installée sur l'appareil peut invoquer directement l'écran de connexion via un Intent explicite. Combiné avec des techniques d'overlay phishing, cela peut conduire à la capture des identifiants utilisateur à leur insu. Peut également permettre l'exploitation directe de vulnérabilités de la LoginActivity via des Intents malformés.

**Remédiation :** Définir `android:exported="false"` sur `LoginActivity` si elle n'est pas destinée à être invoquée par des applications tierces. Si des deep links vers l'écran de connexion sont requis, protéger le composant avec une permission personnalisée de niveau `signature` ou `signatureOrSystem`.

---

### 5. Token JWT hardcodé dans les ressources — FIND-007

| | |
|--|--|
| **Sévérité** | 🔴 High |
| **Source** | Yaazhini — règle "Hardcoded JWT Token" |
| **Preuve** | `res/values/strings.xml` — attribut `secret_token` (valeur masquée) |
| **Référence OWASP** | MASVS-STORAGE-2 |

**Impact :** Si ce token est valide côté serveur (absence d'expiration configurée ou expiration longue), il permet une authentification directe aux APIs backend sans connaître les identifiants utilisateur. La date d'expiration du token détermine la durée de la fenêtre d'exploitation — potentiellement illimitée si le token n'expire pas.

**Remédiation :** Supprimer immédiatement ce token de `strings.xml` et de tout fichier de ressources. Le révoquer côté serveur en invalidant sa signature ou en l'ajoutant à une liste de révocation (blocklist). Mettre en place un processus de revue de code automatisé (git-secrets, truffleHog) pour détecter les secrets avant tout commit.

---

## D. Faux positifs notables

| # | Alerte | Source | Justification du classement |
|---|--------|--------|---------------------------|
| FP-01 | Firebase Analytics — violation RGPD | BeVigil | Signal indicatif uniquement — la conformité RGPD dépend de l'implémentation effective du consentement et de la politique de confidentialité. Classé "À confirmer" car nécessite une analyse juridique et contextuelle hors du périmètre technique de ce lab. |
| FP-02 | Infrastructure legacy active | BeVigil | Le domaine `legacy.eduvulnapp.example.com` est détecté mais son état réel (actif / inactif / redirigé) ne peut pas être confirmé sans test de connectivité — hors périmètre autorisé (pas de test intrusif). Classé "À confirmer". |
| FP-03 | Retrofit 3.12.1 non à jour | BeVigil | Aucun CVE actif documenté sur cette version au moment de l'analyse (vérification NVD). Représente une bonne pratique non respectée (mise à jour des dépendances) mais pas une vulnérabilité confirmée. Classé Low/Info. |

---

## E. Recommandations prioritaires

**1. Auditer et révoquer tous les secrets exposés**  
Identifier immédiatement la clé API Google Maps (FIND-001) et le token JWT (FIND-007) dans les systèmes de gestion backend et les révoquer ou faire pivoter. Mettre en place un scanner automatique de secrets dans la pipeline CI/CD (ex: truffleHog, git-secrets, Gitleaks) pour prévenir toute nouvelle exposition. Délai recommandé : **immédiat**.

**2. Forcer HTTPS sur la totalité des communications réseau**  
Migrer les 5 endpoints HTTP vers HTTPS, désactiver `android:usesCleartextTraffic` dans le manifeste, et configurer `network_security_config.xml` avec `cleartextTrafficPermitted="false"`. Priorité absolue pour l'endpoint `/v1/login` qui expose des identifiants. Envisager la mise en place de certificate pinning pour les endpoints sensibles. Délai recommandé : **avant tout déploiement**.

**3. Sécuriser le processus de build et les configurations Android**  
Retirer `android:debuggable="true"` et `android:allowBackup="true"` du manifeste, définir `android:exported="false"` sur les composants non destinés à être partagés (`LoginActivity`, `UserDataProvider`), et mettre en place un contrôle automatisé de sécurité dans le pipeline de build (MobSF en mode CI, lint rules MASVS). Déclarer LeakCanary uniquement en `debugImplementation`. Délai recommandé : **avant prochaine release**.

---

## F. Annexes

| Livrable | Chemin |
|----------|--------|
| Export BeVigil (JSON) | `01-bevigil/bevigil_export.json` |
| Notes d'analyse BeVigil | `01-bevigil/bevigil_notes.md` |
| Rapport Yaazhini (HTML) | `02-yaazhini/yaazhini_report_20260527.html` |
| Notes d'analyse Yaazhini | `02-yaazhini/yaazhini_notes.md` |
| Triage consolidé (CSV) | `03-triage/triage.csv` |
| Mapping OWASP MASVS | `03-triage/owasp_mapping.md` |
| Informations d'analyse | `analyse_info.txt` |
| Journal des commandes | `commands.log` |
| Périmètre d'analyse | `00-scope/scope.md` |
| Checklist de clôture | `checklist_fin.md` |

---

*Rapport produit dans le cadre d'un exercice pédagogique — LAB 8 Sécurité Mobile — données et résultats fictifs — usage strictement académique.*  
*Analyste : Etudiant — 2026-05-27*
