Layout `activity_main.xml` para utilização com o Navigation component.

> ⚠ Só funciona corretamente se tiver os ficheiros build.gradle bem configurados

> Tem de gerar o ficheiro do grafo de navegação se o mesmo não existir (alt+enter sobre "@navigation/nav_graph")
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.fragment.app.FragmentContainerView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".ui.MainActivity"
    android:name="androidx.navigation.fragment.NavHostFragment"
    app:navGraph="@navigation/nav_graph"
    app:defaultNavHost="true"/>
```