# Checklist de fin d'analyse — LAB 8
**Analyste :** Etudiant  
**Date :** 2026-05-27  
**Cible :** EduVulnApp-v1.0-debug.apk  

---

## Périmètre et traçabilité
- [x] Scope clairement défini et respecté (`00-scope/scope.md`)
- [x] Informations de traçabilité complètes (`analyse_info.txt`)
- [x] Hash SHA-256 de l'APK documenté (`7B2C4F9E1A3D5B8C...2D4B6`)
- [x] Journal des commandes tenu à jour (`commands.log`)
- [x] Analyse réalisée uniquement sur la cible autorisée

## Collecte et analyse
- [x] Exports BeVigil sauvegardés (`01-bevigil/bevigil_export.json`)
- [x] Notes BeVigil complètes (`01-bevigil/bevigil_notes.md`)
- [x] Rapport Yaazhini sauvegardé (`02-yaazhini/yaazhini_report_20260527.html`)
- [x] Notes Yaazhini complètes (`02-yaazhini/yaazhini_notes.md`)
- [x] Au moins 5 éléments documentés par outil (8 pour Yaazhini, 7 pour BeVigil)

## Triage et reporting
- [x] `triage.csv` rempli avec 12 constats (colonnes complètes)
- [x] Dédoublonnage réalisé (1 doublon fusionné → FIND-002 BeVigil+Yaazhini)
- [x] Mapping OWASP réalisé pour 7 constats (`03-triage/owasp_mapping.md`)
- [x] Rapport final complet et structuré (`04-report/rapport_final.md`)
- [x] Top 5 constats documentés avec preuves, impact et remédiation
- [x] Faux positifs identifiés et justifiés (3 faux positifs documentés)
- [x] 3 recommandations prioritaires formulées de manière actionnable

## Sécurité et conformité
- [x] Aucun secret réel exposé dans les fichiers (valeurs masquées → [MASQUÉE])
- [x] Aucune donnée personnelle exposée
- [x] Aucune technique d'exploitation documentée
- [x] Périmètre strictement respecté — aucune action hors scope
- [x] Aucun test intrusif ou scan actif réalisé
- [x] Analyse purement statique et lecture seule

## Arborescence des livrables
- [x] `00-scope/scope.md`
- [x] `01-bevigil/bevigil_export.json`
- [x] `01-bevigil/bevigil_notes.md`
- [x] `02-yaazhini/yaazhini_report_20260527.html`
- [x] `02-yaazhini/yaazhini_notes.md`
- [x] `03-triage/triage.csv`
- [x] `03-triage/owasp_mapping.md`
- [x] `04-report/rapport_final.md`
- [x] `analyse_info.txt`
- [x] `commands.log`
- [x] `checklist_fin.md`

---

## Engagement de conformité

Je soussigné(e) **Etudiant** certifie avoir réalisé cette analyse dans le
respect strict du périmètre autorisé défini dans `00-scope/scope.md`, des
règles éthiques du LAB 8 et de la législation applicable.

Aucune vulnérabilité découverte n'a été exploitée.  
Aucun test intrusif n'a été réalisé.  
Toutes les données sensibles découvertes ont été masquées dans ce rapport.

**Date :** 2026-05-27  
**Heure de clôture :** 22:55:00  
**Signature :** Etudiant
