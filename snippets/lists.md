# Listas com RecyclerView
<br>
<br>

Exemplo de um `ViewHolder`
```kt
class ContactViewHolder(view: View) : ViewHolder(view) {

        private var contact: Contact? = null
        private val binding = ContactListItemBinding.bind(view)

        init {


            view.setOnClickListener {
                // clique no item da lista
            }

            view.setOnLongClickListener {
                
                // clique longo no item da lista

                return@setOnLongClickListener true
            }


        }

        fun bind(contact: Contact) {
            this.contact = contact
            // colocar conteúdo nas Views
        }
    }
```
<br>

---
<br>

### ListAdapter\<ItemType, ViewHolderType\>

Exemplo de um `ListAdapter`
```kt
class ContactsListAdapter : ListAdapter<Contact, ContactViewHolder>(differ) {
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ContactViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.contact_list_item, parent, false)
        return ContactViewHolder(view)
    }

    override fun onBindViewHolder(holder: ContactViewHolder, position: Int) {
        val contact = getItem(position)
        holder.bind(contact)
    }
}
```
<br>

---
<br>

### RecyclerView.Adapter\<ViewHolderType\>
Exemplo de um `RecyclerView.Adapter`
```kt
class TodoListAdapter : RecyclerView.Adapter<TodoViewHolder>() {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): TodoViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.todo_list_item, parent, false)
        return TodoViewHolder(view)
    }

    override fun onBindViewHolder(holder: TodoViewHolder, position: Int) {
        val todo = TodoDatasource.todos[position]
        holder.bind(todo)
    }

    override fun getItemCount() = TodoDatasource.todos.size
}
```
<br>

---
<br>

Configuração de um Adapter (serve para RecyclerView.Adapter e ListAdapter) com uma RecyclerView
> ⚠ Assumindo que está a usar o view binding e que existe uma RecyclerView com o id 'todoList' no layout do Fragment/Activity
```kt
val adapter = TodoListAdapter()

binding.todoList.adapter = adapter
binding.todoList.layoutManager = LinearLayoutManager(requireContext())
// se estiver a utilizar o ListAdapter pode submeter uma lista de dados com: adapter.submitList(...)
```