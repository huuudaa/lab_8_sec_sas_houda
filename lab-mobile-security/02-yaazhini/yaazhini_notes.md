# Notes d'analyse Yaazhini
**Outil :** Yaazhini v1.3.2  
**Cible :** EduVulnApp-v1.0-debug.apk  
**Date :** 2026-05-27  
**Durée d'analyse :** 3 min 23 sec  
**Rapport généré :** `yaazhini_report_20260527.html`  

---

## Éléments identifiés

### Élément 1 : Clé API hardcodée dans le code source
- **Localisation :** `com.example.eduvulnapp.network.ApiClient` — ligne 52  
  Section "Secrets & Keys" du rapport Yaazhini — règle déclenchée : *"Hardcoded API Key"*
- **Description :** Une clé API de service cartographique est inscrite en clair directement dans le code Java. Le pattern `MAPS_API_KEY = "AIzaSyD-[MASQUÉ]"` a été détecté par l'analyse statique. La valeur réelle est masquée dans ce document.
- **Impact potentiel :** Toute personne décompilant l'APK avec des outils publics (apktool, jadx) récupère cette clé immédiatement, sans compétence particulière. Elle peut être utilisée pour des requêtes frauduleuses au nom de l'application (quotas épuisés, facturation, accès à des données liées à la clé).
- **Remédiation suggérée :** Ne jamais stocker de clés API dans le code source. Pour les clés Android côté client (ex: Google Maps), les restreindre à l'empreinte SHA-1 du certificat de signature via la console Google Cloud. Pour les services côté serveur, utiliser des variables d'environnement côté backend.

---

### Élément 2 : Communication en clair autorisée (`usesCleartextTraffic`)
- **Localisation :** `AndroidManifest.xml` — élément `<application>` — attribut `android:usesCleartextTraffic="true"`  
  Section "Manifest Analysis" du rapport Yaazhini — règle : *"Cleartext Traffic Permitted"*
- **Description :** Le manifeste autorise explicitement les connexions HTTP non chiffrées. Ce paramètre remplace la politique par défaut d'Android 9+ (API 28+) qui bloque le trafic en clair. Corroboré par 5 endpoints HTTP identifiés par BeVigil.
- **Impact potentiel :** Toute donnée transitant par les endpoints HTTP (login, profil, export) est lisible sur le réseau. Un attaquant positionné sur le même réseau Wi-Fi peut capturer et modifier ces données sans détection (attaque Man-in-the-Middle).
- **Remédiation suggérée :** Supprimer `android:usesCleartextTraffic="true"`. Migrer tous les endpoints vers HTTPS. Créer ou modifier `network_security_config.xml` avec `cleartextTrafficPermitted="false"` en base-config.

---

### Élément 3 : Mode debug activé en production (`android:debuggable`)
- **Localisation :** `AndroidManifest.xml` — élément `<application>` — attribut `android:debuggable="true"`  
  Section "Manifest Analysis" — règle : *"Application is Debuggable"*
- **Description :** L'application est compilée avec le flag de débogage explicitement forcé à `true` dans le manifeste. Ce flag devrait être `false` (valeur par défaut) dans tout build de production.
- **Impact potentiel :** N'importe quel utilisateur ayant un accès ADB à l'appareil peut attacher un débogueur à l'application en cours d'exécution, même sur un appareil non rooté. Cela permet l'inspection de la mémoire, l'extraction de clés de chiffrement et le contournement des contrôles de sécurité.
- **Remédiation suggérée :** Retirer l'attribut `android:debuggable` du manifeste. Configurer `debuggable false` dans le bloc `buildTypes { release { ... } }` du fichier `build.gradle`. La valeur par défaut est `false` — ne pas la surcharger.

---

### Élément 4 : Sauvegarde automatique non désactivée (`allowBackup`)
- **Localisation :** `AndroidManifest.xml` — élément `<application>` — attribut `android:allowBackup="true"`  
  Section "Manifest Analysis" — règle : *"Backup Allowed"*
- **Description :** L'attribut `allowBackup` permet à Android et à `adb backup` d'extraire l'intégralité des données de l'application sans nécessiter un accès root sur les versions d'Android antérieures à Android 12.
- **Impact potentiel :** Un attaquant ayant un accès physique à l'appareil et le mode débogage USB activé peut extraire toute la base de données locale, les SharedPreferences et les fichiers de l'application. Pour une application contenant des données de session ou des informations personnelles, cela constitue une fuite de données potentiellement grave.
- **Remédiation suggérée :** Définir `android:allowBackup="false"` dans le manifeste si aucune fonctionnalité de sauvegarde n'est requise. Si la sauvegarde est nécessaire, implémenter `BackupAgent` pour contrôler précisément quelles données sont incluses.

---

### Élément 5 : Permissions dangereuses excessives
- **Localisation :** `AndroidManifest.xml` — section `<uses-permission>`  
  Section "Permissions" du rapport Yaazhini — 7 permissions classées "dangereuses" par Android
- **Description :** L'application déclare 7 permissions dangereuses, dont `READ_SMS` et `READ_CONTACTS` qui ne correspondent à aucune fonctionnalité visible de l'application pédagogique analysée.
  ```
  Permissions dangereuses identifiées :
  - android.permission.READ_SMS           [NON JUSTIFIÉE]
  - android.permission.READ_CONTACTS      [NON JUSTIFIÉE]
  - android.permission.ACCESS_FINE_LOCATION
  - android.permission.CAMERA
  - android.permission.RECORD_AUDIO
  - android.permission.WRITE_EXTERNAL_STORAGE
  - android.permission.READ_PHONE_STATE
  ```
- **Impact potentiel :** Permissions excessives qui élargissent inutilement la surface d'attaque et violent le principe du moindre privilège. En cas de compromission de l'application, l'attaquant dispose d'un accès aux SMS et contacts de l'utilisateur.
- **Remédiation suggérée :** Appliquer le principe du moindre privilège. Justifier techniquement chaque permission dangereuse. Supprimer `READ_SMS` et `READ_CONTACTS` si aucune fonctionnalité ne les requiert.

---

### Élément 6 : Composant Activity exporté sans protection
- **Localisation :** `AndroidManifest.xml` — `<activity android:name=".LoginActivity" android:exported="true" />`  
  Section "Exported Components" — règle : *"Exported Activity without Permission"*
- **Description :** La `LoginActivity` est déclarée avec `exported="true"` sans aucun attribut `android:permission`. N'importe quelle application installée sur l'appareil peut invoquer directement cet écran de connexion via un Intent explicite.
- **Impact potentiel :** Une application malveillante peut lancer l'écran de connexion et superposer une interface de phishing pour capturer les identifiants saisis par l'utilisateur, ou exploiter directement des vulnérabilités de cet écran.
- **Remédiation suggérée :** Définir `android:exported="false"` sur `LoginActivity` si elle n'est pas destinée à être invoquée par des applications tierces. Si des deep links sont nécessaires, ajouter une permission `android:permission` avec niveau de protection `signature`.

---

### Élément 7 : Token JWT hardcodé dans les ressources
- **Localisation :** `res/values/strings.xml` — attribut `secret_token`  
  Section "Secrets & Keys" — règle : *"Hardcoded JWT Token"*
- **Description :** Un token JWT de test est stocké en clair dans le fichier de ressources `strings.xml`. La valeur réelle est masquée dans ce document — sa structure (`eyJhb...`) a été identifiée par la règle de détection de tokens JWT.
- **Impact potentiel :** Si ce token est valide côté serveur (absence d'expiration ou expiration longue), il peut permettre une authentification directe aux APIs backend sans connaître les identifiants utilisateur.
- **Remédiation suggérée :** Supprimer immédiatement ce token de toutes les ressources. Révoquer le token côté serveur (invalider la signature ou ajouter à une liste de révocation). Les tokens ne doivent jamais être stockés statiquement dans un APK.

---

### Élément 8 : LeakCanary intégré en production
- **Localisation :** Bibliothèques natives — `lib/arm64-v8a/libLeakCanary.so`  
  Section "Binary Analysis" — présence de la bibliothèque LeakCanary dans le build de production
- **Description :** LeakCanary est un outil de détection de fuites mémoire destiné exclusivement au développement. Sa présence indique qu'elle a été incluse avec `implementation` au lieu de `debugImplementation` dans `build.gradle`, l'incluant donc dans les builds de production.
- **Impact potentiel :** Code de monitoring inutile en production augmentant la surface d'attaque. LeakCanary enregistre des informations sur les objets en mémoire qui pourraient révéler des informations sensibles dans les traces de fuites.
- **Remédiation suggérée :** S'assurer que LeakCanary est déclaré uniquement avec `debugImplementation 'com.squareup.leakcanary:leakcanary-android:2.10'` dans `build.gradle`, jamais avec `implementation`.

---

## Synthèse Yaazhini

| # | Élément | Localisation | Sévérité |
|---|---------|-------------|----------|
| 1 | Clé API hardcodée | `ApiClient.java:52` | 🔴 High |
| 2 | Cleartext traffic autorisé | `AndroidManifest.xml` | 🔴 High |
| 3 | Mode debug activé | `AndroidManifest.xml` | 🔴 High |
| 4 | Backup non désactivé | `AndroidManifest.xml` | 🟠 Medium |
| 5 | Permissions excessives | `AndroidManifest.xml` | 🟠 Medium |
| 6 | LoginActivity exportée | `AndroidManifest.xml` | 🔴 High |
| 7 | Token JWT hardcodé | `res/values/strings.xml` | 🔴 High |
| 8 | LeakCanary en production | `lib/arm64-v8a/` | 🟡 Low |

**Aucun secret réel n'est documenté dans ce fichier — toutes les valeurs sensibles sont masquées.**
