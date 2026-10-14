# Room 3

> ℹ️ Os restantes recursos (e o [libs.versions.toml](../projectConfiguration/libs.versions.toml)) usam o **Room 2.x** — ver [Room Database](room-database.md). Este ficheiro mostra o mesmo exemplo com o **Room 3** e assinala as diferenças.
>
> [Notas de lançamento e guia de migração do Room 3](https://developer.android.com/jetpack/androidx/releases/room3)

## Resumo das diferenças

| | Room 2.x | Room 3 |
|---|---|---|
| Package | `androidx.room` | `androidx.room3` |
| Bibliotecas | `androidx.room:room-runtime` / `room-compiler` | `androidx.room3:room3-runtime` / `room3-compiler` |
| Processador de anotações | KSP, KAPT ou `annotationProcessor` | **Só KSP** |
| Funções do DAO | Normais (bloqueantes) ou `suspend` | **Obrigatoriamente `suspend`** (ou devolvem `Flow`) |
| Driver SQLite | Interno (`SupportSQLite`) | `SQLiteDriver` definido com `.setDriver(...)` |
| Gradle plugin | Opcional | `androidx.room3` com `schemaDirectory` obrigatório |
| `LiveData` / RxJava / Paging no DAO | Incluído | Biblioteca extra + `@DaoReturnTypeConverters` |
| Type converters | `@TypeConverter` / `@TypeConverters` | `@ColumnTypeConverter` / `@ColumnTypeConverters` |
| Transações | `runInTransaction { }` | `withWriteTransaction { }` (suspend) |

<br>

---

## Configuração

`libs.versions.toml`
```toml
[versions]
room3Version = "3.0.3"
sqliteVersion = "2.7.1"

[libraries]
androidx-room3-runtime = { group = "androidx.room3", name = "room3-runtime", version.ref = "room3Version" }
androidx-room3-compiler = { group = "androidx.room3", name = "room3-compiler", version.ref = "room3Version" }
androidx-room3-livedata = { group = "androidx.room3", name = "room3-livedata", version.ref = "room3Version" } # só se usar LiveData no DAO
androidx-sqlite-bundled = { group = "androidx.sqlite", name = "sqlite-bundled", version.ref = "sqliteVersion" }

[plugins]
room3 = { id = "androidx.room3", version.ref = "room3Version" }
```

`build.gradle.kts (Project)`
```kt
plugins {
    ...
    alias(libs.plugins.ksp) apply false
    alias(libs.plugins.room3) apply false   // novo
}
```

`build.gradle.kts (Module :app)`
```kt
plugins {
    ...
    alias(libs.plugins.ksp)
    alias(libs.plugins.room3)   // novo
}

android { ... }

// novo: obrigatório com o plugin. Os ficheiros .json gerados nesta pasta devem ir para o git
room3 {
    schemaDirectory("$projectDir/schemas")
}

dependencies {
    implementation(libs.androidx.room3.runtime)
    ksp(libs.androidx.room3.compiler)
    implementation(libs.androidx.sqlite.bundled)
    implementation(libs.androidx.room3.livedata) // opcional
}
```

> ⚠ Remover as dependências `androidx.room:*` (Room 2) quando migrar. As duas versões podem coexistir porque os packages são diferentes, mas não há razão para manter ambas.

<br>

---

## Entity

**Igual ao Room 2**. Só muda o import.

```kt
import androidx.room3.*   // Room 2: androidx.room.*

@Entity
data class Vehicle(
    val brand: String,
    val model: String,
    val year: Int,
    @PrimaryKey(autoGenerate = true) val id: Long = 0,
)
```

`@Entity(tableName = "some_table")` e `@ColumnInfo(name = "some_col")` funcionam da mesma forma.

<br>

---

## DAO

**Diferença principal:** todas as funções têm de ser `suspend`, exceto as que devolvem um tipo reativo (`Flow`, `LiveData`, ...).

```kt
@Dao
interface VehicleDao {

    @Query("select * from vehicle")
    suspend fun getAll() : List<Vehicle>

    @Query("select * from vehicle")
    fun observeAll() : Flow<List<Vehicle>> // sem suspend: emite uma nova lista sempre que a tabela muda

    @Query("select * from vehicle where id = :id")
    suspend fun get(id: Long) : Vehicle? // null se não existir (com `Vehicle` dá erro quando o id não existe)

    @Insert
    suspend fun insert(vehicle: Vehicle) : Long // devolve o id automaticamente gerado se a primary key tiver autoGenerate a true

    @Update
    suspend fun update(vehicle: Vehicle) : Int // devolve a quantidade de registos afetados

    @Delete
    suspend fun delete(vehicle: Vehicle) : Int // devolve a quantidade de registos afetados
}
```

### LiveData no DAO

No Room 2 bastava devolver `LiveData<...>`. No Room 3 é preciso a dependência `room3-livedata` e indicar o conversor no DAO:

```kt
import androidx.room3.livedata.LiveDataDaoReturnTypeConverter

@Dao
@DaoReturnTypeConverters(LiveDataDaoReturnTypeConverter::class)
interface VehicleDao {

    @Query("select * from vehicle")
    fun observeAllLiveData() : LiveData<List<Vehicle>>
}
```

<br>

---

## RoomDatabase

Diferenças em relação ao Room 2:
- `Room.databaseBuilder<AppDatabase>(context, "app.db")`: a classe passa a ser indicada como tipo genérico, em vez de `AppDatabase::class.java`.
- `.setDriver(BundledSQLiteDriver())`: novo. Define a implementação de SQLite usada.
- `.allowMainThreadQueries()` deixa de ser necessário, porque as funções `suspend` já não bloqueiam a UI.
- `exportSchema = false` não é preciso, porque o plugin `room3` define a pasta onde o esquema é guardado.

```kt
import androidx.room3.*
import androidx.sqlite.driver.bundled.BundledSQLiteDriver

@Database(entities = [Vehicle::class], version = 1)
abstract class AppDatabase : RoomDatabase() {

    abstract fun vehicleDao() : VehicleDao

    companion object {

        private val lock = Any()

        @Volatile
        private var instance: AppDatabase? = null


        operator fun invoke(context: Context) : AppDatabase {

            return instance ?: synchronized(lock) {
                if(instance != null) return  instance!!

                instance = Room.databaseBuilder<AppDatabase>(context.applicationContext, "app.db")
                    .setDriver(BundledSQLiteDriver())
                    .fallbackToDestructiveMigration(dropAllTables = true)
                    .build()

                return instance!!
            }
        }
    }
}
```

<br>

---

## Utilizar o DAO num Fragment

Como as funções são `suspend`, têm de ser chamadas dentro de uma coroutine. Num Fragment usa-se o `lifecycleScope` da view. A coroutine é cancelada automaticamente quando a view é destruída.

```kt
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)

    val dao = AppDatabase(requireContext()).vehicleDao()

    // operações pontuais
    viewLifecycleOwner.lifecycleScope.launch {
        dao.insert(Vehicle("Renault", "Clio", 2020))
        val vehicles = dao.getAll()
        // atualizar a UI com vehicles
    }

    // observar alterações com Flow
    viewLifecycleOwner.lifecycleScope.launch {
        viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
            dao.observeAll().collect { vehicles ->
                // adapter.submitList(vehicles)
            }
        }
    }

    // ou com LiveData (ver secção "LiveData no DAO")
    dao.observeAllLiveData().observe(viewLifecycleOwner) { vehicles ->
        // adapter.submitList(vehicles)
    }
}
```

> No Room 2 (com `allowMainThreadQueries()`) era possível fazer `dao.getAll()` diretamente. No Room 3 isso dá erro de compilação: *"Suspend function 'suspend fun getAll(): List<Vehicle>' can only be called from a coroutine or another suspend function."*

<br>

---

## Type converters

Mudaram de nome: `@TypeConverter` passou a `@ColumnTypeConverter` e `@TypeConverters` passou a `@ColumnTypeConverters`.

```kt
class Converters {
    @ColumnTypeConverter
    fun fromTimestamp(value: Long?): Date? = value?.let { Date(it) }

    @ColumnTypeConverter
    fun dateToTimestamp(date: Date?): Long? = date?.time
}

@Database(entities = [Vehicle::class], version = 1)
@ColumnTypeConverters(Converters::class)
abstract class AppDatabase : RoomDatabase() { ... }
```
