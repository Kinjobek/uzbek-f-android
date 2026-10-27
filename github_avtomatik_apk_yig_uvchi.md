name: Build Uzbek-F Android APK

on:
push:
branches: \[ "main", "master" \]
workflow_dispatch:

jobs:
build-android-apk:
runs-on: ubuntu-latest

```
steps:
  - name: 1. Repozitoriydan kodlarni olish
    uses: actions/checkout@v4

  - name: 2. Java JDK 17 o'rnatish
    uses: actions/setup-java@v4
    with:
      distribution: 'temurin'
      java-version: '17'

  - name: 3. Android SDK va Gradle sozlash
    uses: android-actions/setup-android@v3

  - name: 4. Loyiha strukturasini avtomatik tayyorlash va APK yig'ish
    run: |
      mkdir -p app/src/main/java/com/alharitm/uzbekfkeyboard
      mkdir -p app/src/main/res/xml

      # Kotlin faylni joyiga nusxalash
      if [ -f "android/UzbekFKeyboardService.kt" ]; then
        cp android/UzbekFKeyboardService.kt app/src/main/java/com/alharitm/uzbekfkeyboard/UzbekFKeyboardService.kt
      elif [ -f "UzbekFKeyboardService.kt" ]; then
        cp UzbekFKeyboardService.kt app/src/main/java/com/alharitm/uzbekfkeyboard/UzbekFKeyboardService.kt
      fi

      # method.xml yaratish
      cat << 'EOF' > app/src/main/res/xml/method.xml
      <?xml version="1.0" encoding="utf-8"?>
      <input-method xmlns:android="http://schemas.android.com/apk/res/android">
          <subtype
              android:label="Oʻzbek (F-Klaviatura v1.0)"
              android:imeSubtypeLocale="uz_UZ"
              android:imeSubtypeMode="keyboard" />
      </input-method>
      EOF

      # AndroidManifest.xml yaratish
      cat << 'EOF' > app/src/main/AndroidManifest.xml
      <?xml version="1.0" encoding="utf-8"?>
      <manifest xmlns:android="http://schemas.android.com/apk/res/android">
          <uses-permission android:name="android.permission.VIBRATE" />
          <application
              android:allowBackup="true"
              android:label="Oʻzbek F-Klaviatura">
              <activity
                  android:name=".SetupActivity"
                  android:exported="true"
                  android:label="Oʻzbek F-Klaviatura">
                  <intent-filter>
                      <action android:name="android.intent.action.MAIN" />
                      <category android:name="android.intent.category.LAUNCHER" />
                  </intent-filter>
              </activity>
              <service
                  android:name=".UzbekFKeyboardService"
                  android:label="Oʻzbek F-Klaviaturasi (v1.0)"
                  android:permission="android.permission.BIND_INPUT_METHOD"
                  android:exported="true">
                  <intent-filter>
                      <action android:name="android.view.InputMethod" />
                  </intent-filter>
                  <meta-data
                      android:name="android.view.im"
                      android:resource="@xml/method" />
              </service>
          </application>
      </manifest>
      EOF

      # settings.gradle yaratish
      cat << 'EOF' > settings.gradle
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
      rootProject.name = "UzbekFKeyboard"
      include ':app'
      EOF

      # Root build.gradle yaratish
      cat << 'EOF' > build.gradle
      plugins {
          id 'com.android.application' version '8.2.2' apply false
          id 'org.jetbrains.kotlin.android' version '1.9.22' apply false
      }
      EOF

      # App build.gradle yaratish
      cat << 'EOF' > app/build.gradle
      plugins {
          id 'com.android.application'
          id 'org.jetbrains.kotlin.android'
      }
      android {
          namespace 'com.alharitm.uzbekfkeyboard'
          compileSdk 34
          defaultConfig {
              applicationId "com.alharitm.uzbekfkeyboard"
              minSdk 23
              targetSdk 34
              versionCode 1
              versionName "1.0"
          }
          compileOptions {
              sourceCompatibility JavaVersion.VERSION_17
              targetCompatibility JavaVersion.VERSION_17
          }
          kotlinOptions {
              jvmTarget = '17'
          }
      }
      EOF

      gradle assembleDebug

  - name: 5. Tayyor APK faylni yuklab olish uchun chiqarish
    uses: actions/upload-artifact@v4
    with:
      name: Alharitm-Uzbek-F-Keyboard-APK
      path: app/build/outputs/apk/debug/app-debug.apk

```