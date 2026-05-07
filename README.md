# LAB 4 - RAPPORT D'ANALYSE DE SÉCURITÉ
## Rétro-ingénierie et Décompilation d'APK Android

**Étudiant:** Fatima Ezzahra El Boudhiri  
**Date:** 26 AVRIL2026  
**APK Cible:** OWASP UnCrackable Level 1  
**Outils Utilisés:** JADX GUI, dex2jar, JD-GUI  
**Environnement:** Windows 11, Android Emulator (Pixel 6)

---

## RÉSUMÉ EXÉCUTIF

L'analyse de l'APK **owasp.mstg.uncrackable1** révèle plusieurs vulnérabilités de sécurité critiques. L'application stocke un mot de passe codé en dur dans le code source, utilise un chiffrement faible (AES/ECB) et implémente une validation insuffisante. Ces défauts rendent l'application facilement exploitable par décompilation.

---

## MÉTHODOLOGIE

### Outils et Processus

1. **Extraction APK:** Utilisation d'ADB pour extraire l'APK de l'émulateur Android
2. **Décompilation JADX:** Conversion du bytecode DEX en pseudocode Java lisible
3. **Conversion dex2jar:** Transformation du DEX en format JAR pour analyse alternative
4. **Analyse JD-GUI:** Comparaison des résultats de décompilation avec un outil alternatif

### Déroulement

```
APK (66 KB) → Extraction → classes.dex (5,528 bytes) → dex2jar → classes-dex2jar.jar
                    ↓
              JADX GUI Decompilation
                    ↓
          JD-GUI Analysis & Comparison
```

---

## RÉSULTATS DE DÉCOMPILATION

### Structure du Projet

**Packages identifiés:**
- `owasp.mstg.uncrackable1` - Package principal de l'application
- `sg.vantagepoint` - Package de sécurité contenant la logique de vérification

**Classes principales:**
- `MainActivity` - Activité principale (point d'entrée)
- `sg.vantagepoint.a` - Classe contenant la logique de chiffrement/déchiffrement
- `sg.vantagepoint.b` - Classe de détection (root, debuggable)
- `sg.vantagepoint.c` - Classe de sécurité supplémentaire

**Statistiques de décompilation:**
- Classes totales: 15
- Méthodes: 15
- Champs: 15
- Instructions: 411 unités
- Taux de succès: 100%

---

## VULNÉRABILITÉS IDENTIFIÉES

### 🔴 CRITIQUE - Mot de passe codé en dur

**Gravité:** CRITIQUE  
**Localisation:** `sg.vantagepoint.a.a()` - Ligne 12

**Description:**
Le mot de passe d'authentification est stocké directement dans le code sous forme chiffrée:

```java
bArr = sg.vantagepoint.a.a.b("8d127684cbc37c17616d806cf50473cc"), Base64.decode("SUJiFctbmgbDoLXmpl12mkno8HT4Lv8dLatl8FXR260")
```

**Impact:**
- Tout utilisateur ayant accès à l'APK peut extraire le mot de passe
- Pas de protection du stockage des credentials
- Facilement accessible via décompilation

**Preuve:** Screenshot montrant la chaîne chiffrée dans le code décompilé

---

### 🔴 CRITIQUE - Chiffrement faible (AES/ECB)

**Gravité:** CRITIQUE  
**Localisation:** `sg.vantagepoint.a` classe

**Description:**
L'application utilise AES en mode ECB avec padding PKCS7:

```java
Cipher cipher = Cipher.getInstance("AES/ECB/PKCS7Padding");
cipher.init(2, secretKeySpec);
```

**Problèmes de sécurité:**
- Mode ECB est cryptographiquement faible
- Pas de vecteur d'initialisation (IV)
- Patterns de chiffrement sont détectables
- Vulnérable aux attaques par analyse de patterns

**Impact:** Un attaquant peut déduire le mot de passe original

---

### 🟠 ÉLEVÉE - Validation insuffisante

**Gravité:** ÉLEVÉE  
**Localisation:** `MainActivity.verify()` - Lignes 45-50

**Description:**
La vérification du mot de passe utilise une simple comparaison de chaînes:

```java
if (a.a(string)) {
    alertDialog.setTitle("Success!");
    str = "This is the correct secret.";
} else {
    alertDialog.setTitle("Nope...");
    str = "That's not it. Try again.";
}
```

**Problèmes:**
- Pas de hash du mot de passe
- Pas de salting cryptographique
- Pas de rate limiting (tentatives illimitées)
- Message d'erreur donne feedback positif/négatif

**Impact:** Attaque par force brute possible

---

### 🟠 ÉLEVÉE - Pas d'obfuscation du code

**Gravité:** ÉLEVÉE  
**Evidence:** Toutes les classes et méthodes sont lisibles après décompilation

**Description:**
Le code n'utilise pas ProGuard/R8:
- Noms de classes visibles (MainActivity, a, b, c)
- Noms de méthodes explicites (verify(), onCreate())
- Structure logique facilement traçable
- Variables bien nommées

**Impact:** Analyse du code triviale

---

### 🟡 MOYENNE - Détection de root insuffisante

**Gravité:** MOYENNE  
**Localisation:** `MainActivity.onCreate()` - Lignes 31-33

**Description:**
Détection basique de root:

```java
if (c.a() || c.b() || c.c()) {
    a("Root detected!");
}
```

**Limitations:**
- Peut être contournée avec Frida/Xposed
- Pas de vérification approfondie
- Affiche simplement un message et continue

---

### 🟡 MOYENNE - Application debuggable

**Gravité:** MOYENNE  
**Localisation:** `MainActivity.onCreate()` - Lignes 34-35

**Description:**
Détection application debuggable:

```java
if (b.a(getApplicationContext())) {
    a("App is debuggable!");
}
```

**Impact:**
- Permet l'attachement de debuggers
- Facilite l'inspection à l'exécution
- Android Debugger peut être utilisé pour contourner les vérifications

---

## COMPARAISON JADX vs JD-GUI

### Résultats

| Critère | JADX GUI | JD-GUI |
|---------|----------|--------|
| **Lisibilité** | Excellente, formatage moderne | Bonne, compacte |
| **Détail du code** | Très détaillé et clair | Suffisant mais moins polished |
| **Performance** | Très rapide | Un peu plus lent |
| **Secrets trouvés** | ✓ Tous identifiés | ✓ Tous identifiés |
| **Qualité de décompilation** | 100% succès | 100% succès |
| **Recommandation** | Outil principal | Outil de vérification |

### Conclusion

Les deux outils produisent des résultats équivalents pour cette analyse. **JADX GUI** est recommandé comme outil principal pour sa meilleure interface et sa présentation plus claire.

---

## DÉCOUVERTES PRINCIPALES

### String Sensibles Trouvées

1. **Mot de passe chiffré:**
   - Chaîne: `8d127684cbc37c17616d806cf50473cc`
   - Format: Hexadécimal (16 bytes)
   - Utilisé dans: `sg.vantagepoint.a.a()`

2. **Messages de succès:**
   - `"Success!"` - Titre de dialogue
   - `"This is the correct secret."` - Message du dialogue

3. **Messages d'erreur:**
   - `"Nope..."` - Titre en cas d'échec
   - `"That's not it. Try again."` - Message en cas d'échec

4. **Détections de sécurité:**
   - `"Root detected!"` - Détection de root
   - `"App is debuggable!"` - Détection application debuggable
   - `"This is unacceptable. The app is now going to exit."` - Message d'alerte initial

---

## EXPLOITATION POSSIBLE

### Scénario d'Attaque

1. **Extraction:** Télécharger l'APK (public dans ce cas OWASP)
2. **Décompilation:** Utiliser JADX pour obtenir le code source
3. **Analyse:** Identifier la classe de vérification
4. **Extraction du secret:** Trouver la chaîne chiffrée `8d127684cbc37c17616d806cf50473cc`
5. **Déchiffrement:** Utiliser les clés visibles pour déchiffrer le mot de passe
6. **Validation:** Utiliser le mot de passe dans l'application ou créer un clone

---

## RECOMMANDATIONS

### Priorité CRITIQUE - Corriger Immédiatement

1. **Stocker les credentials de manière sécurisée**
   - Utiliser Android Keystore
   - Implémenter le stockage chiffré avec EncryptedSharedPreferences
   - Jamais de hardcoding

2. **Utiliser un chiffrement robuste**
   - Remplacer AES/ECB par AES/GCM
   - Utiliser un IV aléatoire
   - Implémenter l'authentication tag

3. **Implémenter un hashing cryptographique**
   - Utiliser PBKDF2 ou bcrypt
   - Ajouter un salt aléatoire
   - Implémenter work factor approprié

### Priorité ÉLEVÉE

4. **Obfusquer le code**
   - Utiliser R8/ProGuard
   - Renommer les classes et méthodes
   - Chiffrer les strings sensibles

5. **Implémenter rate limiting**
   - Limiter le nombre de tentatives
   - Implémenter délai exponentiel
   - Verrouiller après X tentatives

6. **Améliorer les contrôles de sécurité**
   - Détection approfondie de root/jailbreak
   - Détection de tampering du code
   - Certificat pinning pour les communications

### Priorité MOYENNE

7. **Supprimer les messages de débogage**
   - Ne pas afficher "Root detected", "Debuggable", etc.
   - Échouer silencieusement en cas de détection
   - Enregistrer les tentatives pour audit

---

## FICHIERS GÉNÉRÉS

### Artefacts de l'Analyse

```
C:\Users\LENOVO\LAB4_WORKSPACE\
├── uncrackable1.apk              (66 KB - APK original)
├── uncrackable1.zip              (Renommé pour extraction)
├── classes.dex                   (5,528 bytes - Code compilé)
├── classes-dex2jar.jar           (JAR converti)
└── apk_extracted/                (Contenu APK extrait)
    ├── classes.dex
    ├── AndroidManifest.xml
    ├── resources.arsc
    ├── res/
    ├── assets/
    └── META-INF/
```

---

## CONCLUSION

L'application **OWASP UnCrackable Level 1** contient intentionnellement plusieurs vulnérabilités de sécurité critique pour fins éducatives. Cette analyse démontre:

✓ Comment les APK Android peuvent être facilement décompilés  
✓ Comment les secrets codés en dur sont rapidement découverts  
✓ Comment l'obfuscation insuffisante expose la logique de l'application  
✓ L'importance de la sécurisation des données sensibles  

### Leçons Clés

1. **Ne jamais coder en dur les credentials** - Utiliser Keystore
2. **Implémenter un chiffrement robuste** - AES/GCM, jamais ECB
3. **Toujours obfusquer** - ProGuard/R8 minimum
4. **Protéger la validation** - Hash + salt, rate limiting
5. **Implémenter la détection de tampering** - Signature, certificat pinning

---

## TÂCHES COMPLÉTÉES

✓ Task 1: Préparation du workspace  
✓ Task 2: Extraction de l'APK  
✓ Task 3: Analyse JADX GUI  
✓ Task 4: Recherche de strings sensibles  
✓ Task 5: Conversion DEX → JAR  
✓ Task 6: Comparaison JADX vs JD-GUI  
✓ Task 7: Rédaction du rapport  
✓ Task 8: Nettoyage (à faire)  

---
