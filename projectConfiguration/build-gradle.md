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
    id("com.google.devtools.ksp")
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
    implementation(libs.androidx.room.ktx)
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
