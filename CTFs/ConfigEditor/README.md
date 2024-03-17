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

Prima di lanciarci in analisi statiche e dinamiche cerchiamo di capire cosa fa l'applicazione.. semplicemente.. usandola

All'apertura ci viene richiesto la possibilità di accedere a foto e media sul device. Clicchiamo su Allow

<p align="center">
<img src="assets/Access.png" width="350"/>
</p>

Abbiamo a disposizione 2 bottoni, Load and Save.

<p align="center">
<img src="assets/1.png" width="350"/>
</p>

Cliccando su Load vediamo che l'applicazione ha caricato un file example.yaml nei nostri download. Selezioniamo il file e vediamo che il file viene parsato dall'applicazione.

<p align="center">
<img src="assets/app.png" width="350"/>
</p>

Cliccando su Save invece possiamo salvare il medesimo file sul dispositivo.

## Static Analysis

Analizziamo prima il file `AndroidManifest.xml`
Di seguito alcune considerazioni sul file `AndroidManifest.xml`

### Permissions

La nostra applicazione, come dichiarato nel manifest ha la possibilità di connettersi ad Internet, e
gestire i file sulla memoria esterna (sdcard), infatti come successo nella fase di recon una volta dato accesso alla memoria è stato caricato un file example.yaml nei download della sdcard, senza questi permessi non sarebbe stato possibile. 
I permessi per Internet non vengono esplicitamente chiesti (inserire il perchè). Di seguito lo snippet di XML di cui stiamo parlando:

```xml
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE"/>
```

MainActivity and deep link explained

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

+ Look into MainActivity and what happen
+ SnakeYaml CVE
+ Fast Analysis CVE
+ SemGrep Rules

## Dynamic Analysis

+ Test DeepLink and what happen with yaml

## Exploiting

+ Adapt exploit to Android Env + Gadget Sink and how to discover

## Shell is the way

+ RCE yes, but where is the shell? method with c script and dalvikvm method

## And now?

+ Android Kernel Privesc, Sandbox Escape, etc
