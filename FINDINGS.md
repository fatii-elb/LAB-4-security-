# Constats de Sécurité Détaillés

## ConverterTabsJava - Analyse Statique

**Analyste:** Fatima Ezzahra El Boudhiri  
**Établissement:** ENSA  
**Classe:** Sécurité des Applications Multimedia  
**Laboratoire:** Lab 4  
**Professeur:** M. Lachgar  

---

## CONSTAT #1: MODE DÉBOGAGE ACTIVÉ (CRITIQUE)

### Informations Générales

| Élément | Valeur |
|---------|--------|
| **ID** | CVE-CUSTOM-001 |
| **Titre** | Mode Débogage Activé en Production |
| **Sévérité** | CRITIQUE (Score: 9/10) |
| **Type** | Misconfiguration de Manifest |
| **Localisation** | `AndroidManifest.xml`, ligne 35 |
| **Fichier** | AndroidManifest.xml |
| **Ligne** | 35 |

### Code Affecté

```xml
<application
    android:theme="@style/Theme.ConverterTabsJava"
    android:label="@string/app_name"
    android:icon="@mipmap/ic_launcher"
    android:debuggable="true"  <!-- PROBLÈME ICI -->
    android:allowBackup="true"
    ...
```

### Description Détaillée

Le mode débogage est activé pour l'application. Cela signifie que:

1. **Attachement de débogueur:** Un attaquant peut attacher Android Studio ou un débogueur externe à l'application en cours d'exécution
2. **Inspection de la mémoire:** Accès complet à toutes les variables, objets et données en mémoire
3. **Modification du code:** Possibilité de modifier le comportement de l'application en temps réel
4. **Contournement de sécurité:** Bypass de tous les mécanismes de sécurité implémentés
5. **Extraction de données:** Vol de tokens, clés, données utilisateur stockées en mémoire

### Exploitation Possible

```bash
# Attacher un débogueur à l'application
adb shell setprop debug.atrace.tags.enableflags 1

# Utiliser Android Studio pour déboguer
# ou utiliser des outils comme:
# - Android Debug Bridge (adb)
# - Frida pour la manipulation à l'exécution
# - GDB pour le débogage au niveau système
```

### Impact Potentiel

- **Compromis Critique:** L'application peut être entièrement compromise
- **Vol de Données:** Extraction de tous les secrets stockés
- **Modification du Comportement:** L'app peut être forcée à faire n'importe quoi
- **Contournement d'Authentification:** Les vérifications de sécurité peuvent être bypassed
- **Injection de Malveillance:** Code malveillant peut être injecté à l'exécution

### Probabilité d'Exploitation

**Très Élevée** - Tout attaquant avec accès physique ou via ADB peut exploiter

### Preuves

- Trouvé dans AndroidManifest.xml
- Flag `debuggable="true"` explicitement défini
- Aucun contrôle de version de compilaison

### Remédiation

#### Solution 1: Désactiver Complètement (RECOMMANDÉE)

**Avant:**
```xml
android:debuggable="true"
```

**Après:**
```xml
android:debuggable="false"
```

#### Solution 2: Utiliser BuildConfig (Meilleure Pratique)

Dans `build.gradle`:
```gradle
buildTypes {
    debug {
        debuggable true
    }
    release {
        debuggable false
    }
}
```

Dans `AndroidManifest.xml`:
```xml
android:debuggable="${debuggable}"
```

#### Solution 3: Vérifier la Version de Build

Dans le code:
```java
if (!BuildConfig.DEBUG) {
    // Mode production - features de sécurité activées
}
```

### Vérification Post-Remédiation

```bash
# Vérifier que le flag est désactivé
adb shell dumpsys package com.example.convertertabsjava | grep debuggable

# Résultat attendu:
# debuggable=false
```

### Références

- OWASP Mobile Top 10 2016: M2 - Insecure Data Storage
- CWE-489: Active Debug Code
- Android Security: https://developer.android.com/guide/topics/manifest/application-element

---

## CONSTAT #2: SAUVEGARDE COMPLÈTE AUTORISÉE (ÉLEVÉE)

### Informations Générales

| Élément | Valeur |
|---------|--------|
| **ID** | CVE-CUSTOM-002 |
| **Titre** | Sauvegarde Complète de Données Autorisée |
| **Sévérité** | ÉLEVÉE (Score: 8/10) |
| **Type** | Misconfiguration de Manifest |
| **Localisation** | `AndroidManifest.xml`, lignes 36-38 |
| **Fichier** | AndroidManifest.xml |
| **Lignes** | 36, 37, 38 |

### Code Affecté

```xml
<application
    ...
    android:allowBackup="true"  <!-- PROBLÈME 1 -->
    android:fullBackupContent="@xml/backup_rules"  <!-- PROBLÈME 2 -->
    ...
>
```

### Description Détaillée

L'application autorise la sauvegarde complète de toutes ses données. Cela permet:

1. **Extraction via ADB:** `adb backup -all` extrait toutes les données
2. **Accès aux SharedPreferences:** Fichiers contenant souvent credentials, tokens, clés
3. **Accès aux Bases de Données:** Données sensibles non chiffrées
4. **Accès aux Fichiers:** Tous les fichiers stockés dans le répertoire privé
5. **Restauration Ailleurs:** Les données peuvent être restaurées sur n'importe quel appareil

### Exploitation Possible

```bash
# Extraire la sauvegarde de l'application
adb backup -f backup.ab com.example.convertertabsjava

# Convertir le fichier de sauvegarde
dd if=backup.ab bs=1 skip=24 | openssl zlib -d > backup.tar

# Extraire les données
tar xf backup.tar

# Accéder aux SharedPreferences
cat shared_prefs/credentials.xml

# Résultat:
# <map>
#   <string name="auth_token">eyJhbGciOiJIUzI1NiJ9...</string>
#   <string name="api_key">sk_live_abc123xyz</string>
# </map>
```

### Impact Potentiel

- **Vol Complet de Données:** Toutes les données de l'app sont accessibles
- **Exposition de Credentials:** Mots de passe, tokens, clés API volés
- **Compromis de Confidentialité:** Données personnelles de l'utilisateur exposées
- **Usurpation d'Identité:** Les credentials volés permettent l'impersonation
- **Accès Compte Non Autorisé:** Les tokens volés donnent accès aux services backend

### Données à Risque

**Dans SharedPreferences:**
- Tokens d'authentification
- Sessions utilisateur
- Clés de chiffrement
- Préférences sensibles

**Dans les Bases de Données:**
- Enregistrements utilisateur
- Données de paiement
- Informations personnelles

### Remédiation

#### Solution 1: Désactiver les Sauvegardes (SIMPLE)

**Avant:**
```xml
android:allowBackup="true"
android:fullBackupContent="@xml/backup_rules"
```

**Après:**
```xml
android:allowBackup="false"
```

#### Solution 2: Restreindre le Contenu Sauvegardé (COMPLEXE)

Créer `res/xml/backup_rules.xml`:
```xml
<?xml version="1.0" encoding="utf-8"?>
<full-backup-content>
    <!-- Inclure uniquement les données non-sensibles -->
    <include domain="sharedpref" path="." />
    
    <!-- Exclure les données sensibles -->
    <exclude domain="sharedpref" path="credentials.xml" />
    <exclude domain="sharedpref" path="tokens.xml" />
    <exclude domain="database" path="sensitive_db.db" />
    <exclude domain="file" path="." />
</full-backup-content>
```

#### Solution 3: Chiffrer les Données Sensibles

Avant de sauvegarder:
```java
// Chiffrer les données sensibles
SharedPreferences prefs = getSharedPreferences("credentials", MODE_PRIVATE);
String token = prefs.getString("auth_token", "");

// Chiffrer avec EncryptedSharedPreferences
EncryptedSharedPreferences encryptedPrefs = 
    EncryptedSharedPreferences.create(
        context,
        "encrypted_prefs",
        MasterKeys.getOrCreate(MasterKeys.AES256_GCM_SPECIFICATION),
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    );
```

### Vérification Post-Remédiation

```bash
# Vérifier que les sauvegardes sont désactivées
adb shell dumpsys package com.example.convertertabsjava | grep allowBackup

# Résultat attendu:
# allowBackup=false
```

### Références

- OWASP Mobile Top 10 2016: M2 - Insecure Data Storage
- Android Developers: https://developer.android.com/guide/topics/data/keystore
- CWE-200: Exposure of Sensitive Information

---

## CONSTAT #3: COMPOSANT RÉCEPTEUR EXPOSÉ (MOYENNE)

### Informations Générales

| Élément | Valeur |
|---------|--------|
| **ID** | CVE-CUSTOM-003 |
| **Titre** | Composant Récepteur Exposé à d'Autres Applications |
| **Sévérité** | MOYENNE (Score: 5/10) |
| **Type** | Misconfiguration de Manifest |
| **Localisation** | `AndroidManifest.xml`, lignes 57-65 |
| **Composant** | `androidx.profileinstaller.ProfileInstallReceiver` |
| **Fichier** | AndroidManifest.xml |
| **Lignes** | 57-65 |

### Code Affecté

```xml
<receiver
    android:name="androidx.profileinstaller.ProfileInstallReceiver"
    android:permission="android.permission.DUMP"
    android:enabled="true"
    android:exported="true"  <!-- PROBLÈME ICI -->
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

### Description Détaillée

Le récepteur `ProfileInstallReceiver` est marqué comme `exported="true"`, ce qui signifie:

1. **Accès par d'autres apps:** Toute application peut envoyer des intentions (intents) à ce récepteur
2. **Déclenchement d'actions:** Certaines actions peuvent être déclenchées par d'autres applications
3. **Comportement inattendu:** L'app peut se comporter de façon non attendue
4. **Dénégation de service:** Une app malveillante peut bloquer les opérations légitimes

### Exploitation Possible

```java
// Code d'une app malveillante
Intent intent = new Intent("androidx.profileinstaller.action.INSTALL_PROFILE");
intent.setPackage("com.example.convertertabsjava");
context.sendBroadcast(intent);

// Ou avec startService
Intent serviceIntent = new Intent("androidx.profileinstaller.action.SAVE_PROFILE");
serviceIntent.setPackage("com.example.convertertabsjava");
context.startService(serviceIntent);
```

### Impact Potentiel

- **Comportement Inattendu:** L'app peut se comporter bizarrement
- **Dénégation de Service:** Les opérations légitimes peuvent être bloquées
- **Manipulation de Profil:** L'installation de profils peut être affectée
- **Consommation de Ressources:** L'app peut être forcée à consommer des ressources

### Remédiation

#### Solution: Désexposer le Récepteur (RECOMMANDÉE)

**Avant:**
```xml
android:exported="true"
```

**Après:**
```xml
android:exported="false"
```

**Justification:** Ce récepteur est interne à Android et ne doit pas être accessible par d'autres applications. Il ne fait partie d'aucune API publique.

### Vérification Post-Remédiation

```bash
# Vérifier que le récepteur est désexposé
adb shell dumpsys package com.example.convertertabsjava

# Ou vérifier dans AndroidManifest.xml
android:exported="false"
```

### Références

- Android Security: https://developer.android.com/guide/topics/manifest/receiver-element
- Google Play Console: https://support.google.com/googleplay/android-developer/answer/9859152
- OWASP: Component Analysis

---

## Résumé Comparatif des Constats

| Constat | Sévérité | Critère | Impact | Remédiation |
|---------|----------|---------|--------|------------|
| **#1 - Débogage** | CRITIQUE | Configuration | CRITIQUE | Désactiver |
| **#2 - Sauvegarde** | ÉLEVÉE | Données | CRITIQUE | Désactiver/Restreindre |
| **#3 - Récepteur** | MOYENNE | Accès | MOYEN | Désexposer |

---

## Checklist de Remédiation

- [ ] Constat #1: `android:debuggable="false"`
- [ ] Constat #2: `android:allowBackup="false"`
- [ ] Constat #3: `android:exported="false"` sur ProfileInstallReceiver
- [ ] Rebuilder l'APK
- [ ] Tester les fonctionnalités
- [ ] Ré-analyser la sécurité
- [ ] Approuver pour la production

---

**Document généré le:** 26 Avril 2026  
**Version:** 1.0  
**Analyste:** Fatima Ezzahra El Boudhiri  
**Établissement:** ENSA  
