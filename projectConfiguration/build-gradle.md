> ℹ️ Configuração testada com Android Studio / AGP 9.4.1, Gradle 9.8.1 e compileSdk 37.
>
> A partir do AGP 9 o suporte a Kotlin está incluído no próprio plugin Android, por isso **já não é necessário** (nem deve ser aplicado) o plugin `org.jetbrains.kotlin.android`.

`build.gradle.kts (Project)`
```kt
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.navigation.safeargs) apply false
    alias(libs.plugins.ksp) apply false
}
```
---
`build.gradle.kts (Module :app)`
```kt
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.navigation.safeargs) // gera as classes *Directions e *Args (Safe Args)
    alias(libs.plugins.ksp)                 // necessário para o Room
}


android {

    ...

    compileSdk {
        version = release(37)
    }

    defaultConfig {
        ...
        targetSdk = 37
        ...
    }

    buildFeatures {
        viewBinding = true
    }
}


dependencies {

    // fragment & navigation
    implementation(libs.androidx.fragment.ktx)
    implementation(libs.bundles.navigation)

    // room (database)
    implementation(libs.androidx.room.runtime)
    ksp(libs.androidx.room.compiler)

    // lifecycle (ViewModel & LiveData)
    implementation(libs.bundles.lifecycle)

    // listas
    implementation(libs.androidx.recyclerview)

    // others (nice-to-have)
    implementation(libs.circleimageview) // documentação em https://github.com/hdodenhof/CircleImageView

    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.appcompat)
    implementation(libs.material)
    implementation(libs.androidx.activity)
    implementation(libs.androidx.constraintlayout)
    testImplementation(libs.junit)
    androidTestImplementation(libs.androidx.junit)
    androidTestImplementation(libs.androidx.espresso.core)
}
```

> ⚠ Depois de alterar estes ficheiros, fazer **Sync Project with Gradle Files**. As classes do Safe Args (ex.: `MainFragmentDirections`, `SecondFragmentArgs`) só aparecem depois de um build (**Build > Rebuild Project**).

---
`gradle/wrapper/gradle-wrapper.properties` - para atualizar o Gradle wrapper (no terminal do projeto):
```
./gradlew wrapper --gradle-version 9.8.1
```
> Se der erro por causa do `distributionSha256Sum`, apagar essa linha do ficheiro `gradle-wrapper.properties` ou passar o checksum indicado em https://gradle.org/release-checksums com `--gradle-distribution-sha256-sum <checksum>`.
