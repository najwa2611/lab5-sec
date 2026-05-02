Ces screenshots montre le processus de l installation d un crackable level 2 , jadx , et finalement ghidra puis leur utilisation pour l analyse et le reverse ing pour finalement trouver le mot cle "Thanks for all the fish"


# Reverse Engineering Android - UnCrackable Level 2

## Objectif

Analyser une application Android qui cache sa logique de vérification dans une bibliothèque native (`libfoo.so`), afin de retrouver le secret attendu.

## Outils utilisés

| Outil | Utilité |
|-------|---------|
| Android Emulator | Exécution de l'application |
| ADB | Installation de l'APK |
| JADX | Décompilation du code Java |
| Ghidra | Reverse engineering du binaire natif |

---

## Démarche suivie

### 1. Installation et observation

bash
adb install UnCrackable-Level2.apk

L'application affiche une interface simple : champ texte + bouton "Verify".
Des entrées incorrectes produisent un message d'erreur → une valeur secrète est attendue.

### 2. Analyse Java avec JADX
MainActivity.java :

java
public void verify(View view) {
    String input = ((EditText) findViewById(R.id.edit_text)).getText().toString();
    if (this.m.a(input)) {
        // Succès
    } else {
        // Échec
    }
}
CodeCheck.java :

java
public class CodeCheck {
    private native boolean bar(byte[] bArr);
    public boolean a(String str) {
        return bar(str.getBytes());
    }
}
Le code Java ne contient pas le secret. La vérification est déléguée à une méthode native bar().

### 3. Extraction de la bibliothèque native
bash
unzip UnCrackable-Level2.apk -d uncrackable_l2
ls uncrackable_l2/lib/x86/libfoo.so
### 4. Analyse native avec Ghidra
La fonction Java_sg_vantagepoint_uncrackable2_CodeCheck_bar a été identifiée.

Code décompilé :

c
undefined4 Java_sg_vantagepoint_uncrackable2_CodeCheck_bar(int *param_1, ...) {
    char local_30 [24];
    
    builtin_strncpy(local_30, "Thanks for all the fish", 0x18);
    
    // Vérification de la longueur (0x17 = 23 caractères)
    if (iVar1 == 0x17) {
        iVar1 = strncmp(__s1, local_30, 0x17);
        if (iVar1 == 0) {
            return 1;  // Succès
        }
    }
    return 0;  // Échec
}
### 5. Secret trouvé
text
Thanks for all the fish
### 6. Validation
Saisie dans l'application → message "Success!" affiché.

Schéma du flux logique
text
Utilisateur
    ↓
MainActivity (récupère l'entrée)
    ↓
CodeCheck.a() (appelle la méthode native)
    ↓
libfoo.so (bibliothèque native)
    ↓
Java_sg_vantagepoint_uncrackable2_CodeCheck_bar
    ↓
strncmp(input, "Thanks for all the fish")
    ↓
Succès / Échec

#### Enseignements retenus
Point clé	Explication
System.loadLibrary()	Charge une bibliothèque native (.so)
native keyword	Méthode implémentée en C/C++
JNI naming	Java_package_Class_method
Ghidra	Outil puissant pour décompiler du code natif
strncmp()	Fonction de comparaison souvent utilisée pour cacher un secret

## Conclusion
Ce laboratoire montre qu'une application Android peut sembler simple en Java, mais cacher sa logique critique dans du code natif. Cependant, même le code natif n'est pas invulnérable : avec les bons outils (JADX + Ghidra), on peut retrouver le secret et comprendre le fonctionnement interne.

### Références

OWASP MSTG - Crackmes
JADX
Ghidra
