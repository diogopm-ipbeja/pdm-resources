`build.gradle (Project)`
```kt
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.ksp) apply false
}

buildscript {
    repositories {
        google()
    }

    dependencies {
        classpath(libs.androidx.navigation.safe.args.gradle.plugin)
    }
}
```
---
`build.gradle (Module)`
```kt
plugins {
    alias(libs.plugins.android.application)a
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.ksp) // se der um erro relacionado, apague esta linha e descomente a linha abaixo
    // id("com.google.devtools.ksp")
}


android {
    
    ...

    compileSdk = 35 // <- mudar

    defaultConfig {
        ...
        targetSdk = 35 // <- mudar
        ...
    }
    
    buildFeatures {
        viewBinding = true
    }
}


dependencies {

    // fragment & navigation
    implementation(libs.androidx.fragment.ktx)
    implementation(libs.androidx.navigation.fragment)
    implementation(libs.androidx.navigation.ui)

    // room (database)
    implementation(libs.androidx.room.runtime)
    annotationProcessor(libs.androidx.room.compiler)
    ksp(libs.androidx.room.compiler)

    // others (nice-to-have)
    implementation(libs.circleimageview) // documentação em https://github.com/hdodenhof/CircleImageView

    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.appcompat)
    implementation(libs.material)
    implementation(libs.androidx.constraintlayout)
    implementation(libs.androidx.navigation.fragment.ktx)
    implementation(libs.androidx.navigation.ui.ktx)
    testImplementation(libs.junit)
    androidTestImplementation(libs.androidx.junit)
    androidTestImplementation(libs.androidx.espresso.core)
}
```
---

`libs.versions.toml`
```toml
[versions]
agp = "8.6.1" # alterar a versão para a indicada se der um erro relacionado
fragmentKtx = "1.8.5"
kotlin = "1.9.24"
coreKtx = "1.15.0"
junit = "4.13.2"
junitVersion = "1.2.1"
espressoCore = "3.6.1"
appcompat = "1.7.0"
material = "1.12.0"
activity = "1.9.3"
constraintlayout = "2.2.0"
navVersion = "2.8.3"
legacySupportV4 = "1.0.0"
recyclerview = "1.3.2"
circleimageviewVersion = "3.1.0"
lifecycleLivedataKtx = "2.8.7"
lifecycleViewmodelKtx = "2.8.7"
navigationFragmentKtx = "2.8.3"
navigationUiKtx = "2.8.3"
roomVersion = "2.6.1"
kspVersion = "2.0.21-1.0.25"

[libraries]
androidx-room-compiler = { module = "androidx.room:room-compiler", version.ref = "roomVersion" }
androidx-room-runtime = { module = "androidx.room:room-runtime", version.ref = "roomVersion" }
androidx-core-ktx = { group = "androidx.core", name = "core-ktx", version.ref = "coreKtx" }
androidx-fragment-ktx = { module = "androidx.fragment:fragment-ktx", version.ref = "fragmentKtx" }
androidx-navigation-fragment = { module = "androidx.navigation:navigation-fragment", version.ref = "navVersion" }
androidx-navigation-safe-args-gradle-plugin = { module = "androidx.navigation:navigation-safe-args-gradle-plugin", version.ref = "navVersion" }
androidx-navigation-ui = { module = "androidx.navigation:navigation-ui", version.ref = "navVersion" }
circleimageview = { module = "de.hdodenhof:circleimageview", version.ref = "circleimageviewVersion" }
junit = { group = "junit", name = "junit", version.ref = "junit" }
androidx-junit = { group = "androidx.test.ext", name = "junit", version.ref = "junitVersion" }
androidx-espresso-core = { group = "androidx.test.espresso", name = "espresso-core", version.ref = "espressoCore" }
androidx-appcompat = { group = "androidx.appcompat", name = "appcompat", version.ref = "appcompat" }
material = { group = "com.google.android.material", name = "material", version.ref = "material" }
androidx-activity = { group = "androidx.activity", name = "activity", version.ref = "activity" }
androidx-constraintlayout = { group = "androidx.constraintlayout", name = "constraintlayout", version.ref = "constraintlayout" }
androidx-legacy-support-v4 = { group = "androidx.legacy", name = "legacy-support-v4", version.ref = "legacySupportV4" }
androidx-recyclerview = { group = "androidx.recyclerview", name = "recyclerview", version.ref = "recyclerview" }
androidx-lifecycle-livedata-ktx = { group = "androidx.lifecycle", name = "lifecycle-livedata-ktx", version.ref = "lifecycleLivedataKtx" }
androidx-lifecycle-viewmodel-ktx = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-ktx", version.ref = "lifecycleViewmodelKtx" }
androidx-navigation-fragment-ktx = { group = "androidx.navigation", name = "navigation-fragment-ktx", version.ref = "navigationFragmentKtx" }
androidx-navigation-ui-ktx = { group = "androidx.navigation", name = "navigation-ui-ktx", version.ref = "navigationUiKtx" }



[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
kotlin-safeargs = { id = "androidx.navigation.safeargs.kotlin"}
ksp = { id = "com.google.devtools.ksp", version.ref = "kspVersion"}
```
