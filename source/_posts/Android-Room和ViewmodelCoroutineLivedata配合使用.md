---

title: Android-Room和ViewmodelCoroutineLivedata配合使用
date: 2021-12-18 12:41:06
tags: Android

---



<img src="/Users/xuedi/Desktop/blog/source/images/roomviewmodelMain.png" style="zoom:25%;" />

****

这是小项目的样子，主界面是一个recyclerView从room取出的所有数据并使用livedata观察，左边floatiingButton点击清除room数据库，右边floatingButton点击进入存储数据界面insert表。



# db

room数据库所有操作

## WordRoomDatabase

> 1、创建abstract WordRoomDatabase继承RoomDatabase，create abstract fun wordDao()
>
> 2、dcl单例wordRoomDatabase instance by fun getDatabase
>
> 3、addCallback()，wordDatabaseCallback override fun onCreate use coroutine function launch()对room init。deleteAll() insert() insert()

```kotlin
@Database(entities = arrayOf(Word::class), version = 1, exportSchema = false)
abstract class WordRoomDatabase : RoomDatabase() {
    abstract fun wordDao(): WordDao

    companion object {
        @Volatile
        private var INSTANCE: WordRoomDatabase? = null
        fun getDatabase(context: Context, scope: CoroutineScope): WordRoomDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    WordRoomDatabase::class.java,
                    "word_database"
                ).addCallback(WordDatabaseCallback(scope)).build()
                INSTANCE = instance
                instance
            }
        }
    }

    private class WordDatabaseCallback(private val scope: CoroutineScope) :
        RoomDatabase.Callback() {

        override fun onCreate(db: SupportSQLiteDatabase) {
            super.onCreate(db)
            INSTANCE?.let { wordRoomDatabase ->
                scope.launch {
                    populateDatabase(wordRoomDatabase.wordDao())
                }
            }
        }

        suspend fun populateDatabase(wordDao: WordDao) {
            //deleteAll
            wordDao.deleteAll()
            //Add words
            wordDao.insert(Word("BE BRAVE DI"))
            wordDao.insert(Word("DO NOT GIVEUP"))
        }
    }
}
```

## Word

> word表。tableName 设置表名称，ColumnInfo设置每一列名称，在这里只有word和autoGenerate id

```kotlin
@Entity(tableName = "work_table")
data class Word(
    @ColumnInfo(name = "word")
    var word: String = ""
) {
    @PrimaryKey(autoGenerate = true)
    var id: Int = 0
}
```

## WordDao

> 对word表进行的sql操作。

```kotlin
@Dao
interface WordDao {

    @Query("SELECT * FROM work_table ORDER BY word ASC")
    fun getAlphabetizedWords() :Flow<List<Word>>

    @Insert(onConflict = OnConflictStrategy.IGNORE)
    suspend fun insert(word: Word)

    @Query("DELETE FROM work_table")
    suspend fun deleteAll()
}
```

# repo

WordRepository是对db和net进行的统一管理，调用wordDao.function

```kotlin
class WordRepository(private val wordDao: WordDao) {
    val allWords: Flow<List<Word>> = wordDao.getAlphabetizedWords()

    @Suppress("RedundantSuspendModifier")
    @WorkerThread
    suspend fun insert(word: Word){
        wordDao.insert(word)
    }

    @WorkerThread
    suspend fun deleteAll(){
        wordDao.deleteAll()
    }
}
```

# viewmodel

viewmodel对activity界面数据保存

## WordViewModelFactory

> Return WordViewModel(repository) as T

```kotlin
class WordViewModelFactory(private val repository: WordRepository) : ViewModelProvider.Factory {
    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(WordViewModel::class.java)){
            @Suppress("UNCHECK_CAST")
            return WordViewModel(repository) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}
```

## WordViewModel

> allWords()使用Flow.asLiveData
>
> Insert()使用协程viewmodelscope.launch()
>
> deleteAdd()同上

```kotlin
class WordViewModel(private var repository: WordRepository) : ViewModel() {

    val allWords:LiveData<List<Word>> = repository.allWords.asLiveData()

    fun insert(word: Word) = viewModelScope.launch {
        repository.insert(word)
    }

    fun deleteAdd() = viewModelScope.launch {
        repository.deleteAll()
    }
}
```

# adapter

## WordListAdapter

```kotlin
class WordListAdapter : ListAdapter<Word, WordListAdapter.WordViewHolder>(WordsComparator()) {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): WordViewHolder {
        return WordViewHolder.create(parent)
    }

    override fun onBindViewHolder(holder: WordViewHolder, position: Int) {
        val current = getItem(position)
        holder.bind(current.word)
    }

    class WordViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        private val wordItemView: TextView = itemView.findViewById(R.id.textView)
        private val wordItemContent: ConstraintLayout = itemView.findViewById(R.id.recyclerviewcontent)
        fun bind(text: String?) {
            wordItemView.text = text
            val colorArr = arrayOf(Color.WHITE, Color.YELLOW, Color.GREEN, Color.RED)
            wordItemContent.setBackgroundColor(colorArr[Random.nextInt(colorArr.size)])
        }

        companion object {
            fun create(parent: ViewGroup): WordViewHolder {
                val view: View = LayoutInflater.from(parent.context)
                    .inflate(R.layout.recyclerview_item, parent, false)
                return WordViewHolder(view)
            }
        }
    }

    class WordsComparator : DiffUtil.ItemCallback<Word>() {
        override fun areItemsTheSame(oldItem: Word, newItem: Word): Boolean {
            return oldItem === newItem
        }

        override fun areContentsTheSame(oldItem: Word, newItem: Word): Boolean {
            return oldItem.word == newItem.word
        }

    }
}
```

# Activity and application

##  WordsApplication

> By lazy调用到在初始化
>
> Init function: WordRoomDatabase.getDatabase(this,scope), WordRepository(database.wordDao)

```kotlin
class WordsApplication : Application() {
    val applicationScope = CoroutineScope(SupervisorJob())
    val database by lazy { WordRoomDatabase.getDatabase(this, scope = applicationScope) }
    val repository by lazy { WordRepository(database.wordDao()) }
}
```

## NewWordActivity

> 存储word表

```kotlin
class NewWordActivity : AppCompatActivity() {
    private lateinit var binding:ActivityNewWordBinding
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityNewWordBinding.inflate(layoutInflater)
        setContentView(binding.root)
        binding.buttonSave.setOnClickListener {
            val replyIntent = Intent()
            if (TextUtils.isEmpty(binding.editWord.text)){
                setResult(Activity.RESULT_CANCELED, replyIntent)
            } else {
                Log.d("DI", binding.editWord.text.toString())
                replyIntent.putExtra(EXTRA_REPLY, binding.editWord.text.toString())
                setResult(Activity.RESULT_OK, replyIntent)
            }
            finish()
        }
    }

    companion object{
        const val EXTRA_REPLY = "com.example.android.wordlistsqp.REPLY"
    }
}
```

## MainActivity

> first       ---> viewModel. ''Viewmodel include liveData scope''
>
> Second ---> livedata.observe() 数据变化则adapter跟着变化.

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    private val newWOrdActivityRequestCode = 1
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        var adapter = WordListAdapter()
        binding.recyclerview.adapter = adapter
        binding.recyclerview.layoutManager = LinearLayoutManager(this)
        wordViewModel.allWords.observe(this, Observer { wordViewModel ->
            wordViewModel?.let {
                adapter.submitList(it)
            }
        })
        binding.floatingActionButton.setOnClickListener{
            val intent = Intent(this, NewWordActivity::class.java)
            startActivityForResult(intent, newWOrdActivityRequestCode)
        }
        binding.clear.setOnClickListener{
            wordViewModel.deleteAdd()
        }
    }

    private val wordViewModel: WordViewModel by viewModels {
        WordViewModelFactory((application as WordsApplication).repository)
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        if (requestCode == newWOrdActivityRequestCode && resultCode == Activity.RESULT_OK) {
            data?.getStringExtra(NewWordActivity.EXTRA_REPLY)?.let {
                wordViewModel.insert(Word(it))
            }
        } else {
            Snackbar.make(binding.content, "没有数据返回哦", Snackbar.LENGTH_LONG).show()
        }
    }
}
```
