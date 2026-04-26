# Permissions et Composants Exportés

## ConverterTabsJava

**Analyste:** Fatima Ezzahra El Boudhiri  
**Établissement:** ENSA  
**Classe:** Sécurité des Applications Multimedia  
**Laboratoire:** Lab 4  
**Professeur:** M. Lachgar  

---

## Permissions Demandées

### Permissions Système (Standard Android):

**AUCUNE** - L'application ne demande aucune permission système

### Permissions Personnalisées:

| Permission | Protection Level | Utilisation |
|-----------|------------------|------------|
| `com.example.convertertabsjava.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION` | `signature` | Utilisée en interne pour contrôler l'accès au récepteur |

### Évaluation de Sécurité des Permissions:

**BON** - L'application demande un minimum de permissions
- Aucune permission dangereuse (CAMERA, MICROPHONE, CONTACTS, LOCATION, etc.)
- Seulement une permission personnalisée
- Respect du principe du moindre privilège (Principle of Least Privilege)

---

## Composants Exportés

### Liste des Composants:

| # | Composant | Type | Exporté | Intent-Filter | Risque | Commentaire |
|---|-----------|------|---------|----------------|--------|------------|
| 1 | `com.example.convertertabsjava.MainActivity` | Activity | OUI | OUI (MAIN/LAUNCHER) | BAS | C'est le point d'entrée de l'app, doit être exporté |
| 2 | `androidx.startup.InitializationProvider` | Provider | NON | NON | BAS | Provider interne Android, correctement désexposé |
| 3 | `androidx.profileinstaller.ProfileInstallReceiver` | Receiver | OUI | OUI (4 actions) | MOYEN | **À CORRIGER** - Devrait être exported="false" |

---

## Composant à Corriger

### ProfileInstallReceiver (androidx.profileinstaller.ProfileInstallReceiver)

**État Actuel:**
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
    <intent-filter>
        <action android:name="androidx.profileinstaller.action.SKIP_FILE"/>
    </intent-filter>
    <intent-filter>
        <action android:name="androidx.profileinstaller.action.SAVE_PROFILE"/>
    </intent-filter>
    <intent-filter>
        <action android:name="androidx.profileinstaller.action.BENCHMARK_OPERATION"/>
    </intent-filter>
</receiver>
```

**Problème:** `android:exported="true"` permet à d'autres applications d'accéder à ce récepteur

**Solution:**
```xml
android:exported="false"
```

**Justification:** Ce récepteur est interne à Android et ne doit pas être accessible par d'autres applications

---

## Composants Correctement Configurés

### MainActivity

**État:** CORRECT
```xml
<activity
    android:name="com.example.convertertabsjava.MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
```

**Justification:** 
- MainActivity doit être exporté car c'est le point d'entrée principal de l'application
- Les intent-filters MAIN et LAUNCHER indiquent que c'est l'activity de lancement
- C'est la configuration correcte et attendue

---

## Résumé de la Surface d'Attaque

| Aspect | Nombre | Évaluation |
|--------|--------|-----------|
| **Permissions système** | 0 | Excellent |
| **Permissions personnalisées** | 1 | Bon |
| **Activities exportées** | 1 | Attendu (main app) |
| **Services exportés** | 0 | Bon |
| **Providers exposés** | 0 | Bon |
| **Receivers exposés** | 1 | À corriger |
| **Intent-filters** | 5 | Acceptable |

**Surface d'Attaque Globale:** FAIBLE

---

## Recommandations

### Immédiat:
- [ ] Ajouter `android:exported="false"` au ProfileInstallReceiver

### Best Practices:
- [ ] Toujours définir explicitement `android:exported` sur tous les composants (depuis Android 12)
- [ ] Minimaliser le nombre de composants exportés
- [ ] Utiliser des intent-filters spécifiques plutôt que génériques
- [ ] Vérifier régulièrement la surface d'attaque

---

**Analyse complétée le:** 26 Avril 2026  
**Analyste:** Fatima Ezzahra El Boudhiri  
**Établissement:** ENSA  
