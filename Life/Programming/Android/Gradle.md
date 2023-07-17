#### Build variants
Build type - gradle options that define different configs for different lifecycle of app development, such as dev, release. Dev might contain additional logging logic and release might shrink and obfuscate.
Product flavors - gradle options for different versions of application. For example free and paid versions.
Build variant - cross option for building that includes all different combinations of build types and product flavors.

#### Manifest entries

You can specify values for some properties of the manifest file in the build variant configuration. These build values override the existing values in the manifest file. This is useful if you want to generate multiple variants of your app with a different application name, minimum SDK version, or target SDK version. When multiple manifests are present, the manifest merger tool [merges manifest settings](https://developer.android.com/studio/build/manage-manifests#merge-manifests).

#### Proguard
Proguard is an util that shrinks, optimizes and obfuscates code for better performance and security. Android gradle plugin lets you configure proguard rule

#### Dependencies

The build system manages project dependencies from your local file system and from remote repositories. This means you don't have to manually search, download, and copy binary packages of your dependencies into your project directory. To find out more, see [Add build dependencies](https://developer.android.com/studio/build/dependencies).

#### Signing

The build system lets you specify signing settings in the build configuration, and it can automatically sign your app during the build process. The build system signs the debug version with a default key and certificate using known credentials to avoid a password prompt at build time. The build system does not sign the release version unless you explicitly define a signing configuration for this build. If you don't have a release key, you can generate one as described in [Sign your app](https://developer.android.com/studio/publish/app-signing). Signed release builds are required for distributing apps through most app stores.
Приложения для Android имеют криптографическую подпись разработчика. С ее помощью менеджер пакетов на устройстве пользователя может проверить, что каждое обновление приложения происходит из одного и того же источника, и что оно не было подделано. Google Play также применяет эту проверку подписи, когда вы загружаете свой APK-файл на консоль Google Play, так что даже если бы у кого-то были ваши учетные данные для входа, было бы невозможно отправить вредоносное обновление, не имея также доступа к вашему закрытому ключу.

#### Android app bundle 
An _Android App Bundle_ is a publishing format that includes all your app’s compiled code and resources, and defers APK generation and signing to Google Play.

Google Play uses your app bundle to generate and serve optimized APKs for each device configuration, so only the code and resources that are needed for a specific device are downloaded to run your app. You no longer have to build, sign, and manage multiple APKs to optimize support for different devices, and users get smaller, more-optimized downloads.


#### Gradle Wrapper
Gradle wrapper - is an application which performs installing and launching gradle. In gradle-wrapper.properties we define gradle version and other stuff/

#### Gradle settings
Inside settings.gradle file we define project-level repository settings and informings Gradle which modules it should include when building your app

#### Top-level build file
Inside build.gradle file in root of the project we typically define the common versions of plugins used by modules in our project.
plugins {    /**  
     * Use `apply false` in the top-level build.gradle file to add a Gradle  
     * plugin as a build dependency but not apply it to the current (root)  
     * project. Don't use `apply false` in sub-projects. For more information,  
     * see [Applying external plugins with same version to subprojects](https://docs.gradle.org/current/userguide/plugins.html#sec:subprojects_plugins_dsl).  
     */    id("com.android.application") version "8.0.0" apply false    id("com.android.library") version "8.0.0" apply false    id("org.jetbrains.kotlin.android") version "1.8.22" apply false  
}

#### Module-level build file
Inside build.gradle file on module level we define productFlavors, build types, module dependencies etc. for particular module.

#### Gradle properties files
Inside gradle.properties file we can configure project-wide Gradle settings, such as the Gradle daemon's maximum heap size.

#### Source sets
We can defines source sets of files for all build variants or for particular ones.
`src/main/`

This source set includes code and resources common to all build variants.

`src/buildType/`

Create this source set to include code and resources only for a specific build type.

`src/productFlavor/`

Create this source set to include code and resources only for a specific product flavor.

**Note:** If you configure your build to [combine multiple product flavors](https://developer.android.com/studio/build/build-variants#flavor-dimensions), you can create source set directories for each _combination_ of product flavors between the flavor dimensions: `src/productFlavor1ProductFlavor2/`.

`src/productFlavorBuildType/`

Create this source set to include code and resources only for a specific build variant.