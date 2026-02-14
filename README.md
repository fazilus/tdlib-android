# TDLib for Android

[![](https://jitpack.io/v/fazilus/tdlib-android.svg)](https://jitpack.io/#fazilus/tdlib-android)
[![TDLib](https://img.shields.io/badge/TDLib-1.8.61-orange.svg)](https://github.com/tdlib/td)
[![License](https://img.shields.io/badge/License-BSL--1.0-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Android-green.svg)](https://developer.android.com)
[![Language](https://img.shields.io/badge/Language-Java%20%7C%20Kotlin-purple.svg)](https://kotlinlang.org)

[TDLib](https://github.com/tdlib/td) is a cross-platform library for building Telegram clients. This library contains pre-compiled native libraries (.so files) and Java wrappers for use in Android projects. Both Java and Kotlin projects are supported.

## What is it for?

Why do you need this library if you can build TDLib yourself? The answer is quite simple:
- **Ready to use** — no need to spend time building from source;
- **Identical functionality** — fully compliant with official TDLib;
- **Easy updates** — this library will be updated with important TDLib updates, you just need to update the library instead of building it yourself every time
- **Perfect for Android Studio** — IDE will no longer mark all accesses to TDLib as "errors", because now it sees its code.

If you are still interested in building the library from scratch, refer to these guides:\
[Official build instructions](https://github.com/tdlib/td/tree/master/example/android) or [My more detailed build instructions](https://github.com/fazilus/tdlib-android/blob/main/BUILD.MD) 

## Installation

### For Kotlin DSL (`settings.gradle.kts` + `build.gradle.kts`)

#### Step 1: Add JitPack repository

In your `settings.gradle.kts` (project root):

```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven { url = uri("https://jitpack.io") }
    }
}
```

#### Step 2: Add dependency

In your `build.gradle.kts` (:app):

```kotlin
dependencies {
    implementation("com.github.fazilus:tdlib-android:1.8.61")
}
```

**OR** if you use Version Catalog:
```kotlin
dependencies {
    implementation(libs.tdlib.android)
}
```

and in `libs.version.toml`:

```kotlin
[versions]
tdlib = "1.8.61"

[libraries]
# Core
tdlib-android = { module = "com.github.fazilus:tdlib-android", version.ref = "tdlib" }
```

#### Step 3: Sync project

Click **File → Sync Project with Gradle Files** in Android Studio.

---

### For Groovy (`settings.gradle` + `build.gradle`)

#### Step 1: Add JitPack repository

In your `settings.gradle` (project root):

```groovy
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven { url 'https://jitpack.io' }
    }
}
```

**OR** if you use legacy format, in your `build.gradle` (project root):

```groovy
allprojects {
    repositories {
        google()
        mavenCentral()
        maven { url 'https://jitpack.io' }
    }
}
```

#### Step 2: Add dependency

In your `app/build.gradle`:

```groovy
dependencies {
    implementation 'com.github.fazilus:tdlib-android:1.8.61'
}
```

#### Step 3: Sync project

Click **File → Sync Project with Gradle Files** in Android Studio.

---

## Usage

### Java Example

```java
import org.drinkless.tdlib.Client;
import org.drinkless.tdlib.TdApi;

public class TelegramClient {
    private Client client;

    public void initialize() {
        // Create client with result handler
        client = Client.create(object -> {
            // Handle updates
            if (object instanceof TdApi.UpdateAuthorizationState) {
                onAuthorizationStateUpdated(((TdApi.UpdateAuthorizationState) object).authorizationState);
            }
        }, null, null);
    }

    private void onAuthorizationStateUpdated(TdApi.AuthorizationState authorizationState) {
        if (authorizationState instanceof TdApi.AuthorizationStateWaitTdlibParameters) {
            // Set TDLib parameters
            TdApi.SetTdlibParameters parameters = new TdApi.SetTdlibParameters();
            parameters.apiId = YOUR_API_ID;
            parameters.apiHash = "YOUR_API_HASH";
            parameters.databaseDirectory = getFilesDir().getAbsolutePath() + "/tdlib";
            parameters.useMessageDatabase = true;
            parameters.useSecretChats = false;

            client.send(parameters, object -> {});
        } else if (authorizationState instanceof TdApi.AuthorizationStateWaitPhoneNumber) {
            // Enter phone number
            TdApi.SetAuthenticationPhoneNumber phoneNumber = new TdApi.SetAuthenticationPhoneNumber();
            phoneNumber.phoneNumber = "+1234567890";
            client.send(phoneNumber, object -> {});
        } else if (authorizationState instanceof TdApi.AuthorizationStateWaitCode) {
            // Enter verification code
            TdApi.CheckAuthenticationCode code = new TdApi.CheckAuthenticationCode();
            code.code = "12345";
            client.send(code, object -> {});
        } else if (authorizationState instanceof TdApi.AuthorizationStateReady) {
            // Authorized successfully!
        }
    }
}
```

### Kotlin Example

```kotlin
import org.drinkless.tdlib.Client
import org.drinkless.tdlib.TdApi

class TelegramClient {
    private lateinit var client: Client

    fun initialize() {
        // Create client with result handler
        client = Client.create({ obj ->
            // Handle updates
            if (obj is TdApi.UpdateAuthorizationState) {
                onAuthorizationStateUpdated(obj.authorizationState)
            }
        }, null, null)
    }

    private fun onAuthorizationStateUpdated(authorizationState: TdApi.AuthorizationState) {
        when (authorizationState) {
            is TdApi.AuthorizationStateWaitTdlibParameters -> {
                // Set TDLib parameters
                val parameters = TdApi.SetTdlibParameters().apply {
                    apiId = YOUR_API_ID
                    apiHash = "YOUR_API_HASH"
                    databaseDirectory = filesDir.absolutePath + "/tdlib"
                    useMessageDatabase = true
                    useSecretChats = false
                }
                client.send(parameters) {}
            }
            is TdApi.AuthorizationStateWaitPhoneNumber -> {
                // Enter phone number
                val phoneNumber = TdApi.SetAuthenticationPhoneNumber().apply {
                    this.phoneNumber = "+1234567890"
                }
                client.send(phoneNumber) {}
            }
            is TdApi.AuthorizationStateWaitCode -> {
                // Enter verification code
                val code = TdApi.CheckAuthenticationCode().apply {
                    this.code = "12345"
                }
                client.send(code) {}
            }
            is TdApi.AuthorizationStateReady -> {
                // Authorized successfully!
            }
        }
    }
}
```

For detailed API documentation, see the [official TDLib documentation](https://core.telegram.org/tdlib/docs/).

---

## Supported Architectures

The library includes native libraries for the following architectures:

- **arm64-v8a** (64-bit ARM)
- **armeabi-v7a** (32-bit ARM)
- **x86** (32-bit Intel/AMD)
- **x86_64** (64-bit Intel/AMD)

## Requirements

- **Minimum SDK**: Android 4.1 (API 16)
- **Target SDK**: Android 14 (API 34)
- **Java**: 17+

## License

This library is distributed under the same license as the original TDLib — **Boost Software License 1.0**.

```
Boost Software License - Version 1.0 - August 17th, 2003

Permission is hereby granted, free of charge, to any person or organization
obtaining a copy of the software and accompanying documentation covered by
this license (the "Software") to use, reproduce, display, distribute,
execute, and transmit the Software, and to prepare derivative works of the
Software, and to permit third-parties to whom the Software is furnished to
do so, all subject to the following:

The copyright notices in the Software and this entire statement, including
the above license grant, this restriction and the following disclaimer,
must be included in all copies of the Software, in whole or in part, and
all derivative works of the Software, unless such copies or derivative
works are solely in the form of machine-executable object code generated by
a source language processor.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE, TITLE AND NON-INFRINGEMENT. IN NO EVENT
SHALL THE COPYRIGHT HOLDERS OR ANYONE DISTRIBUTING THE SOFTWARE BE LIABLE
FOR ANY DAMAGES OR OTHER LIABILITY, WHETHER IN CONTRACT, TORT OR OTHERWISE,
ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
DEALINGS IN THE SOFTWARE.
```

## Acknowledgments

- [TDLib](https://github.com/tdlib/td)  — original library by Telegram team
- [JitPack](https://jitpack.io) — for convenient Android library publishing

## Feedback

If you encounter any issues with a library, please create an [Issue](https://github.com/fazilus/tdlib-android/issues) in this repository.
