Android build system compiles code and resources into APK or Android App Bundle (AAB) files.
**Gradle** is an advanced build toolkit. It lets you define flexible customizable builds and provide automation of building applications. It is used by Android Studio. 
**Android Gradle plugin** is a plugin which provides Android specific build configurations, settings and processes.
These tools let you make different builds with different settings without changing code base.
Gradle and Android Gradle plugin can be both run from Android Studio or command line, which provide IDE independent builds (for example for CI/CD servers). They have to be updated separately from Android Studio and each other.

There are different build goals for different projects. For example, android library projects are build into Android Archive (AAR) or Java Archive (JAR) files. Most Android project are targeted to users and are build into APK or AAB files.

Gradle and the Android Gradle plugin help you configure the following aspects of your build:
#### Build variants
**Build type** - gradle options that define different configs for different lifecycle of app development, such as dev, release. Dev might contain additional logging logic and release might shrink and obfuscate.
**Product flavors** - gradle options for different versions of application. For example free and paid versions. They let you defince different code and resources for different versions and use common code for all application.
**Build variant** - cross option for building that includes all different combinations of build types and product flavors.
#### Manifest entries
You can specify values for some properties of the manifest file in the build variant configuration. These build values override the existing values in the manifest file. This is useful if you want to generate multiple variants of your app with a different application name, minimum SDK version, or target SDK version. When multiple manifests are present, the manifest merger tool [merges manifest settings](https://developer.android.com/studio/build/manage-manifests#merge-manifests).
#### Proguard
Proguard is an util that shrinks, optimizes and obfuscates code for better performance and security. Android gradle plugin lets you configure different proguard rules for each build variant.
#### Dependencies
The build system manages project dependencies from your local file system and from remote repositories. This means you don't have to manually search, download, and copy binary packages of your dependencies into your project directory.
#### Signing
The build system lets you specify signing settings in the build configuration, and it can automatically sign your app during the build process. The build system signs the debug version with a default key and certificate using known credentials to avoid a password prompt at build time. The build system does not sign the release version unless you explicitly define a signing configuration for this build. If you don't have a release key, you can generate one as described in [Sign your app](https://developer.android.com/studio/publish/app-signing). Signed release builds are required for distributing apps through most app stores.
Приложения для Android имеют криптографическую подпись разработчика. С ее помощью менеджер пакетов на устройстве пользователя может проверить, что каждое обновление приложения происходит из одного и того же источника, и что оно не было подделано. Google Play также применяет эту проверку подписи, когда вы загружаете свой APK-файл на консоль Google Play, так что даже если бы у кого-то были ваши учетные данные для входа, было бы невозможно отправить вредоносное обновление, не имея также доступа к вашему закрытому ключу.

#### Multiple APK support
You can define different code for different screen density or Android Application Binary Interface and compile only the code and resources need for it and package them into APK. However AAB is now a prefered approach because it lets you define splitting by language.


#### Android app bundle 
An _Android App Bundle_ is a publishing format that includes all your app’s compiled code and resources, and defers APK generation and signing to Google Play.

Google Play uses your app bundle to generate and serve optimized APKs for each device configuration, so only the code and resources that are needed for a specific device are downloaded to run your app. You no longer have to build, sign, and manage multiple APKs to optimize support for different devices, and users get smaller, more-optimized downloads.


#### Gradle Wrapper
Gradle wrapper - is an application which performs installing and launching gradle.  In gradle-wrapper.properties we define gradle version and other stuff. We use `gradlew` to perform downloading of Gradle distribution and starting build of the current project.

#### Gradle settings
Inside settings.gradle file we define project-level repository settings and informings Gradle which modules it should include when building your app. We have to define all modules of multi-module app here.

```kotlin
pluginManagement {

    /**
      * The pluginManagement.repositories block configures the
      * repositories Gradle uses to search or download the Gradle plugins and
      * their transitive dependencies. Gradle pre-configures support for remote
      * repositories such as JCenter, Maven Central, and Ivy. You can also use
      * local repositories or define your own remote repositories. The code below
      * defines the Gradle Plugin Portal, Google's Maven repository,
      * and the Maven Central Repository as the repositories Gradle should use to look for its
      * dependencies.
      */

    repositories {
        gradlePluginPortal()
        google()
        mavenCentral()
    }
}
dependencyResolutionManagement {

    /**
      * The dependencyResolutionManagement.repositories
      * block is where you configure the repositories and dependencies used by
      * all modules in your project, such as libraries that you are using to
      * create your application. However, you should configure module-specific
      * dependencies in each module-level build.gradle file. For new projects,
      * Android Studio includes Google's Maven repository and the Maven Central
      * Repository by default, but it does not configure any dependencies (unless
      * you select a template that requires some).
      */

  repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
  repositories {
      google()
      mavenCentral()
  }
}
rootProject.name = "My Application"
include(":app")
```

#### Top-level build file
Inside build.gradle file in root of the project we typically define the common versions of plugins used by modules in our project or any other settings.  These settings will be applied to each module.
```kotlin
plugins {    
 * Use `apply false` in the top-level build.gradle file to add a Gradle  
 * plugin as a build dependency but not apply it to the current (root)  
 * project. Do not use `apply false` in sub-projects. For more information,  
 * see [Applying external plugins with same version to subprojects](https://docs.gradle.org/current/userguide/plugins.html#sec:subprojects_plugins_dsl).  
    id("com.android.application") version "8.0.0" apply false    
    id("com.android.library") version "8.0.0" apply false    
    id("org.jetbrains.kotlin.android") version "1.8.22" apply false  
}
```

#### Module-level build file
Inside build.gradle file on module level we define productFlavors, build types, module dependencies etc. for particular module. These settings override main/ module settings, top-level settings and AndroidManifest.

#### Gradle properties files
Inside gradle.properties file we can configure project-wide Gradle settings, such as the Gradle daemon's maximum heap size.
There is also local.properties file, which defines where sdk and ndk and other paths are defined.

#### SDK versions
In your module-level build file there are different versions of Android SDK are defined to compile project, set minimum available version for project or specify platform behaviour.

**compileSdk** - Android SDK used to compile your application, defines which Android and Java APIs features can be used **by developer** during compilation. In order to use latest Android and Java features set compileSdk to latest version, it is highly recommended as it doesn't have any negative impact on your app. Some features may not be supported on earlier Android versions, so you can conditionally define logic for different OS versions or use AndroidX support libraries. Some libraries also may require minimum `compileSDK` version.

**minSdk** - defines minimum supported version of Android OS for your project. Restricts number of devices that can install your application. You should decide what percentage of users is acceptable for your application. If you are setting `minSdk` to low value, you have to use conditional usage of feature or support libraries in more places in your code. Linter will help. If you don't use support for older versions your application will crash on them.

**targetSdk** - serves 2 main purposes: 
1. Defines runtime features for your application
2. Asserts on which version you have tested your application
`targetSdk` defines which features are preferable for your application. If app runs on SDK higher that your `targetSdk` it will run compatibility mode to behave similar to your `targetSdk`. For example, in API level 23 runtime permissions were introduced, by setting your `targetSdk` to 22 your application will not require to use runtime permissions on SDK higher than 23, but you can use features available in your `compileSdk`. Google Play defines additional restrictions on `targetSdk`. It is required for new apps to set almost the most latest `targetSdk` in order to have latest security and performance behaviour.
The value of `targetSdk` must be less or equal to `compileSdk`.

The values of `compileSdk` and `targetSdk` don't need to be the same. Keep the following basic principles in mind:

- `compileSdk` gives you access to new APIs
- `targetSdk` sets the runtime behavior of your app
- `targetSdk` must be less than or equal to `compileSdk`
- `minSdk` and `compileSdk` defines direct compatibility and impacts runtime environment, `targetSdk` defins backward compatibility

#### App-module build file sample with description comments

```kotlin
/**
 * The first section in the build configuration applies the Android Gradle plugin
 * to this build and makes the android block available to specify
 * Android-specific build options.
 */

plugins {
    id("com.android.application")
}

/**
 * Locate (and possibly download) a JDK used to build your kotlin
 * source code. This also acts as a default for sourceCompatibility,
 * targetCompatibility and jvmTarget. Note that this does not affect which JDK
 * is used to run the Gradle build itself, and does not need to take into
 * account the JDK version required by Gradle plugins (such as the
 * Android Gradle Plugin)
 */
kotlin {
    jvmToolchain(11)
}

/**
 * The android block is where you configure all your Android-specific
 * build options.
 */

android {

    /**
     * The app's namespace. Used primarily to access app resources.
     */

    namespace = "com.example.myapp"

    /**
     * compileSdk specifies the Android API level Gradle should use to
     * compile your app. This means your app can use the API features included in
     * this API level and lower.
     */

    compileSdk = 33

    /**
     * The defaultConfig block encapsulates default settings and entries for all
     * build variants and can override some attributes in main/AndroidManifest.xml
     * dynamically from the build system. You can configure product flavors to override
     * these values for different versions of your app.
     */

    defaultConfig {

        // Uniquely identifies the package for publishing.
        applicationId = "com.example.myapp"

        // Defines the minimum API level required to run the app.
        minSdk = 21

        // Specifies the API level used to test the app.
        targetSdk = 33

        // Defines the version number of your app.
        versionCode = 1

        // Defines a user-friendly version name for your app.
        versionName = "1.0"
    }

    /**
     * The buildTypes block is where you can configure multiple build types.
     * By default, the build system defines two build types: debug and release. The
     * debug build type is not explicitly shown in the default build configuration,
     * but it includes debugging tools and is signed with the debug key. The release
     * build type applies ProGuard settings and is not signed by default.
     */

    buildTypes {

        /**
         * By default, Android Studio configures the release build type to enable code
         * shrinking, using minifyEnabled, and specifies the default ProGuard rules file.
         */

        getByName("release") {
            isMinifyEnabled = true // Enables code shrinking for the release build type.
            proguardFiles(
                getDefaultProguardFile("proguard-android.txt"),
                "proguard-rules.pro"
            )
        }
    }

    /**
     * The productFlavors block is where you can configure multiple product flavors.
     * This lets you create different versions of your app that can
     * override the defaultConfig block with their own settings. Product flavors
     * are optional, and the build system does not create them by default.
     *
     * This example creates a free and paid product flavor. Each product flavor
     * then specifies its own application ID, so that they can exist on the Google
     * Play Store, or an Android device, simultaneously.
     *
     * If you declare product flavors, you must also declare flavor dimensions
     * and assign each flavor to a flavor dimension. Flavor dimension defines combinations of
     * build variants. For example, the following is grouped [debug, release], [paid, free]
     * [minApi21, minApi24]. Resulting version may be minApi24PaidDebug
     */

    flavorDimensions += "tier"
    productFlavors {
        create("free") {
            dimension = "tier"
            applicationId = "com.example.myapp.free"
        }

        create("paid") {
            dimension = "tier"
            applicationId = "com.example.myapp.paid"
        }
        create("minApi21") {
            dimension = "apiVersion"
            applicationId = "com.example.myapp.free"
        }

        create("minApi24") {
            dimension = "apiVersion"
            applicationId = "com.example.myapp.paid"
        }
    }

    /**
     * To override source and target compatibility (if different from the
     * toolchain JDK version), add the following. All of these
     * default to the same value as kotlin.jvmToolchain. If you're using the
     * same version for these values and kotlin.jvmToolchain, you can
     * remove these blocks.
     */
    //compileOptions {
    //    sourceCompatibility = JavaVersion.VERSION_11
    //    targetCompatibility = JavaVersion.VERSION_11
    //}
    //kotlinOptions {
    //    jvmTarget = "11"
    //}
}

/**
 * The dependencies block in the module-level build configuration file
 * specifies dependencies required to build only the module itself.
 * To learn more, go to Add build dependencies.
 */

dependencies {
    implementation(project(":lib"))
    implementation("androidx.appcompat:appcompat:1.6.1")
    implementation(fileTree(mapOf("dir" to "libs", "include" to listOf("*.jar"))))
}
```

#### Source sets

When you are creating different build variants Android Studio helps you organize source code and resources into the folder. By defaul Android Studio only create `src/main/`, it doesn't create new source folders when you are defining new build variants, you have to manually create them.

We can defines source sets of files for all build variants or for particular ones.
`src/main/` - This source set includes code and resources common to all build variants.

`src/buildType/` - Create this source set to include code and resources only for a specific build type.

`src/productFlavor/` - Create this source set to include code and resources only for a specific product flavor.

**Note:** If you configure your build to [combine multiple product flavors](https://developer.android.com/studio/build/build-variants#flavor-dimensions), you can create source set directories for each _combination_ of product flavors between the flavor dimensions: `src/productFlavor1ProductFlavor2/`.

`src/productFlavorBuildType/` - Create this source set to include code and resources only for a specific build variant.

Priority of defining build files are:

`buildVariant -> buildType -> productFlavor -> main source set -> library dependency`

Gradle merges all the source code and resources in this order as well as AndroidManifest files to create specific build variant.

When you are creating new file use Android Studio's `File -> New` to let studio help you decide which build variant to use for this file

### Java versions in Android builds

![[Pasted image 20240229082909.png]]

**JDK (Java Development Kit)** - set of tools that provide following features:
- Java tools, such as compiler, profiler, archive creator. They are used behind the scenes to create your application.
- Java API, that provide classes and libraries available to use in your code. Not all libraries are supported by Android
- JVM - java virtual machine, that is used to run Java code. It is used by Android Studio IDE and Gradle, but not used on Android devices and emulators.

The startup scripts for **Android Studio** look for a JVM in the following order:
1. `STUDIO_JDK` environment variable
2. `studio.jdk` directory (in the Android Studio distribution)
3. `jbr` directory (JetBrains Runtime), in the Android Studio distribution. Recommended.
4. `JDK_HOME` environment variable
5. `JAVA_HOME` environment variable
6. `java` executable in the `PATH` environment variable

If you run Gradle using the buttons in Android Studio, the **JDK set in the Android Studio** settings is used to run **Gradle**. If you run Gradle in a terminal, either inside or outside Android Studio, the `JAVA_HOME` environment variable (if set) determines which JDK runs the Gradle scripts. If `JAVA_HOME` is not set, it uses the `java` command on your `PATH` environment variable. We can set Gradle JDK manually in **File** (or **Android Studio** on macOS) **> Settings > Build, Execution, Deployment > Build Tools > Gradle**
Make sure to choose a JDK version that is higher than or equal to the JDK versions used by plugins that you use in your Gradle build. To determine the minimum required JDK version for the Android Gradle Plugin (AGP), see the compatibility table in the [release notes](https://developer.android.com/build/releases/gradle-plugin).
When running your build, Gradle creates a process called a _daemon_ to perform the actual build. This process can be reused, as long as the builds are using the same JDK and Gradle version. Reusing a daemon reduces the time to start a new JVM and initialize the build system.

If you start builds with different JDKs or Gradle versions, additional daemons are created, consuming more CPU and memory.

Android SDK defines which Java versions and java libraries Android can use. `compileSdk` defines what features can be used inside project.

| Android      | Java | API and language features supported                                                       |
| ------------ | ---- | ----------------------------------------------------------------------------------------- |
| 14 (API 34)  | 17   | [Core libraries](https://developer.android.com/about/versions/14/features#core-libraries) |
| 13 (API 33)  | 11   | [Core libraries](https://developer.android.com/about/versions/13/features#core-libraries) |
| 12 (API 32)  | 11   | [Java API](https://developer.android.com/about/versions/12/features#java-api)             |
| 11 and lower |      | [Android versions](https://developer.android.com/about/versions)                          |
JDK used to compile Java and Kotlin source code defaults to Gradle JDK if not specified. This means that run on different machines or CI/CD servers may use different versions of JDK. However you can specify JDK toolchain explicitly. This allows following:
- Locates JDK on a machine, if it can't locate it - downloads it (if [toolchain resolver](https://docs.gradle.org/current/userguide/toolchains.html#sec:provisioning) is specified )
- Provides Java API classes and methods to use from source code
- Compiles source code using this JDK version
- Supplies default values for `sourceCompatibility` and `targetCompatibility`

If your source code is only written in Java, specify the Java toolchain version like this:
```kotlin
java {
    toolchain {
        languageVersion.set(JavaLanguageVersion.of(17))
    }
}
```

If your source is only Kotlin or a mix of Kotlin and Java, specify the Java toolchain version like this:
```kotlin
kotlin {
    jvmToolchain(17)
}
```

`sourceCompatibility` determines which JDK will be used to compile Java code (Kotlin is not targeted):

```kotlin
android {
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
    }
}
```

Specifying `targetCompatibility` and `jvmTarget` determines the Java class-format version used when generating bytecode for compiled Java and Kotlin source, respectively.

Some Kotlin features existed before equivalent Java features were added. Early Kotlin compilers had to create their own way to represent those Kotlin features. Some of these features were later added to Java. With later `jvmTarget` levels, the Kotlin compiler might directly use the Java feature, which might result in better performance.

```kotlin
android {
    compileOptions {
        targetCompatibility = JavaVersion.VERSION_17
    }
    kotlinOptions {
        jvmTarget = "17"
    }
}
```

### Configure App Module

**applicationId** is used to uniquely identify your app. It defaults to package name, that you define during the setup of application. Once you uploaded an app to Google Play you should never change applicationId, because Google Play will treat it like separate app. Naming rules for the application ID are a bit more restrictive:

- It must have at least two segments (one or more dots).
- Each segment must start with a letter.
- All characters must be alphanumeric or an underscore [a-zA-Z0-9_].
It is recommended to:
- Explicitely define `applicationId`. If it is not defined it defaults to namespace, so when you change `namespace` `applicationId` is also changed
- Keep `namespace` and `applicationId` same value to avoid confusions.
- Don't change `applicationId` after upload

`namespace` is used as Kotlin or Java package name for generated **`R`** file and `BuildConfig`. It usually defaults to project base package name that is set during project creation. 

Inside dependency block we can use different configurations:

| Configuration                      | Behavior                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `implementation`                   | Gradle adds the dependency to the compile classpath (path where gradle looks for classes during compilation) and packages the dependency to the build output. This configuration informs Gradle that you want include some library or module and make this dependency not accessible to other module. It makes dependency not accessible to other module dependant on current module                                                                                                                                                                        |
| `api`                              | Gradle adds the dependency to compile classpath and packages the dependency to build output. It tells Gradle to transitively include dependency, making it available for other dependant modules.<br>This configuration option can impact build time, so you should use `implementation` where it is possible.                                                                                                                                                                                                                                              |
| `compileOnly`                      | This configuration tells Gradle to add dependency only to compile classpath, not added to build output. It is needed for some code generation or reflection libraries that are not crucial for the runtime.<br>This option can increase performance during runtime, but library module have to implement runtime conditions checks to provide stable work if classes are not available                                                                                                                                                                      |
| `runtimeOnly`                      | Gradle add this dependency only to build output making it available only during runtime. It is rarely used in Android development. It can be used for logging in server applications for defining api as a compile dependency and implementation of logging as a runtime dependency.                                                                                                                                                                                                                                                                        |
| `ksp   kapt   annotationProcessor` | These configuration options are used for code generations/code inspection purposes.<br>This configuration options separates annotation processors from compile classes, improving build time<br>`ksp` is a Kotlin symbol processor and it runs using kotlin compiler. Use it if your project is Kotlin only<br>`kapt` is a separate tools that process annotations before Kotlin or Java compilers execute. Use it if your project is both Kotlin and Java<br>`annotationProcessor` is a annotation processor for Java code. Use it for Java only projects. |
| `lintChecks`                       | Use this configuration to include a library containing lint checks you want Gradle to execute when building your Android app project.                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `lintPublish`                      | Use this configuration in Android library projects to include lint checks you want Gradle to compile into a `lint.jar` file and package in your AAR. This causes projects that consume your AAR to also apply those lint checks. If you were previously using the `lintChecks` dependency configuration to include lint checks in the published AAR, you need to migrate those dependencies to instead use the `lintPublish` configuration.<br>                                                                                                             |
|                                    | Used for publishing library that use lints.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
