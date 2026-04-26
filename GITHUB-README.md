# ConverterTabsJava - Analyse de Sécurité APK

**Analyste:** Fatima Ezzahra El Boudhiri  
**Établissement:** ENSA  
**Classe:** Sécurité des Applications Multimedia  
**Laboratoire:** Lab 4  
**Professeur:** M. Lachgar  
**Date:** 26 Avril 2026

---

## Vue d'ensemble

Ce repository contient une analyse de sécurité statique complète de l'application Android **ConverterTabsJava**. L'analyse a identifié **3 vulnérabilités** de sécurité nécessitant une remédiation avant la mise en production.

---

## Résumé des Résultats

| Métrique | Valeur |
|----------|--------|
| **Score de Sécurité** | 5/10 |
| **Niveau de Risque** | ÉLEVÉ |
| **Vulnérabilités Critiques** | 1 |
| **Vulnérabilités Élevées** | 1 |
| **Vulnérabilités Moyennes** | 1 |
| **Secrets en Dur Trouvés** | 0 |
| **Permissions Dangereuses** | 0 |
| **Statut** | NON PRÊTE POUR LA PRODUCTION |

---

## Vulnérabilités Identifiées

### 1. Mode Débogage Activé (CRITIQUE)
- **Fichier:** AndroidManifest.xml (ligne 35)
- **Issue:** `android:debuggable="true"`
- **Impact:** Permet le débogage et l'accès complet à l'application
- **Remédiation:** Changer à `android:debuggable="false"`

### 2. Sauvegarde Complète Autorisée (ÉLEVÉE)
- **Fichier:** AndroidManifest.xml (lignes 36-38)
- **Issue:** `android:allowBackup="true"`
- **Impact:** Exposition de toutes les données de l'application
- **Remédiation:** Changer à `android:allowBackup="false"`

### 3. Composant Récepteur Exposé (MOYENNE)
- **Fichier:** AndroidManifest.xml (lignes 57-65)
- **Issue:** ProfileInstallReceiver avec `android:exported="true"`
- **Impact:** Accessible par d'autres applications
- **Remédiation:** Changer à `android:exported="false"`

---

## Fichiers dans ce Repository

```
.
├── README.md                      # Ce fichier
├── RAPPORT.md                     # Rapport principal d'audit
├── PERMISSIONS.md                 # Analyse des permissions
├── FINDINGS.md                    # Constats détaillés
└── APK/
    ├── app-debug.apk             # Application analysée
    └── hash-sha256.txt           # Empreinte de l'APK
```

---

## Comment Utiliser ce Repository

1. **Lire le rapport principal:** [RAPPORT.md](README.md)
2. **Consulter les permissions:** [PERMISSIONS.md](PERMISSIONS.md)
3. **Voir les détails techniques:** [FINDINGS.md](FINDINGS.md)

---

## Points Positifs

- ✓ Aucun secret (API key, token, password) en dur
- ✓ Pas de permissions dangereuses demandées
- ✓ Code source propre et bien structuré
- ✓ Versions SDK modernes (Min: 24, Target: 36)

---

## Remédiations Recommandées

### Immédiat (avant tout déploiement):
1. Désactiver le mode débogage
2. Désactiver les sauvegardes complètes
3. Désexposer le récepteur ProfileInstallReceiver

**Temps estimé:** 20-30 minutes

---

## Outils Utilisés

- **JADX GUI v1.5.5** - Décompilation et analyse du code
- **dex2jar v2.4** - Conversion DEX vers JAR
- **Android Manifest Analysis** - Évaluation des configurations

---

## Métadonnées de l'APK

- **Nom:** ConverterTabsJava
- **Package:** com.example.convertertabsjava
- **Version:** 1.0
- **SHA-256:** 0b06f14786d8ffde514c53cb1bf70dae56256338af19393dd5afa1839999e5b7

---

## Recommandation Finale

**REJETER pour la production.** L'application présente 3 problèmes de sécurité critiques qui doivent être corrigés avant tout déploiement en production.

---

## Contact

**Analyste:** Fatima Ezzahra El Boudhiri  
**Institution:** ENSA - Sécurité des Applications Multimedia  
**Date de l'Analyse:** 26 Avril 2026

---

## License

Ce rapport est confidentiel et destiné à des fins éducatives et d'amélioration de la sécurité.

---

*Généré automatiquement avec JADX GUI v1.5.5*
