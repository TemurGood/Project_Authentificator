# 🛠 SPEC.md: Android Web-to-App Authenticator

This document serves as the single source of truth for generating the **Android TOTP Authenticator** application using Kotlin, Android WebView, Tailwind CSS, and `otpauth.js`.

---

## 1. Project Specifications & Requirements

- **App Name:** Authenticator
- **Package Name:** `com.example.authenticator`
- **Target SDK:** 34 | **Min SDK:** 24
- **Language:** Kotlin (JVM 17)
- **Engine:** Android WebView (`file:///android_asset/www/index.html`)
- **Web Stack:** HTML5, Tailwind CSS (CDN), Vanilla JS (ES6+), `otpauth` (v9.x via CDN).
- **Storage:** LocalStorage (zero network dependency for credentials).

---

## 2. Directory Tree Structure to Generate

```text
.
├── .gitignore
├── build.gradle
├── settings.gradle
├── gradle.properties
├── PRD.md
├── SPEC.md
└── app/
    ├── build.gradle
    └── src/
        └── main/
            ├── AndroidManifest.xml
            ├── assets/
            │   └── www/
            │       └── index.html
            ├── java/
            │   └── com/
            │       └── example/
            │           └── authenticator/
            │               └── MainActivity.kt
            └── res/
                ├── layout/
                │   └── activity_main.xml
                ├── values/
                │   ├── colors.xml
                │   ├── strings.xml
                │   └── themes.xml
                └── xml/
                    └── network_security_config.xml
```

---

## 3. Step-by-Step Implementation Details

### Step 1: Root & Gradle Configuration

#### `settings.gradle`
```groovy
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
rootProject.name = "AuthenticatorApp"
include ':app'
```

#### `build.gradle` (Root)
```groovy
plugins {
    id 'com.android.application' version '8.2.2' apply false
    id 'org.jetbrains.kotlin.android' version '1.9.22' apply false
}

tasks.register('clean', Delete) {
    delete rootProject.buildDir
}
```

#### `gradle.properties`
```properties
org.gradle.jvmargs=-Xmx2048m -Dfile.encoding=UTF-8
android.useAndroidX=true
android.nonTransitiveRClass=true
```

#### `app/build.gradle`
```groovy
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
}

android {
    namespace 'com.example.authenticator'
    compileSdk 34

    defaultConfig {
        applicationId "com.example.authenticator"
        minSdk 24
        targetSdk 34
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_17
        targetCompatibility JavaVersion.VERSION_17
    }
    kotlinOptions {
        jvmTarget = '17'
    }
}

dependencies {
    implementation 'androidx.core:core-ktx:1.12.0'
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'com.google.android.material:material:1.11.0'
    implementation 'androidx.activity:activity-ktx:1.8.2'
    implementation 'androidx.webkit:webkit:1.10.0'
}
```

---

### Step 2: Android Native Layer

#### `app/src/main/AndroidManifest.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

    <application
        android:allowBackup="true"
        android:icon="@android:drawable/sym_def_app_icon"
        android:label="@string/app_name"
        android:roundIcon="@android:drawable/sym_def_app_icon"
        android:supportsRtl="true"
        android:theme="@style/Theme.Authenticator"
        android:hardwareAccelerated="true"
        android:networkSecurityConfig="@xml/network_security_config">
        
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:configChanges="orientation|screenSize|keyboardHidden"
            android:theme="@style/Theme.Authenticator.NoActionBar">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

#### `app/src/main/res/xml/network_security_config.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="true">
        <trust-anchors>
            <certificates src="system" />
            <certificates src="user" />
        </trust-anchors>
    </base-config>
</network-security-config>
```

#### `app/src/main/res/values/strings.xml`
```xml
<resources>
    <string name="app_name">Authenticator</string>
</resources>
```

#### `app/src/main/res/values/colors.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="black">#FF0F172A</color>
    <color name="white">#FFFFFFFF</color>
</resources>
```

#### `app/src/main/res/values/themes.xml`
```xml
<resources>
    <style name="Theme.Authenticator" parent="Theme.MaterialComponents.DayNight.DarkActionBar">
        <item name="colorPrimary">#6366F1</item>
        <item name="android:statusBarColor">#0F172A</item>
        <item name="android:windowBackground">#0F172A</item>
    </style>
    
    <style name="Theme.Authenticator.NoActionBar" parent="Theme.MaterialComponents.DayNight.NoActionBar">
        <item name="android:statusBarColor">#0F172A</item>
        <item name="android:navigationBarColor">#0F172A</item>
        <item name="android:windowBackground">#0F172A</item>
    </style>
</resources>
```

#### `app/src/main/res/layout/activity_main.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#0F172A">

    <WebView
        android:id="@+id/webView"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</FrameLayout>
```

#### `app/src/main/java/com/example/authenticator/MainActivity.kt`
```kotlin
package com.example.authenticator

import android.annotation.SuppressLint
import android.os.Bundle
import android.webkit.WebChromeClient
import android.webkit.WebSettings
import android.webkit.WebView
import android.webkit.WebViewClient
import androidx.activity.OnBackPressedCallback
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {

    private lateinit var webView: WebView

    @SuppressLint("SetJavaScriptEnabled")
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        webView = findViewById(R.id.webView)

        with(webView.settings) {
            javaScriptEnabled = true
            domStorageEnabled = true
            databaseEnabled = true
            allowFileAccess = true
            allowContentAccess = true
            cacheMode = WebSettings.LOAD_DEFAULT
            useWideViewPort = true
            loadWithOverviewMode = true
        }

        webView.webViewClient = WebViewClient()
        webView.webChromeClient = WebChromeClient()

        onBackPressedDispatcher.addCallback(this, object : OnBackPressedCallback(true) {
            override fun handleOnBackPressed() {
                if (webView.canGoBack()) {
                    webView.goBack()
                } else {
                    isEnabled = false
                    onBackPressedDispatcher.onBackPressed()
                }
            }
        })

        webView.loadUrl("file:///android_asset/www/index.html")
    }
}
```

---

### Step 3: Web App Layer (`assets/www/index.html`)

```html
<!DOCTYPE html>
<html lang="en" class="dark">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <title>Authenticator</title>
  
  <script src="https://cdn.tailwindcss.com"></script>
  <script>
    tailwind.config = {
      darkMode: 'class',
      theme: {
        extend: {
          colors: {
            brand: {
              50: '#EEF2FF',
              500: '#6366F1',
              600: '#4F46E5',
              700: '#4338CA'
            }
          }
        }
      }
    }
  </script>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/otpauth/9.2.2/otpauth.umd.min.js"></script>

  <style>
    * {
      -webkit-tap-highlight-color: transparent;
      user-select: none;
    }
    .mono-num {
      font-feature-settings: "tnum";
      font-variant-numeric: tabular-nums;
    }
  </style>
</head>
<body class="bg-slate-950 text-slate-100 min-h-screen flex flex-col antialiased selection:bg-brand-500 selection:text-white">

  <!-- Header -->
  <header class="sticky top-0 z-20 backdrop-blur-md bg-slate-950/80 border-b border-slate-800/80 px-4 py-3.5 flex items-center justify-between">
    <div class="flex items-center space-x-2.5">
      <div class="w-8 h-8 rounded-lg bg-brand-600 flex items-center justify-center font-bold text-white shadow-lg shadow-brand-500/30">
        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"/>
        </svg>
      </div>
      <h1 class="text-lg font-bold tracking-tight text-white">Authenticator</h1>
    </div>
    <button id="openAddModalBtn" class="bg-brand-600 hover:bg-brand-500 active:scale-95 text-white p-2 rounded-xl transition shadow-md shadow-brand-500/20">
      <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M12 4v16m8-8H4"/>
      </svg>
    </button>
  </header>

  <!-- Accounts List Container -->
  <main class="flex-1 p-4 max-w-md mx-auto w-full space-y-3" id="accountsList"></main>

  <!-- Empty State Placeholder -->
  <div id="emptyState" class="hidden flex-1 flex flex-col items-center justify-center p-8 text-center">
    <div class="w-16 h-16 rounded-2xl bg-slate-900 border border-slate-800 flex items-center justify-center text-slate-500 mb-4">
      <svg class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M15 7a2 2 0 012 2m4 0a6 6 0 01-7.743 5.743L11 17H9v2H7v2H4a1 1 0 01-1-1v-2.586a1 1 0 01.293-.707l5.964-5.964A6 6 0 1121 9z"/>
      </svg>
    </div>
    <h3 class="text-base font-semibold text-slate-300">No accounts added yet</h3>
    <p class="text-xs text-slate-500 mt-1 max-w-xs">Tap the "+" button above to add your first 2FA authentication token.</p>
  </div>

  <!-- Add Account Modal -->
  <div id="addModal" class="fixed inset-0 z-50 flex items-end sm:items-center justify-center p-0 sm:p-4 bg-slate-950/70 backdrop-blur-sm hidden transition-opacity">
    <div class="bg-slate-900 border border-slate-800 w-full sm:max-w-md rounded-t-3xl sm:rounded-2xl p-6 space-y-4 shadow-2xl">
      <div class="flex items-center justify-between pb-2 border-b border-slate-800">
        <h3 class="text-lg font-bold text-white">Add New Account</h3>
        <button id="closeAddModalBtn" class="text-slate-400 hover:text-white p-1">
          <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/>
          </svg>
        </button>
      </div>

      <form id="addAccountForm" class="space-y-3.5">
        <div>
          <label class="block text-xs font-semibold uppercase tracking-wider text-slate-400 mb-1">Service / Issuer</label>
          <input type="text" id="issuerInput" required placeholder="e.g. Google, GitHub, Binance" class="w-full bg-slate-950 border border-slate-800 focus:border-brand-500 rounded-xl px-3.5 py-2.5 text-sm text-white placeholder-slate-600 focus:outline-none transition" />
        </div>

        <div>
          <label class="block text-xs font-semibold uppercase tracking-wider text-slate-400 mb-1">Account / Email</label>
          <input type="text" id="accountInput" required placeholder="e.g. user@example.com" class="w-full bg-slate-950 border border-slate-800 focus:border-brand-500 rounded-xl px-3.5 py-2.5 text-sm text-white placeholder-slate-600 focus:outline-none transition" />
        </div>

        <div>
          <label class="block text-xs font-semibold uppercase tracking-wider text-slate-400 mb-1">Secret Key (Base32)</label>
          <input type="text" id="secretInput" required placeholder="JBSWY3DPEHPK3PXP" class="w-full bg-slate-950 border border-slate-800 focus:border-brand-500 rounded-xl px-3.5 py-2.5 text-sm text-white font-mono uppercase placeholder-slate-600 focus:outline-none transition" />
          <p id="secretError" class="text-xs text-rose-500 mt-1 hidden">Invalid Base32 secret key format.</p>
        </div>

        <div class="pt-2">
          <button type="submit" class="w-full bg-brand-600 hover:bg-brand-500 active:scale-[0.98] text-white font-semibold py-3 rounded-xl transition shadow-lg shadow-brand-500/25">Save Account</button>
        </div>
      </form>
    </div>
  </div>

  <!-- Toast Notification -->
  <div id="toast" class="fixed bottom-6 left-1/2 -translate-x-1/2 z-50 bg-slate-800 text-slate-100 border border-slate-700 text-xs px-4 py-2.5 rounded-full shadow-2xl opacity-0 pointer-events-none transition-all duration-200">
    Code copied to clipboard!
  </div>

  <script>
    const STORAGE_KEY = 'authenticator_accounts_v1';
    let accounts = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]');

    const accountsList = document.getElementById('accountsList');
    const emptyState = document.getElementById('emptyState');
    const addModal = document.getElementById('addModal');
    const openAddModalBtn = document.getElementById('openAddModalBtn');
    const closeAddModalBtn = document.getElementById('closeAddModalBtn');
    const addAccountForm = document.getElementById('addAccountForm');
    const toast = document.getElementById('toast');

    function showToast(message) {
      toast.innerText = message;
      toast.classList.remove('opacity-0', 'translate-y-2');
      toast.classList.add('opacity-100', 'translate-y-0');
      setTimeout(() => {
        toast.classList.remove('opacity-100', 'translate-y-0');
        toast.classList.add('opacity-0', 'translate-y-2');
      }, 2000);
    }

    openAddModalBtn.addEventListener('click', () => {
      addModal.classList.remove('hidden');
      document.getElementById('issuerInput').focus();
    });
    closeAddModalBtn.addEventListener('click', () => addModal.classList.add('hidden'));

    function saveAccounts() {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(accounts));
      renderAccounts();
    }

    function cleanSecret(secret) {
      return secret.replace(/\s+/g, '').toUpperCase();
    }

    addAccountForm.addEventListener('submit', (e) => {
      e.preventDefault();
      const issuer = document.getElementById('issuerInput').value.trim();
      const label = document.getElementById('accountInput').value.trim();
      const secret = cleanSecret(document.getElementById('secretInput').value);

      try {
        const testTotp = new OTPAuth.TOTP({
          issuer: issuer,
          label: label,
          algorithm: 'SHA1',
          digits: 6,
          period: 30,
          secret: OTPAuth.Secret.fromBase32(secret)
        });
        testTotp.generate();

        accounts.push({
          id: Date.now().toString(),
          issuer,
          label,
          secret
        });

        saveAccounts();
        addAccountForm.reset();
        addModal.classList.add('hidden');
        showToast('Account added successfully');
      } catch (err) {
        document.getElementById('secretError').classList.remove('hidden');
      }
    });

    window.deleteAccount = function(id, e) {
      e.stopPropagation();
      if (confirm('Are you sure you want to delete this token?')) {
        accounts = accounts.filter(acc => acc.id !== id);
        saveAccounts();
        showToast('Account removed');
      }
    };

    window.copyCode = function(code, e) {
      navigator.clipboard.writeText(code).then(() => {
        showToast('Copied: ' + code);
      }).catch(() => {
        showToast('Copied: ' + code);
      });
    };

    function renderAccounts() {
      if (accounts.length === 0) {
        accountsList.innerHTML = '';
        emptyState.classList.remove('hidden');
        return;
      }

      emptyState.classList.add('hidden');
      const nowEpoch = Math.floor(Date.now() / 1000);
      const remainingSeconds = 30 - (nowEpoch % 30);
      const progressPercent = (remainingSeconds / 30) * 100;

      accountsList.innerHTML = accounts.map(acc => {
        let code = '000000';
        try {
          const totp = new OTPAuth.TOTP({
            issuer: acc.issuer,
            label: acc.label,
            algorithm: 'SHA1',
            digits: 6,
            period: 30,
            secret: OTPAuth.Secret.fromBase32(acc.secret)
          });
          code = totp.generate();
        } catch (e) {
          code = 'ERROR';
        }

        const formattedCode = code.length === 6 ? `${code.slice(0, 3)} ${code.slice(3)}` : code;

        return `
          <div onclick="copyCode('${code}', event)" class="group cursor-pointer relative bg-slate-900 border border-slate-800 hover:border-slate-700 active:scale-[0.99] rounded-2xl p-4 transition-all duration-150 shadow-lg">
            <div class="flex items-start justify-between">
              <div>
                <span class="inline-block px-2 py-0.5 text-[10px] font-bold uppercase tracking-wider rounded bg-brand-500/10 text-brand-500 border border-brand-500/20">${acc.issuer}</span>
                <p class="text-xs text-slate-400 mt-1 truncate max-w-[200px]">${acc.label}</p>
              </div>
              <button onclick="deleteAccount('${acc.id}', event)" class="text-slate-600 hover:text-rose-400 p-1 rounded transition">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"/>
                </svg>
              </button>
            </div>

            <div class="mt-3 flex items-baseline justify-between">
              <span class="mono-num text-3xl font-extrabold tracking-wider text-white font-mono group-hover:text-brand-50 transition">${formattedCode}</span>
              <span class="text-xs font-semibold ${remainingSeconds <= 5 ? 'text-rose-400 animate-pulse' : 'text-slate-400'}">${remainingSeconds}s</span>
            </div>

            <div class="w-full bg-slate-800 h-1 rounded-full mt-3 overflow-hidden">
              <div class="h-full ${remainingSeconds <= 5 ? 'bg-rose-500' : 'bg-brand-500'} transition-all duration-1000 ease-linear" style="width: ${progressPercent}%"></div>
            </div>
          </div>
        `;
      }).join('');
    }

    setInterval(renderAccounts, 1000);
    renderAccounts();
  </script>
</body>
</html>
```

---

## 4. Verification and Build Instructions

1. **Permissions:**
   ```bash
   chmod +x gradlew
   ```
2. **Build Debug APK:**
   ```bash
   ./gradlew assembleDebug
   ```
3. **Target Binary:**
   `app/build/outputs/apk/debug/app-debug.apk`

---

## 5. Acceptance Checklist for Cursor

- [ ] All Gradle configuration and wrapper files are generated.
- [ ] `MainActivity.kt` configures `domStorageEnabled = true` and `javaScriptEnabled = true`.
- [ ] `assets/www/index.html` loads offline, parses TOTP, shows 30s timers, and saves to LocalStorage.
- [ ] Running `./gradlew assembleDebug` completes with `BUILD SUCCESSFUL`.
