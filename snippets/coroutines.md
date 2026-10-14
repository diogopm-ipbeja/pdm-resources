# Kotlin Coroutines

[Kotlin coroutines on Android](https://developer.android.com/kotlin/coroutines)

> ℹ️ Não é preciso adicionar dependências: as coroutines já vêm incluídas com as bibliotecas `lifecycle` e `fragment` do [libs.versions.toml](../projectConfiguration/libs.versions.toml).

<br>

## Conceitos

Na thread principal (**main thread** ou **UI thread**) desenha-se a interface e respondem-se aos cliques. Se uma operação demorada (base de dados, ficheiros, rede) correr nessa thread, a app bloqueia. Se bloquear durante vários segundos, o Android mostra o erro *"App isn't responding"* (ANR).

As **coroutines** permitem escrever código assíncrono como se fosse sequencial, sem bloquear a thread principal.

| Conceito | O que é |
|---|---|
| `suspend fun` | Função que pode **suspender** (pausar sem bloquear a thread) e continuar mais tarde. Só pode ser chamada dentro de uma coroutine ou de outra função `suspend` |
| Coroutine | "Tarefa leve" que executa código `suspend`. É criada com `launch` ou `async` |
| Scope | Define o **tempo de vida** das coroutines. Quando o scope termina, as suas coroutines são canceladas automaticamente |
| `launch { }` | Inicia uma coroutine que **não devolve resultado** ("dispara e esquece") |
| `async { }` + `await()` | Inicia uma coroutine que **devolve um resultado**. Útil para fazer várias coisas em paralelo |
| Dispatcher | Define **em que thread(s)** o código corre |
| `withContext(Dispatchers.X) { }` | Muda de dispatcher só para aquele bloco e devolve o resultado |
| `Flow` | Sequência de valores ao longo do tempo (ex.: o resultado de uma query sempre que a tabela muda) |

### Scopes no Android

| Scope | Onde | Cancelado quando |
|---|---|---|
| `lifecycleScope` | Activity | A Activity é destruída |
| `viewLifecycleOwner.lifecycleScope` | Fragment | A **view** do Fragment é destruída |
| `viewModelScope` | ViewModel | O ViewModel é destruído |

> ⚠ Num Fragment usar sempre `viewLifecycleOwner.lifecycleScope` (e não só `lifecycleScope`) quando o código mexe nas Views.

### Dispatchers

| Dispatcher | Para quê |
|---|---|
| `Dispatchers.Main` | Atualizar a UI. É o dispatcher por omissão do `lifecycleScope` e do `viewModelScope` |
| `Dispatchers.IO` | Ler/escrever ficheiros, SharedPreferences, rede, base de dados |
| `Dispatchers.Default` | Cálculos pesados (CPU): ordenar listas grandes, processar imagens, etc. |

<br>

---

## Primeira coroutine (Fragment)

```kt
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)

    binding.button.setOnClickListener {
        viewLifecycleOwner.lifecycleScope.launch {
            // aqui estamos na main thread
            val result = withContext(Dispatchers.IO) {
                // aqui estamos numa thread de IO: trabalho demorado
                "resultado"
            }
            // de volta à main thread: podemos atualizar a UI
            binding.text.text = result
        }
    }
}
```

> `delay(1000)` suspende a coroutine durante 1 segundo **sem bloquear** a thread. Nunca usar `Thread.sleep(...)` dentro de coroutines.

<br>

---

## Room

O Room (2.x) suporta funções `suspend` e `Flow` no DAO. O próprio Room executa as queries fora da main thread, por isso **não é preciso** `withContext(Dispatchers.IO)` nem `allowMainThreadQueries()`.

```kt
@Dao
interface VehicleDao {

    @Query("select * from vehicle")
    suspend fun getAll() : List<Vehicle>

    @Query("select * from vehicle")
    fun observeAll() : Flow<List<Vehicle>> // sem suspend: emite uma nova lista sempre que a tabela muda

    @Insert
    suspend fun insert(vehicle: Vehicle) : Long
}
```

```kt
val dao = AppDatabase(requireContext()).vehicleDao()

viewLifecycleOwner.lifecycleScope.launch {
    dao.insert(Vehicle("Renault", "Clio", 2020))
    val vehicles = dao.getAll()
    adapter.submitList(vehicles)
}
```

Ver também: [Room Database](room-database.md)

<br>

---

## Ficheiros

Ler e escrever ficheiros é uma operação de IO, por isso usa-se `withContext(Dispatchers.IO)`.

```kt
suspend fun writeNote(context: Context, text: String) = withContext(Dispatchers.IO) {
    File(context.filesDir, "note.txt").writeText(text)
}

suspend fun readNote(context: Context): String = withContext(Dispatchers.IO) {
    val file = File(context.filesDir, "note.txt")
    if (file.exists()) file.readText() else ""
}
```

```kt
viewLifecycleOwner.lifecycleScope.launch {
    writeNote(requireContext(), binding.noteInput.text.toString())
    binding.noteText.text = readNote(requireContext())
}
```

> `context.filesDir` é uma pasta privada da app. Não precisa de permissões.

<br>

---

## SharedPreferences

Para guardar valores simples (chave → valor), como definições do utilizador. Na primeira utilização o ficheiro é lido do disco, por isso também se usa `Dispatchers.IO`.

```kt
suspend fun saveUsername(context: Context, name: String) = withContext(Dispatchers.IO) {
    context.getSharedPreferences("settings", Context.MODE_PRIVATE).edit {
        putString("username", name)
    }
}

suspend fun getUsername(context: Context): String? = withContext(Dispatchers.IO) {
    context.getSharedPreferences("settings", Context.MODE_PRIVATE).getString("username", null)
}
```

```kt
viewLifecycleOwner.lifecycleScope.launch {
    saveUsername(requireContext(), "Ana")
    binding.username.text = getUsername(requireContext())
}
```

> O `edit { }` (do `androidx.core:core-ktx`) chama `apply()` no fim, que grava no disco em segundo plano.
>
> A alternativa mais recente às SharedPreferences é o [DataStore](https://developer.android.com/topic/libraries/architecture/datastore), que já é baseado em coroutines e `Flow`.

<br>

---

## Cálculos pesados

```kt
suspend fun countPrimes(n: Int): Int = withContext(Dispatchers.Default) {
    (2..n).count { x -> (2..x / 2).none { x % it == 0 } }
}
```

<br>

---

## Várias tarefas em paralelo (`async`)

Com `launch` e chamadas seguidas, a segunda tarefa só começa quando a primeira termina. Com `async` as duas correm ao mesmo tempo:

```kt
viewLifecycleOwner.lifecycleScope.launch {
    val user = async { loadUser() }    // começa já
    val posts = async { loadPosts() }  // começa já, sem esperar pelo loadUser
    binding.text.text = "${user.await()} tem ${posts.await().size} posts" // espera pelos dois
}
```

<br>

---

## ViewModel

No ViewModel usa-se o `viewModelScope`. As coroutines continuam a correr quando o ecrã roda e só são canceladas quando o ViewModel é destruído.

```kt
class VehiclesViewModel(application: Application) : AndroidViewModel(application) {

    private val dao = AppDatabase(application).vehicleDao()

    // Flow do Room convertido em LiveData
    val vehicles: LiveData<List<Vehicle>> = dao.observeAll().asLiveData()

    fun add(vehicle: Vehicle) {
        viewModelScope.launch {
            dao.insert(vehicle)
        }
    }
}
```

```kt
class VehiclesFragment : Fragment() {

    private val viewModel: VehiclesViewModel by viewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        viewModel.vehicles.observe(viewLifecycleOwner) { vehicles ->
            adapter.submitList(vehicles)
        }

        binding.addButton.setOnClickListener {
            viewModel.add(Vehicle("Renault", "Clio", 2020))
        }
    }
}
```

<br>

---

## Observar um `Flow` num Fragment

Em alternativa ao `LiveData`, pode-se recolher (`collect`) o `Flow` diretamente. O `repeatOnLifecycle` pára a recolha quando o Fragment não está visível e retoma quando volta a estar.

```kt
viewLifecycleOwner.lifecycleScope.launch {
    viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
        dao.observeAll().collect { vehicles ->
            adapter.submitList(vehicles)
        }
    }
}
```

<br>

---

## Erros e cancelamento

Os erros tratam-se com `try`/`catch`, como no código normal:

```kt
viewLifecycleOwner.lifecycleScope.launch {
    try {
        writeNote(requireContext(), "olá")
    } catch (e: IOException) {
        Toast.makeText(requireContext(), "Erro ao gravar", Toast.LENGTH_SHORT).show()
    }
}
```

O `launch` devolve um `Job`, que permite cancelar a coroutine:

```kt
val job = viewLifecycleOwner.lifecycleScope.launch {
    while (isActive) {      // false depois de cancelada
        // atualizar algo a cada segundo
        delay(1000)
    }
}

job.cancel()
```

<br>

---

## Erros comuns

- ❌ `GlobalScope.launch { }`: a coroutine não é cancelada quando o ecrã é fechado. Usar `lifecycleScope` ou `viewModelScope`.
- ❌ `runBlocking { }` na app: bloqueia a thread, o que anula a vantagem das coroutines.
- ❌ `Thread.sleep(...)`: bloqueia a thread. Usar `delay(...)`.
- ❌ Atualizar Views dentro de `withContext(Dispatchers.IO) { }`: a UI só pode ser alterada na main thread. Devolver o valor e atualizar depois do `withContext`.
