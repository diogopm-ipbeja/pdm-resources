### Entity

A definição desta classe anotada com `@Entity` resulta numa tabela com o nome da class. As propriedades desta class traduzem-se em colunas dessa tabela.

```kt
@Entity
data class Vehicle(
    val brand: String,
    val model: String,
    val year: Int,
    @PrimaryKey(autoGenerate = true) val id: Long = 0,
)
```

```@Entity(tableName = "some_table")``` se quisermos dar um nome à tabela diferente do nome da class

```@ColumnInfo(name = "some_col")``` se quisermos dar um nome à coluna diferente do nome da propriedade, por exemplo:

```kt
@Entity
data class Vehicle(
    val brand: String,
    @ColumnInfo(name = "some_model") val model: String,
    val year: Int,
    @PrimaryKey(autoGenerate = true) val id: Long = 0,
)
```

### DAO
Data Access Object

Permite operações sobre a base de dados e normalmente sobre uma tabela

A definição de um DAO deve ser feita com uma `interface` ou `abstract class` e esta deve estar anotada com `@Dao`

```kt
@Dao
interface VehicleDao {

    @Query("select * from vehicle")
    fun getAll() : List<Vehicle>

    @Query("select * from vehicle where id = :id")
    fun get(id: Long) : Vehicle

    @Insert
    fun insert(vehicle: Vehicle) : Long // devolve o id automaticamente gerado se a primary key tiver autoGenerate a true

    @Update
    fun update(vehicle: Vehicle) : Int // devolve a quantidade de registos afetados

    @Delete
    fun delete(vehicle: Vehicle) : Int // devolve a quantidade de registos afetados
}
```

Os métodos abstratos devem estar anotados com o tipo de operação que se pretende


### RoomDatabase

A classe que representa a base de dados. Deve ser abstracta e sub-classe de RoomDatabase.

Além disso é necessário anotar esta class com `@Database`. Ambos os argumentos `entities` e `version` são obrigatórios.

No argumento `entities` passamos uma lista das classes que representam entidades da BD.

```kt
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

                instance = Room.databaseBuilder(context.applicationContext, AppDatabase::class.java, "app.db")
                    .allowMainThreadQueries()
                    .fallbackToDestructiveMigration()
                    .build()

                return instance!!
            }
        }
    }
}
```
