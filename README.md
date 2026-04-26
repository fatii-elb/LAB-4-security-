# Rapport d'Analyse Statique de Sécurité

## Application: ConverterTabsJava

---

## Informations Générales

| Élément | Valeur |
|---------|--------|
| **Titre** | Analyse Statique de ConverterTabsJava |
| **Date d'analyse** | 26 Avril 2026 |
| **Analyste** | Fatima Ezzahra El Boudhiri |
| **Établissement** | ENSA |
| **Classe** | Sécurité des Applications Multimedia |
| **Laboratoire** | Lab 4 |
| **Professeur** | M. Lachgar |
| **APK analysé** | app-debug.apk |
| **Version** | 1.0 |
| **Code de version** | 1 |
| **Provenance** | Projet Android Studio personnel |
| **Outils utilisés** | JADX GUI v1.5.5, dex2jar v2.4 |
| **Hash SHA-256** | 0b06f14786d8ffde514c53cb1bf70dae56256338af19393dd5afa1839999e5b7 |

---

## Résumé Exécutif

Cette analyse statique a révélé **3 vulnérabilités potentielles** dans l'application ConverterTabsJava. Les principales préoccupations concernent:

1. **Mode débogage activé** (debuggable="true") - Permet aux attaquants de déboguer l'application en temps réel
2. **Sauvegarde complète autorisée** (allowBackup="true") - Exposition des données sensibles
3. **Composant récepteur exposé** - Accès par d'autres applications

Le **niveau de risque global** est évalué comme **ÉLEVÉ**.

### Actions Prioritaires Recommandées:

1. **IMMÉDIAT**: Désactiver le mode débogage (`android:debuggable="false"`)
2. **IMMÉDIAT**: Désactiver les sauvegardes complètes (`android:allowBackup="false"`)
3. **COURT TERME**: Désexposer le récepteur (`android:exported="false"`)

---

## Constats Détaillés

### Constat #1: Mode Débogage Activé

**Sévérité:** CRITIQUE (Score: 9/10)

**Description:** L'application a le mode débogage activé dans le manifeste Android. Cela permet à un attaquant de:
- Attacher un débogueur à l'application en cours d'exécution
- Accéder à la mémoire et toutes les variables de l'application
- Modifier le comportement de l'application en temps réel
- Contourner les mécanismes de sécurité
- Extraire des données sensibles

**Localisation:** `AndroidManifest.xml`, ligne 35

```xml
<application
    ...
    android:debuggable="true"
    ...
>
```

**Impact Potentiel:**
- Compromis complet de l'application
- Vol de données utilisateur
- Modification du code à l'exécution
- Contournement de l'authentification
- Injection de malveillance

**Remédiation Recommandée:**

Remplacer:
```xml
android:debuggable="true"
```

Par:
```xml
android:debuggable="false"
```

**Note:** Le mode débogage ne doit JAMAIS être `true` en production. C'est acceptable uniquement durant le développement.

---

### Constat #2: Sauvegarde Complète Autorisée

**Sévérité:** ÉLEVÉE (Score: 8/10)

**Description:** L'application permet la sauvegarde complète de tous ses données. Un attaquant peut:
- Sauvegarder l'application en utilisant: `adb backup -all`
- Extraire tous les fichiers de l'application (bases de données, SharedPreferences, fichiers)
- Restaurer la sauvegarde sur un autre appareil
- Accéder à tous les mots de passe, tokens, clés API stockés
- Compromettre la confidentialité des données utilisateur

**Localisation:** `AndroidManifest.xml`, lignes 36-38

```xml
<application
    ...
    android:allowBackup="true"
    android:fullBackupContent="@xml/backup_rules"
    ...
>
```

**Impact Potentiel:**
- Vol complet des données de l'application
- Exposition des credentials et tokens
- Compromis des données utilisateur
- Accès non autorisé à des informations sensibles
- Violation de la confidentialité

**Remédiation Recommandée:**

**Option 1 - Désactiver complètement les sauvegardes:**
```xml
android:allowBackup="false"
```

**Option 2 - Restreindre le contenu sauvegardé:**
Modifier `@xml/backup_rules` pour exclure les données sensibles:
```xml
<?xml version="1.0" encoding="utf-8"?>
<full-backup-content>
    <!-- Inclure uniquement les fichiers non-sensibles -->
    <include domain="sharedpref" path="." />
    <!-- Exclure les données sensibles -->
    <exclude domain="sharedpref" path="credentials.xml" />
    <exclude domain="database" path="sensitive_db.db" />
</full-backup-content>
```

---

### Constat #3: Composant Récepteur Exposé

**Sévérité:** MOYENNE (Score: 5/10)

**Description:** Le récepteur `ProfileInstallReceiver` est exposé et peut être accédé par d'autres applications. Cela permet à une application malveillante de:
- Envoyer des intentions (intents) à ce récepteur
- Déclencher un comportement inattendu
- Potentiellement manipuler l'installation de profils
- Causer une dénégation de service

**Localisation:** `AndroidManifest.xml`, lignes 57-65

```xml
<receiver
    android:name="androidx.profileinstaller.ProfileInstallReceiver"
    android:permission="android.permission.DUMP"
    android:enabled="true"
    android:exported="true"
    android:directBootAware="false">
    <intent-filter>
        <action android:name="androidx.profileinstaller.action.INSTALL_PROFILE"/>
    </intent-filter>
    ...
</receiver>
```

**Impact Potentiel:**
- Détournement de composant par une app malveillante
- Comportement inattendu de l'application
- Dénégation de service
- Manipulation de l'installation de profils

**Remédiation Recommandée:**

Changer:
```xml
android:exported="true"
```

En:
```xml
android:exported="false"
```

**Note:** À moins que ce composant ait besoin d'être accessible par d'autres applications, il doit toujours être défini comme non-exporté.

---

## Analyse des Permissions et Composants

### Permissions Demandées:

```
✓ com.example.convertertabsjava.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION
  (ProtectionLevel: signature)
```

**Évaluation:** BON
- Seulement 1 permission personnalisée
- Aucune permission dangereuse (CAMERA, CONTACTS, LOCATION, etc.)
- Surface d'attaque minimale
- Respect du principe du moindre privilège

### Composants Exportés:

| Composant | Exporté | Type | Raison | Risque |
|-----------|---------|------|--------|--------|
| MainActivity | OUI | Activity | Nécessaire (MAIN/LAUNCHER) | Bas |
| ProfileInstallReceiver | OUI | Receiver | Peut être désexposé | Moyen |
| InitializationProvider | NON | Provider | Correct | Bas |

---

## Résultats de Recherche de Secrets

### Recherches Effectuées:

| # | Terme | Résultat | Détail |
|---|-------|----------|--------|
| 1 | `http://` | NON | Pas d'URLs non chiffrées trouvées |
| 2 | `https://` | NON | Pas d'URLs chiffrées trouvées |
| 3 | `api_key` | NON | Pas de clés API en dur |
| 4 | `token` | NON | Pas de tokens en dur |
| 5 | `password` | NON | Pas de mots de passe en dur |
| 6 | `secret` | NON | Pas de secrets en dur |
| 7 | `DEBUG` | OUI | Dans AndroidManifest.xml (déjà documenté) |
| 8 | `firebase` | NON | Pas de Firebase trouvé |
| 9 | `endpoint` | NON | Pas d'endpoints en dur |

**Conclusion:** Aucun secret en dur n'a été découvert dans le code source

---

## Points Positifs

- Pas de clés API en dur
- Pas de tokens d'authentification en dur
- Pas de mots de passe en dur
- Code source propre et bien structuré
- Versions SDK modernes (Min SDK: 24, Target SDK: 36)
- Pas de permissions excessives demandées
- Pas de dépendances suspectes

---

## Évaluation des Risques Globale

**Application:** ConverterTabsJava  
**Package:** com.example.convertertabsjava  
**Version:** 1.0

### Résumé des Vulnérabilités:

- CRITIQUES: 1 (debuggable="true")
- ÉLEVÉES: 1 (allowBackup="true")
- MOYENNES: 1 (exported receiver)
- BASSES: 0

**Score de Sécurité:** 5/10

**Niveau de Risque Global:** ÉLEVÉ

**Statut:** NON PRÊTE POUR LA PRODUCTION

---

## Plan de Remédiation

### Phase 1: IMMÉDIATE (Avant tout déploiement)

1. **Désactiver le mode débogage**
   - Fichier: `AndroidManifest.xml`
   - Changer: `android:debuggable="true"` → `android:debuggable="false"`
   - Temps estimé: 5 minutes

2. **Désactiver les sauvegardes complètes**
   - Fichier: `AndroidManifest.xml`
   - Changer: `android:allowBackup="true"` → `android:allowBackup="false"`
   - Temps estimé: 5 minutes

### Phase 2: À COURT TERME (Avant la première version stable)

3. **Désexposer le récepteur ProfileInstallReceiver**
   - Fichier: `AndroidManifest.xml`
   - Changer: `android:exported="true"` → `android:exported="false"`
   - Vérifier que cette application n'a pas besoin d'être accessible par d'autres apps
   - Temps estimé: 10 minutes

### Vérification Post-Remédiation:

- [ ] Rebuildé l'APK avec les changements
- [ ] Ré-exécuter l'analyse de sécurité
- [ ] Vérifier que tous les flags sont corrects
- [ ] Effectuer des tests QA complets
- [ ] Vérifier la signature de l'APK

---

## Conclusion

L'application ConverterTabsJava est une application simple et bien structurée pour convertir les températures et distances. Cependant, elle présente **3 problèmes de configuration critiques** qui la rendent **non adaptée à la production**:

1. **Mode débogage activé** - Permet à n'importe quel attaquant de déboguer l'application
2. **Sauvegarde complète autorisée** - Expose toutes les données de l'application
3. **Composant récepteur exposé** - Permet l'accès par d'autres applications

**Points positifs:** Aucun secret en dur n'a été découvert dans le code source. L'application n'utilise pas de permissions dangereuses.

**Recommandation:** **REJETER pour la production.** Corriger les 3 vulnérabilités et ré-analyser avant tout déploiement.

**Temps estimé pour les corrections:** 20-30 minutes

---

## Annexes

### Captures d'Écran des Éléments Critiques:

- `1-MainActivity-full-view.png` - Code MainActivity
- `2-Manifest-debuggable-critical.png` - Flag debuggable en détail
- `2-Manifest-allowBackup-high.png` - Flag allowBackup en détail
- `2-Manifest-exported-receiver.png` - Récepteur exposé en détail

---

**Rapport généré le:** 26 Avril 2026  
**Analyste:** Fatima Ezzahra El Boudhiri  
**Établissement:** ENSA  
**Classe:** Sécurité des Applications Multimedia  
**Laboratoire:** Lab 4  
**Professeur:** M. Lachgar  

---

*Ce rapport est confidentiel et destiné à des fins d'amélioration de la sécurité.*
