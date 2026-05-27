# Notes d'analyse BeVigil
**Outil :** BeVigil v2.1.0  
**Projet :** LAB-BEGINNER-20260527  
**Cible :** EduVulnApp-v1.0-debug.apk  
**Date :** 2026-05-27  

---

## Ce qui est certain
*(éléments disposant d'une preuve directe dans l'export BeVigil)*

- L'application cible le domaine principal `api.eduvulnapp.example.com` (référencé dans 6 endpoints distincts)
- Trois adresses email sont présentes dans les métadonnées et le code de l'APK
- La bibliothèque **Retrofit 3.12.1** est intégrée à l'application (client HTTP Android)
- **5 endpoints HTTP** (non chiffrés) ont été détectés, dont l'endpoint de login
- Le **SDK Firebase Analytics v21.0.0** est intégré à l'application
- Un domaine de développement `dev.eduvulnapp.example.com` est actif et expose un endpoint `/debug/logs`
- **LeakCanary 2.10** (outil de debug) est détecté dans le binaire de l'application

## Ce qui est hypothèse
*(interprétations des signaux — nécessitent confirmation)*

- La présence du domaine `legacy.eduvulnapp.example.com` **suggère** une infrastructure ancienne potentiellement non maintenue — à confirmer par test de connectivité (hors périmètre du lab)
- L'email `dev-intern@eduvulnapp.example.com` **pourrait** indiquer des accès de développeurs temporaires non révoqués — à vérifier auprès de l'équipe
- La version de Retrofit (3.12.1) **n'est pas la plus récente** — des CVE pourraient exister sur cette version mineure, à vérifier sur NVD
- Firebase Analytics **pourrait** poser des problèmes RGPD selon l'implémentation du consentement — analyse juridique nécessaire

## Points d'intérêt
*(éléments méritant une investigation approfondie)*

- L'endpoint `http://dev.eduvulnapp.example.com/debug/logs` est accessible publiquement : en environnement réel, ce type d'endpoint exposerait des journaux d'exécution contenant potentiellement des données utilisateur, des traces de pile et des informations d'infrastructure.
- La coexistence d'une API v1 (HTTP) et d'une API v2 (HTTPS) suggère une migration en cours — l'ancienne version HTTP est probablement maintenue pour compatibilité descendante mais représente un risque.
- La présence de Firebase Analytics soulève des questions RGPD sur la collecte et le transfert de données vers des serveurs de Google sans consentement explicite documenté.

---

## Domaines et sous-domaines

| Domaine | Type | Risque | Observation |
|---------|------|--------|-------------|
| `api.eduvulnapp.example.com` | API principale | 🔴 Élevé | Backend central — endpoints HTTP non chiffrés |
| `legacy.eduvulnapp.example.com` | Infrastructure legacy | 🟠 Moyen | Ancienne infrastructure potentiellement non maintenue |
| `cdn.eduvulnapp.example.com` | CDN | 🟢 Faible | Ressources statiques publiques — risque faible |
| `dev.eduvulnapp.example.com` | Développement | 🔴 Élevé | Env. de dev exposé publiquement avec endpoint debug |

---

## Endpoints et APIs

| Endpoint | Protocole | Risque | Observation |
|----------|-----------|--------|-------------|
| `http://api.eduvulnapp.example.com/v1/login` | HTTP | 🔴 Critique | Identifiants transmis en clair |
| `http://api.eduvulnapp.example.com/v1/user/profile` | HTTP | 🔴 Élevé | Données personnelles exposées |
| `http://api.eduvulnapp.example.com/v1/data/export` | HTTP | 🔴 Élevé | Export de données potentiellement sensibles |
| `https://api.eduvulnapp.example.com/v2/auth` | HTTPS | 🟢 Faible | Version sécurisée — correcte |
| `http://legacy.eduvulnapp.example.com/sync` | HTTP | 🔴 Élevé | Endpoint legacy non chiffré |
| `http://dev.eduvulnapp.example.com/debug/logs` | HTTP | 🔴 Critique | Logs système exposés publiquement |
| `https://cdn.eduvulnapp.example.com/assets/images/` | HTTPS | 🟢 Faible | Ressources statiques — OK |
| `https://www.google-analytics.com/collect` | HTTPS | 🟠 Moyen | Tracking Firebase — questions RGPD |

---

## URLs HTTP/HTTPS

```
[HTTP - NON CHIFFRÉ]
http://api.eduvulnapp.example.com/v1/login
http://api.eduvulnapp.example.com/v1/user/profile
http://api.eduvulnapp.example.com/v1/data/export
http://legacy.eduvulnapp.example.com/sync
http://dev.eduvulnapp.example.com/debug/logs

[HTTPS - CHIFFRÉ]
https://api.eduvulnapp.example.com/v2/auth
https://cdn.eduvulnapp.example.com/assets/images/
https://www.google-analytics.com/collect
```

**Ratio HTTP/HTTPS :** 5 URLs non chiffrées pour 3 sécurisées → situation préoccupante

---

## Emails et identifiants

| Email | Source | Risque | Observation |
|-------|--------|--------|-------------|
| `admin@eduvulnapp.example.com` | `AndroidManifest.xml` | 🟠 Moyen | Contact admin exposé dans le manifeste |
| `dev-intern@eduvulnapp.example.com` | Code source | 🔴 Élevé | Email de stagiaire — potentiel compte temporaire |
| `support@eduvulnapp.example.com` | Métadonnées APK | 🟢 Faible | Contact public volontaire — acceptable |

---

## Technologies détectées

| Bibliothèque | Version | Statut | Observation |
|-------------|---------|--------|-------------|
| Retrofit | 3.12.1 | 🟠 Attention | Pas la dernière version — vérifier CVE |
| OkHttp | 4.10.0 | 🟢 OK | Version récente |
| Firebase Analytics | 21.0.0 | 🟠 Attention | Implications RGPD à vérifier |
| Gson | 2.10.1 | 🟢 OK | Version récente |
| Android SDK | 33 / 21 | 🟢 OK | Support Android 5.0+ |
| Glide | 4.14.2 | 🟢 OK | Version récente |
| SQLCipher | 4.5.3 | 🟢 Bonne pratique | Base de données chiffrée |
| **LeakCanary** | **2.10** | 🔴 **ALERTE** | **Outil de debug présent en production** |

---

## Synthèse BeVigil

- **Score de risque global BeVigil :** 67/100 (risque modéré à élevé)
- **Principal problème identifié :** 5 endpoints HTTP non chiffrés dont le login
- **Signal le plus préoccupant :** Environnement de développement exposé publiquement (`dev.eduvulnapp.example.com`)
- **Bonne pratique identifiée :** SQLCipher utilisé pour le chiffrement de la base de données locale
