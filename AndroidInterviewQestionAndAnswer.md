# Android Interview Questions & Answers

## Table of Contents

- [Basic Concepts](#basic-concepts)
- [Core Android Components](#core-android-components)
  - [Activity](#activity)
  - [Fragment](#fragment)
  - [Services](#services)
  - [BroadcastReceiver](#broadcastreceiver)
- [Views and UI](#views-and-ui)
- [Lists and RecyclerView](#lists-and-recyclerview)
- [Intents and Broadcasting](#intents-and-broadcasting)
- [Background Processing](#background-processing)
- [Data Storage](#data-storage)
- [Memory and Performance](#memory-and-performance)
- [Architecture Patterns](#architecture-patterns)
- [Testing](#testing)
- [Tools and Technologies](#tools-and-technologies)
- [Advanced Topics](#advanced-topics)

---

## Basic Concepts

### Q: Can you tell me the lifecycle methods of an Activity? What order are they called in?

**Answer:**
The Activity lifecycle methods are called in the following order:

1. **onCreate()** - Called when the activity is first created
2. **onStart()** - Called when the activity becomes visible to the user
3. **onResume()** - Called when the activity starts interacting with the user
4. **onPause()** - Called when the system is about to start resuming another activity
5. **onStop()** - Called when the activity is no longer visible to the user
6. **onDestroy()** - Called before the activity is destroyed

**Complete lifecycle flow:**
```
onCreate() → onStart() → onResume() → [Activity Running] → onPause() → onStop() → onDestroy()
```

**Partial lifecycle (when returning to activity):**
```
onStop() → onRestart() → onStart() → onResume()
```

### Q: What happens to an Activity when a device is rotated from portrait to landscape? Why?

**Answer:**
When a device is rotated:

1. **Current Activity is destroyed**: The system calls `onPause()` → `onStop()` → `onDestroy()`
2. **New Activity instance is created**: The system calls `onCreate()` → `onStart()` → `onResume()`
3. **Configuration change occurs**: The system detects a configuration change and recreates the activity with new resources

**Why this happens:**
- Android treats rotation as a configuration change
- Different orientations may require different layouts, resources, or dimensions
- The system needs to reload resources optimized for the new orientation

**How to handle it:**
```kotlin
// Save state in onSaveInstanceState
override fun onSaveInstanceState(outState: Bundle) {
    super.onSaveInstanceState(outState)
    outState.putString("key", value)
}

// Restore state in onCreate
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    if (savedInstanceState != null) {
        val value = savedInstanceState.getString("key")
    }
}
```

### Q: What is Context? How is it used?

**Answer:**
Context is an abstract class that provides access to application-specific resources and classes, as well as calls for application-level operations.

**Types of Context:**
1. **Application Context** - Tied to application lifecycle
2. **Activity Context** - Tied to activity lifecycle
3. **Service Context** - Tied to service lifecycle

**Common uses:**
- Starting activities
- Accessing resources (strings, drawables, etc.)
- Accessing system services
- Creating views
- Accessing databases

```kotlin
// Examples of Context usage
val context = this // In Activity
val appContext = applicationContext

// Starting an activity
val intent = Intent(context, MainActivity::class.java)
context.startActivity(intent)

// Accessing resources
val string = context.getString(R.string.app_name)
val drawable = ContextCompat.getDrawable(context, R.drawable.icon)

// Getting system services
val layoutInflater = context.getSystemService(Context.LAYOUT_INFLATER_SERVICE) as LayoutInflater
```

---

## Core Android Components

### Activity

### Q: What is Activity?

**Answer:**
An Activity represents a single screen with a user interface. It's one of the four main components of an Android application.

**Key characteristics:**
- Extends from `Activity` or `AppCompatActivity`
- Has its own lifecycle
- Can be started by other activities or the system
- Declared in AndroidManifest.xml

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}
```

### Q: What are "launch modes"?

**Answer:**
Launch modes determine how a new instance of an activity is associated with the current task.

**Four launch modes:**

1. **standard** (default)
   - Creates a new instance every time
   - Multiple instances can exist

2. **singleTop**
   - Reuses existing instance if it's at the top of the stack
   - Calls `onNewIntent()` instead of creating new instance

3. **singleTask**
   - Only one instance exists in the system
   - Creates new task if needed
   - Clears activities above it if it exists

4. **singleInstance**
   - Only one instance exists globally
   - Runs in its own task
   - No other activities can be part of its task

```xml
<!-- In AndroidManifest.xml -->
<activity
    android:name=".MainActivity"
    android:launchMode="singleTop" />
```

### Fragment

### Q: What is Fragment?

**Answer:**
A Fragment represents a reusable portion of your app's UI. It defines and manages its own layout, has its own lifecycle, and can handle its own input events.

**Key benefits:**
- **Modularity** - Encapsulate UI components
- **Reusability** - Use same fragment in multiple activities
- **Adaptability** - Different layouts for different screen sizes

```kotlin
class MyFragment : Fragment() {
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.fragment_my, container, false)
    }
}
```

### Q: What is the difference between a Fragment and an Activity?

**Answer:**

| Aspect | Activity | Fragment |
|--------|----------|----------|
| **Independence** | Can exist independently | Must be hosted by an Activity |
| **Lifecycle** | Has its own complete lifecycle | Lifecycle tied to host Activity |
| **Context** | Is a Context | Not a Context (use getContext()) |
| **Back Stack** | Managed by system | Managed by FragmentManager |
| **Declaration** | Must be declared in manifest | No manifest declaration needed |
| **Purpose** | Represents a screen | Represents a portion of UI |

### Q: Why is it recommended to use only the default constructor to create a Fragment?

**Answer:**
The Android system recreates fragments during configuration changes or when restoring state. It uses reflection to call the default constructor.

**Wrong approach:**
```kotlin
// DON'T DO THIS
class MyFragment(private val data: String) : Fragment() {
    // This will crash when system recreates the fragment
}
```

**Correct approach:**
```kotlin
class MyFragment : Fragment() {
    companion object {
        fun newInstance(data: String): MyFragment {
            val fragment = MyFragment()
            val args = Bundle()
            args.putString("data", data)
            fragment.arguments = args
            return fragment
        }
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val data = arguments?.getString("data")
    }
}
```

### Q: How would you communicate between two Fragments?

**Answer:**
There are several ways to communicate between fragments:

**1. Through the host Activity (Traditional approach):**
```kotlin
// In Fragment A
interface OnDataPassListener {
    fun onDataPass(data: String)
}

class FragmentA : Fragment() {
    private var listener: OnDataPassListener? = null
    
    override fun onAttach(context: Context) {
        super.onAttach(context)
        listener = context as OnDataPassListener
    }
    
    private fun sendData() {
        listener?.onDataPass("Hello from Fragment A")
    }
}

// In Activity
class MainActivity : AppCompatActivity(), OnDataPassListener {
    override fun onDataPass(data: String) {
        val fragmentB = supportFragmentManager.findFragmentByTag("FragmentB") as FragmentB
        fragmentB.receiveData(data)
    }
}
```

**2. Using ViewModel (Recommended approach):**
```kotlin
class SharedViewModel : ViewModel() {
    private val _data = MutableLiveData<String>()
    val data: LiveData<String> = _data
    
    fun setData(value: String) {
        _data.value = value
    }
}

// In both fragments
class FragmentA : Fragment() {
    private lateinit var sharedViewModel: SharedViewModel
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        sharedViewModel = ViewModelProvider(requireActivity())[SharedViewModel::class.java]
        
        // Send data
        sharedViewModel.setData("Hello from Fragment A")
    }
}

class FragmentB : Fragment() {
    private lateinit var sharedViewModel: SharedViewModel
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        sharedViewModel = ViewModelProvider(requireActivity())[SharedViewModel::class.java]
        
        // Observe data
        sharedViewModel.data.observe(viewLifecycleOwner) { data ->
            // Handle received data
        }
    }
}
```

### Services

### Q: What is Service?

**Answer:**
A Service is an application component that can perform long-running operations in the background without providing a user interface.

**Types of Services:**

1. **Foreground Service**
   - Performs operations noticeable to the user
   - Must display a persistent notification
   - Continues running even when user isn't interacting with the app

2. **Background Service**
   - Performs operations not directly noticed by the user
   - Subject to background execution limits

3. **Bound Service**
   - Provides a client-server interface
   - Allows components to bind to it, send requests, receive results

```kotlin
class MyService : Service() {
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        // Perform background work
        return START_STICKY
    }
    
    override fun onBind(intent: Intent?): IBinder? {
        return null // For started service
    }
}
```

### Q: Service vs IntentService

**Answer:**

| Aspect | Service | IntentService |
|--------|---------|---------------|
| **Thread** | Runs on main thread | Runs on worker thread |
| **Multiple requests** | Must handle manually | Handles sequentially |
| **Stopping** | Must call stopSelf() | Stops automatically |
| **Implementation** | More complex | Simpler for background tasks |

```kotlin
// Service - runs on main thread
class MyService : Service() {
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        // This runs on main thread - need to create worker thread
        Thread {
            // Background work
            stopSelf() // Must stop manually
        }.start()
        return START_STICKY
    }
}

// IntentService - automatically uses worker thread
class MyIntentService : IntentService("MyIntentService") {
    override fun onHandleIntent(intent: Intent?) {
        // This runs on worker thread automatically
        // Service stops automatically when work is done
    }
}
```

### BroadcastReceiver

### Q: What is a BroadcastReceiver?

**Answer:**
A BroadcastReceiver is a component that responds to system-wide broadcast announcements or custom broadcasts from other applications.

**Types of broadcasts:**
1. **System broadcasts** - Battery low, network changes, boot completed
2. **Custom broadcasts** - Sent by your app or other apps

**Registration methods:**
1. **Static registration** (in manifest)
2. **Dynamic registration** (in code)

```kotlin
// BroadcastReceiver implementation
class MyBroadcastReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context?, intent: Intent?) {
        when (intent?.action) {
            Intent.ACTION_BATTERY_LOW -> {
                // Handle battery low
            }
            "com.example.CUSTOM_ACTION" -> {
                // Handle custom broadcast
            }
        }
    }
}

// Static registration in manifest
<receiver android:name=".MyBroadcastReceiver">
    <intent-filter>
        <action android:name="android.intent.action.BATTERY_LOW" />
    </intent-filter>
</receiver>

// Dynamic registration
val receiver = MyBroadcastReceiver()
val filter = IntentFilter(Intent.ACTION_BATTERY_LOW)
registerReceiver(receiver, filter)
```

---

## Views and UI

### Q: What is View in Android?

**Answer:**
View is the basic building block for user interface components. It occupies a rectangular area on the screen and is responsible for drawing and event handling.

**Key characteristics:**
- All UI components inherit from View
- Has properties like width, height, padding, margin
- Can handle touch events
- Participates in the layout and drawing process

**View hierarchy:**
```
ViewGroup (Container)
├── View (Child)
├── ViewGroup (Child container)
│   ├── View (Grandchild)
│   └── View (Grandchild)
└── View (Child)
```

### Q: Difference between View.GONE and View.INVISIBLE?

**Answer:**

| Aspect | View.INVISIBLE | View.GONE |
|--------|----------------|-----------|
| **Space occupation** | Takes up space in layout | Doesn't take up space |
| **Drawing** | Not drawn | Not drawn |
| **Layout calculation** | Included in layout | Excluded from layout |
| **Performance** | Faster to toggle | May cause layout recalculation |

```kotlin
// INVISIBLE - view is hidden but space is reserved
view.visibility = View.INVISIBLE

// GONE - view is hidden and no space is reserved
view.visibility = View.GONE

// VISIBLE - view is shown
view.visibility = View.VISIBLE
```

### Q: Can you create custom views? How?

**Answer:**
Yes, you can create custom views by extending existing View classes or creating completely new ones.

**Approaches:**

1. **Extend existing View:**
```kotlin
class CustomButton : Button {
    constructor(context: Context) : super(context)
    constructor(context: Context, attrs: AttributeSet) : super(context, attrs)
    
    override fun onDraw(canvas: Canvas?) {
        super.onDraw(canvas)
        // Custom drawing code
    }
}
```

2. **Extend View directly:**
```kotlin
class CustomView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {
    
    private val paint = Paint().apply {
        color = Color.BLUE
        strokeWidth = 5f
    }
    
    override fun onDraw(canvas: Canvas?) {
        super.onDraw(canvas)
        canvas?.drawCircle(width/2f, height/2f, 100f, paint)
    }
    
    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        val desiredWidth = 200
        val desiredHeight = 200
        
        val width = resolveSize(desiredWidth, widthMeasureSpec)
        val height = resolveSize(desiredHeight, heightMeasureSpec)
        
        setMeasuredDimension(width, height)
    }
}
```

### Q: Relative Layout vs Linear Layout

**Answer:**

| Aspect | LinearLayout | RelativeLayout |
|--------|--------------|----------------|
| **Arrangement** | Sequential (horizontal/vertical) | Relative positioning |
| **Performance** | Better for simple layouts | Can be slower with complex nesting |
| **Flexibility** | Limited positioning options | Very flexible positioning |
| **Nesting** | Often requires nesting | Can reduce nesting |

**LinearLayout:**
```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical">
    
    <TextView android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:text="First" />
              
    <TextView android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:text="Second" />
</LinearLayout>
```

**RelativeLayout:**
```xml
<RelativeLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    
    <TextView
        android:id="@+id/first"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:text="Center" />
        
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/first"
        android:layout_centerHorizontal="true"
        android:text="Below center" />
</RelativeLayout>
```

### Q: Tell about Constraint Layout

**Answer:**
ConstraintLayout is a flexible layout that allows you to create complex layouts with a flat view hierarchy, improving performance.

**Key features:**
- **Flat hierarchy** - Reduces nested layouts
- **Flexible positioning** - Position views relative to parent or other views
- **Performance** - Better than nested LinearLayouts
- **Responsive design** - Adapts to different screen sizes

**Basic constraints:**
```xml
<androidx.constraintlayout.widget.ConstraintLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    
    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />
        
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Click me"
        app:layout_constraintTop_toBottomOf="@id/textView"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginTop="16dp" />
        
</androidx.constraintlayout.widget.ConstraintLayout>
```

---

## Lists and RecyclerView

### Q: What is the difference between ListView and RecyclerView?

**Answer:**

| Feature | ListView | RecyclerView |
|---------|----------|--------------|
| **ViewHolder** | Optional (manual implementation) | Mandatory |
| **Layout Manager** | Only vertical scrolling | Flexible (Linear, Grid, Staggered) |
| **Animations** | Limited | Rich animation support |
| **Performance** | Good | Better (view recycling) |
| **Flexibility** | Less flexible | Highly customizable |
| **Item decorations** | Limited | Extensive decoration support |

### Q: What is the ViewHolder pattern? Why should we use it?

**Answer:**
The ViewHolder pattern is a design pattern used to improve the performance of list views by caching view references.

**Without ViewHolder (inefficient):**
```kotlin
// DON'T DO THIS
override fun getView(position: Int, convertView: View?, parent: ViewGroup): View {
    val view = convertView ?: layoutInflater.inflate(R.layout.item_layout, parent, false)
    
    // findViewById is called every time - expensive!
    val textView = view.findViewById<TextView>(R.id.textView)
    val imageView = view.findViewById<ImageView>(R.id.imageView)
    
    textView.text = items[position].title
    imageView.setImageResource(items[position].imageRes)
    
    return view
}
```

**With ViewHolder (efficient):**
```kotlin
class MyAdapter : RecyclerView.Adapter<MyAdapter.ViewHolder>() {
    
    class ViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val textView: TextView = itemView.findViewById(R.id.textView)
        val imageView: ImageView = itemView.findViewById(R.id.imageView)
    }
    
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_layout, parent, false)
        return ViewHolder(view)
    }
    
    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        // Views are already cached in ViewHolder
        holder.textView.text = items[position].title
        holder.imageView.setImageResource(items[position].imageRes)
    }
}
```

**Benefits:**
- **Performance** - Avoids expensive findViewById calls
- **Memory efficiency** - Reuses view objects
- **Smooth scrolling** - Reduces UI thread work

---

## Intents and Broadcasting

### Q: What is Intent?

**Answer:**
An Intent is a messaging object used to request an action from another app component. It facilitates communication between components.

**Types of Intents:**

1. **Explicit Intent** - Specifies the exact component to start
2. **Implicit Intent** - Declares a general action to perform

**Common uses:**
- Starting activities
- Starting services
- Delivering broadcasts
- Passing data between components

```kotlin
// Explicit Intent
val intent = Intent(this, SecondActivity::class.java)
intent.putExtra("key", "value")
startActivity(intent)

// Implicit Intent
val intent = Intent(Intent.ACTION_VIEW, Uri.parse("https://www.google.com"))
startActivity(intent)
```

### Q: What is an Implicit Intent?

**Answer:**
An Implicit Intent doesn't specify the exact component to start. Instead, it declares a general action to perform, allowing other apps to handle it.

**Examples:**
```kotlin
// Open a web page
val webIntent = Intent(Intent.ACTION_VIEW, Uri.parse("https://www.example.com"))
startActivity(webIntent)

// Send an email
val emailIntent = Intent(Intent.ACTION_SEND).apply {
    type = "text/plain"
    putExtra(Intent.EXTRA_EMAIL, arrayOf("recipient@example.com"))
    putExtra(Intent.EXTRA_SUBJECT, "Subject")
    putExtra(Intent.EXTRA_TEXT, "Email body")
}
startActivity(Intent.createChooser(emailIntent, "Send email"))

// Make a phone call
val callIntent = Intent(Intent.ACTION_CALL, Uri.parse("tel:+1234567890"))
startActivity(callIntent)

// Take a photo
val photoIntent = Intent(MediaStore.ACTION_IMAGE_CAPTURE)
startActivityForResult(photoIntent, REQUEST_IMAGE_CAPTURE)
```

### Q: What is a PendingIntent?

**Answer:**
A PendingIntent is a wrapper around an Intent that grants permission to a foreign application to use the contained Intent as if it were executed from your app's process.

**Key characteristics:**
- **Delayed execution** - Intent is executed later
- **Permission delegation** - Other apps can execute with your permissions
- **Immutable by default** (Android 12+)

**Common uses:**
- Notifications
- App widgets
- Alarm Manager

```kotlin
// For notification
val intent = Intent(this, MainActivity::class.java)
val pendingIntent = PendingIntent.getActivity(
    this, 
    0, 
    intent, 
    PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
)

val notification = NotificationCompat.Builder(this, CHANNEL_ID)
    .setContentTitle("Title")
    .setContentText("Content")
    .setSmallIcon(R.drawable.ic_notification)
    .setContentIntent(pendingIntent)
    .build()
```

---

## Background Processing

### Q: How would you perform a long-running operation in an application?

**Answer:**
There are several approaches for long-running operations, each suitable for different scenarios:

**1. Kotlin Coroutines (Modern approach):**
```kotlin
class MainActivity : AppCompatActivity() {
    
    private fun performBackgroundTask() {
        lifecycleScope.launch {
            val result = withContext(Dispatchers.IO) {
                doLongRunningOperation()
            }
            updateUI(result) // Automatically on main thread
        }
    }
}
```

**2. WorkManager (For deferrable tasks):**
```kotlin
class UploadWorker(context: Context, params: WorkerParameters) : Worker(context, params) {
    override fun doWork(): Result {
        return try {
            uploadData()
            Result.success()
        } catch (e: Exception) {
            Result.failure()
        }
    }
}

// Schedule work
val uploadWork = OneTimeWorkRequestBuilder<UploadWorker>().build()
WorkManager.getInstance(context).enqueue(uploadWork)
```

### Q: What is ANR? How can the ANR be prevented?

**Answer:**
ANR (Application Not Responding) occurs when the UI thread is blocked for too long.

**ANR triggers:**
- **Activity**: No response to input event within 5 seconds
- **BroadcastReceiver**: No completion within 10 seconds
- **Service**: No response within 20 seconds (foreground service)

**Prevention strategies:**

1. **Move long operations off main thread:**
```kotlin
// DON'T DO THIS
class MainActivity : AppCompatActivity() {
    fun onClick() {
        // This will cause ANR
        val data = downloadDataFromServer() // Network call on main thread
        updateUI(data)
    }
}

// DO THIS
class MainActivity : AppCompatActivity() {
    fun onClick() {
        lifecycleScope.launch {
            val data = withContext(Dispatchers.IO) {
                downloadDataFromServer() // Network call on background thread
            }
            updateUI(data) // UI update on main thread
        }
    }
}
```

### Q: Explain Looper, Handler and HandlerThread

**Answer:**

**Looper:**
- Runs a message loop for a thread
- Main thread has a Looper by default
- Background threads need to create their own Looper

**Handler:**
- Allows sending and processing messages/runnables
- Associated with a specific Looper
- Used for thread communication

**HandlerThread:**
- Thread with its own Looper
- Useful for background processing with message queue

```kotlin
// Basic Handler usage
class MainActivity : AppCompatActivity() {
    private val mainHandler = Handler(Looper.getMainLooper())
    
    fun performTask() {
        Thread {
            // Background work
            val result = doWork()
            
            // Post result to main thread
            mainHandler.post {
                updateUI(result)
            }
        }.start()
    }
}
```

---

## Data Storage

### Q: How to persist data in an Android app?

**Answer:**
Android provides several data storage options:

**1. SharedPreferences - Key-value pairs:**
```kotlin
// Save data
val sharedPref = getSharedPreferences("MyPrefs", Context.MODE_PRIVATE)
with(sharedPref.edit()) {
    putString("username", "john_doe")
    putInt("user_id", 123)
    putBoolean("is_logged_in", true)
    apply() // or commit()
}

// Read data
val username = sharedPref.getString("username", "default_value")
val userId = sharedPref.getInt("user_id", -1)
val isLoggedIn = sharedPref.getBoolean("is_logged_in", false)
```

**2. Room Database (Modern approach):**
```kotlin
// Entity
@Entity(tableName = "users")
data class User(
    @PrimaryKey(autoGenerate = true)
    val id: Int = 0,
    val name: String,
    val email: String
)

// DAO
@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    fun getAllUsers(): LiveData<List<User>>
    
    @Insert
    suspend fun insertUser(user: User)
    
    @Update
    suspend fun updateUser(user: User)
    
    @Delete
    suspend fun deleteUser(user: User)
}

// Database
@Database(entities = [User::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
    
    companion object {
        @Volatile
        private var INSTANCE: AppDatabase? = null
        
        fun getDatabase(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "app_database"
                ).build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```

### Q: How would you preserve Activity state during a screen rotation?

**Answer:**
There are several ways to preserve state during configuration changes:

**1. onSaveInstanceState() and onRestoreInstanceState():**
```kotlin
class MainActivity : AppCompatActivity() {
    private var counter = 0
    
    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        outState.putInt("counter", counter)
        outState.putString("user_input", editText.text.toString())
    }
    
    override fun onRestoreInstanceState(savedInstanceState: Bundle) {
        super.onRestoreInstanceState(savedInstanceState)
        counter = savedInstanceState.getInt("counter", 0)
        editText.setText(savedInstanceState.getString("user_input", ""))
    }
}
```

**2. ViewModel (Recommended approach):**
```kotlin
class MainViewModel : ViewModel() {
    private val _counter = MutableLiveData<Int>()
    val counter: LiveData<Int> = _counter
    
    init {
        _counter.value = 0
    }
    
    fun incrementCounter() {
        _counter.value = (_counter.value ?: 0) + 1
    }
}

class MainActivity : AppCompatActivity() {
    private lateinit var viewModel: MainViewModel
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        viewModel = ViewModelProvider(this)[MainViewModel::class.java]
        
        viewModel.counter.observe(this) { count ->
            counterTextView.text = count.toString()
        }
    }
}
```

---

## Memory and Performance

### Q: How do you find memory leaks in Android applications?

**Answer:**
Memory leaks occur when objects are held in memory longer than necessary. Here are ways to detect and fix them:

**Detection Tools:**

**1. LeakCanary (Recommended):**
```kotlin
// In build.gradle (app level)
dependencies {
    debugImplementation 'com.squareup.leakcanary:leakcanary-android:2.12'
}

// LeakCanary automatically detects leaks in debug builds
// No additional code needed
```

**Common Memory Leak Scenarios and Fixes:**

**1. Static references to Context:**
```kotlin
// DON'T DO THIS - Memory leak
class Utils {
    companion object {
        private var context: Context? = null
        
        fun init(context: Context) {
            this.context = context // Holds reference to Activity
        }
    }
}

// DO THIS - Use Application Context
class Utils {
    companion object {
        private var context: Context? = null
        
        fun init(context: Context) {
            this.context = context.applicationContext // Safe
        }
    }
}
```

**2. Non-static inner classes:**
```kotlin
// DON'T DO THIS - Inner class holds reference to outer class
class MainActivity : AppCompatActivity() {
    
    inner class AsyncTaskExample : AsyncTask<Void, Void, Void>() {
        override fun doInBackground(vararg params: Void?): Void? {
            // Long running task
            return null
        }
    }
}

// DO THIS - Use static class with WeakReference
class MainActivity : AppCompatActivity() {
    
    private class AsyncTaskExample(activity: MainActivity) : AsyncTask<Void, Void, Void>() {
        private val activityReference = WeakReference(activity)
        
        override fun doInBackground(vararg params: Void?): Void? {
            // Long running task
            return null
        }
        
        override fun onPostExecute(result: Void?) {
            val activity = activityReference.get()
            if (activity != null && !activity.isFinishing) {
                // Update UI
            }
        }
    }
}
```

### Q: What is the onTrimMemory() method?

**Answer:**
`onTrimMemory()` is a callback method that helps your app respond to memory pressure by releasing non-critical resources.

**Memory levels:**
- **TRIM_MEMORY_UI_HIDDEN** - App UI is hidden
- **TRIM_MEMORY_RUNNING_MODERATE** - Device is running low on memory
- **TRIM_MEMORY_RUNNING_LOW** - Device is running much lower on memory
- **TRIM_MEMORY_RUNNING_CRITICAL** - Device is running extremely low on memory

```kotlin
class MyApplication : Application() {
    override fun onTrimMemory(level: Int) {
        super.onTrimMemory(level)
        
        when (level) {
            TRIM_MEMORY_UI_HIDDEN -> {
                // Release UI resources
                clearImageCache()
            }
            TRIM_MEMORY_RUNNING_MODERATE,
            TRIM_MEMORY_RUNNING_LOW,
            TRIM_MEMORY_RUNNING_CRITICAL -> {
                // Release non-critical resources
                clearMemoryCache()
                System.gc() // Suggest garbage collection
            }
        }
    }
    
    private fun clearImageCache() {
        // Clear image caches like Glide, Picasso
        Glide.get(this).clearMemory()
    }
}
```

---

## Architecture Patterns

### Q: Describe MVP (Model-View-Presenter)

**Answer:**
MVP is an architectural pattern that separates concerns into three components:

- **Model** - Data layer (repositories, network, database)
- **View** - UI layer (Activities, Fragments)
- **Presenter** - Business logic layer (mediates between View and Model)

**Key characteristics:**
- View has no direct access to Model
- Presenter handles all business logic
- View is passive and only displays data
- Easier to unit test than MVC

```kotlin
// Contract interface
interface LoginContract {
    interface View {
        fun showProgress()
        fun hideProgress()
        fun showLoginSuccess(user: User)
        fun showLoginError(message: String)
    }
    
    interface Presenter {
        fun attachView(view: View)
        fun detachView()
        fun login(username: String, password: String)
    }
}

// Presenter
class LoginPresenter(private val repository: LoginRepository) : LoginContract.Presenter {
    private var view: LoginContract.View? = null
    
    override fun attachView(view: LoginContract.View) {
        this.view = view
    }
    
    override fun detachView() {
        this.view = null
    }
    
    override fun login(username: String, password: String) {
        view?.showProgress()
        
        GlobalScope.launch {
            val result = repository.login(username, password)
            withContext(Dispatchers.Main) {
                view?.hideProgress()
                if (result.isSuccess) {
                    view?.showLoginSuccess(result.getOrNull()!!)
                } else {
                    view?.showLoginError("Login failed")
                }
            }
        }
    }
}

// View (Activity)
class LoginActivity : AppCompatActivity(), LoginContract.View {
    private lateinit var presenter: LoginContract.Presenter
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_login)
        
        val repository = LoginRepository()
        presenter = LoginPresenter(repository)
        presenter.attachView(this)
    }
    
    override fun onDestroy() {
        presenter.detachView()
        super.onDestroy()
    }
    
    override fun showProgress() {
        progressBar.visibility = View.VISIBLE
    }
    
    override fun hideProgress() {
        progressBar.visibility = View.GONE
    }
    
    override fun showLoginSuccess(user: User) {
        Toast.makeText(this, "Welcome ${user.name}", Toast.LENGTH_SHORT).show()
    }
    
    override fun showLoginError(message: String) {
        Toast.makeText(this, message, Toast.LENGTH_SHORT).show()
    }
}
```

### Q: Describe MVVM (Model-View-ViewModel)

**Answer:**
MVVM is an architectural pattern that uses data binding to separate UI from business logic:

- **Model** - Data layer
- **View** - UI layer (Activities, Fragments)
- **ViewModel** - Holds UI state and business logic

**Key characteristics:**
- ViewModel survives configuration changes
- Two-way data binding possible
- ViewModel doesn't hold reference to View
- Better lifecycle management

```kotlin
// ViewModel
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    private val _user = MutableLiveData<User>()
    val user: LiveData<User> = _user
    
    private val _loading = MutableLiveData<Boolean>()
    val loading: LiveData<Boolean> = _loading
    
    private val _error = MutableLiveData<String>()
    val error: LiveData<String> = _error
    
    fun loadUser(userId: Int) {
        viewModelScope.launch {
            _loading.value = true
            try {
                val user = repository.getUser(userId)
                _user.value = user
            } catch (e: Exception) {
                _error.value = e.message
            } finally {
                _loading.value = false
            }
        }
    }
}

// View (Activity)
class UserActivity : AppCompatActivity() {
    private lateinit var binding: ActivityUserBinding
    private lateinit var viewModel: UserViewModel
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = DataBindingUtil.setContentView(this, R.layout.activity_user)
        
        val repository = UserRepository()
        viewModel = ViewModelProvider(this, UserViewModelFactory(repository))[UserViewModel::class.java]
        
        binding.viewModel = viewModel
        binding.lifecycleOwner = this
        
        observeViewModel()
    }
    
    private fun observeViewModel() {
        viewModel.user.observe(this) { user ->
            // UI automatically updates via data binding
        }
        
        viewModel.loading.observe(this) { isLoading ->
            binding.progressBar.visibility = if (isLoading) View.VISIBLE else View.GONE
        }
        
        viewModel.error.observe(this) { errorMessage ->
            if (errorMessage.isNotEmpty()) {
                Toast.makeText(this, errorMessage, Toast.LENGTH_SHORT).show()
            }
        }
    }
}
```

---

## Testing

### Q: What is Espresso?

**Answer:**
Espresso is Android's UI testing framework for writing automated tests that simulate user interactions.

**Key features:**
- Automatic synchronization with UI thread
- Fluent API for writing tests
- Built-in idling resources
- Integration with Android Test Orchestrator

**Basic Espresso test:**
```kotlin
@RunWith(AndroidJUnit4::class)
class LoginActivityTest {
    
    @get:Rule
    val activityRule = ActivityScenarioRule(LoginActivity::class.java)
    
    @Test
    fun loginWithValidCredentials_showsSuccessMessage() {
        // Type username
        onView(withId(R.id.username_edittext))
            .perform(typeText("testuser"), closeSoftKeyboard())
        
        // Type password
        onView(withId(R.id.password_edittext))
            .perform(typeText("password123"), closeSoftKeyboard())
        
        // Click login button
        onView(withId(R.id.login_button))
            .perform(click())
        
        // Verify success message is displayed
        onView(withText("Login successful"))
            .check(matches(isDisplayed()))
    }
    
    @Test
    fun loginWithInvalidCredentials_showsErrorMessage() {
        onView(withId(R.id.username_edittext))
            .perform(typeText("invalid"), closeSoftKeyboard())
        
        onView(withId(R.id.password_edittext))
            .perform(typeText("wrong"), closeSoftKeyboard())
        
        onView(withId(R.id.login_button))
            .perform(click())
        
        onView(withText("Invalid credentials"))
            .check(matches(isDisplayed()))
    }
}
```

### Q: What is Robolectric?

**Answer:**
Robolectric is a testing framework that allows you to run Android unit tests on the JVM without an emulator or device.

**Benefits:**
- Fast test execution
- No need for emulator/device
- Easy debugging
- Integration with JUnit

**Basic Robolectric test:**
```kotlin
@RunWith(RobolectricTestRunner::class)
@Config(sdk = [Build.VERSION_CODES.O_MR1])
class MainActivityTest {
    
    @Test
    fun clickingButton_shouldChangeText() {
        val activity = Robolectric.buildActivity(MainActivity::class.java)
            .create()
            .resume()
            .get()
        
        val button = activity.findViewById<Button>(R.id.button)
        val textView = activity.findViewById<TextView>(R.id.textview)
        
        button.performClick()
        
        assertThat(textView.text.toString()).isEqualTo("Button clicked!")
    }
}
```

### Q: Why Mockito is used?

**Answer:**
Mockito is a mocking framework used to create mock objects for unit testing, allowing you to isolate the code under test.

**Benefits:**
- Isolate dependencies
- Control behavior of dependencies
- Verify interactions
- Simplify testing setup

**Basic Mockito usage:**
```kotlin
class UserServiceTest {
    
    @Mock
    private lateinit var userRepository: UserRepository
    
    @Mock
    private lateinit var emailService: EmailService
    
    private lateinit var userService: UserService
    
    @Before
    fun setup() {
        MockitoAnnotations.openMocks(this)
        userService = UserService(userRepository, emailService)
    }
    
    @Test
    fun createUser_shouldSaveUserAndSendEmail() {
        // Arrange
        val user = User(1, "John Doe", "john@example.com")
        `when`(userRepository.save(user)).thenReturn(user)
        
        // Act
        val result = userService.createUser(user)
        
        // Assert
        verify(userRepository).save(user)
        verify(emailService).sendWelcomeEmail(user.email)
        assertThat(result).isEqualTo(user)
    }
}
```

---

## Tools and Technologies

### Q: What is ADB?

**Answer:**
ADB (Android Debug Bridge) is a command-line tool that allows communication with Android devices.

**Common ADB commands:**
```bash
# Device management
adb devices                    # List connected devices
adb connect <ip:port>         # Connect to device over network
adb disconnect                # Disconnect from device

# App management
adb install app.apk           # Install APK
adb uninstall com.package.name # Uninstall app
adb shell pm list packages    # List installed packages

# File operations
adb push local/file /sdcard/  # Copy file to device
adb pull /sdcard/file local/  # Copy file from device

# Debugging
adb logcat                    # View device logs
adb shell                     # Open device shell
adb shell dumpsys activity   # Dump activity information
```

### Q: What is Lint? What is it used for?

**Answer:**
Lint is a static analysis tool that checks Android project source files for potential bugs, performance issues, and code improvements.

**What Lint checks:**
- **Correctness** - Incorrect API usage, typos
- **Security** - Security vulnerabilities
- **Performance** - Inefficient layouts, unused resources
- **Usability** - Missing translations, accessibility issues
- **Accessibility** - Content descriptions, color contrast

**Running Lint:**
```bash
# Command line
./gradlew lint

# Android Studio
Analyze > Inspect Code
```

**Lint configuration:**
```xml
<!-- lint.xml -->
<lint>
    <issue id="IconMissingDensityFolder" severity="ignore" />
    <issue id="GoogleAppIndexingWarning" severity="ignore" />
    <issue id="InvalidPackage">
        <ignore regexp=".*okio.*" />
    </issue>
</lint>
```

### Q: What is ProGuard used for?

**Answer:**
ProGuard is a tool that shrinks, optimizes, and obfuscates Java bytecode.

**Main functions:**
1. **Shrinking** - Removes unused classes, fields, methods
2. **Optimization** - Optimizes bytecode
3. **Obfuscation** - Renames classes/methods with meaningless names
4. **Preverification** - Adds preverification information

**Benefits:**
- Reduces APK size
- Makes reverse engineering harder
- Improves performance slightly
- Removes dead code

**ProGuard configuration:**
```proguard
# Keep main activity
-keep public class * extends android.app.Activity

# Keep custom views
-keep public class * extends android.view.View {
    public <init>(android.content.Context);
    public <init>(android.content.Context, android.util.AttributeSet);
    public <init>(android.content.Context, android.util.AttributeSet, int);
}

# Keep Parcelable implementations
-keep class * implements android.os.Parcelable {
    public static final android.os.Parcelable$Creator *;
}
```

---

## Advanced Topics

### Q: What is the Dalvik Virtual Machine?

**Answer:**
Dalvik was Android's original runtime environment for executing Android applications.

**Key characteristics:**
- **Register-based** - Uses registers instead of stack
- **DEX format** - Executes Dalvik Executable files
- **Just-in-time compilation** - Compiles bytecode at runtime
- **Memory efficient** - Optimized for mobile devices

### Q: What are the differences between Dalvik and ART?

**Answer:**

| Aspect | Dalvik | ART |
|--------|--------|-----|
| **Compilation** | Just-in-time (JIT) | Ahead-of-time (AOT) |
| **Installation time** | Faster | Slower (compilation during install) |
| **Runtime performance** | Slower | Faster |
| **Memory usage** | Lower | Higher (compiled code) |
| **Battery life** | Lower | Better |
| **Garbage collection** | Stop-the-world | Concurrent, improved |

### Q: What is DEX?

**Answer:**
DEX (Dalvik Executable) is a file format that contains compiled Android application code.

**Characteristics:**
- **Optimized for mobile** - Designed for limited memory/processing
- **Constant pool sharing** - Multiple classes share constants
- **Compact format** - Smaller than traditional JAR files
- **Register-based** - Uses register-based instruction set

**DEX creation process:**
```
Java source (.java) → Java bytecode (.class) → DEX bytecode (.dex)
```

### Q: What is the NDK and why is it useful?

**Answer:**
NDK (Native Development Kit) allows you to implement parts of your app using native-code languages like C and C++.

**Benefits:**
- **Performance** - Critical performance-intensive tasks
- **Reuse existing libraries** - Use existing C/C++ libraries
- **Platform-specific features** - Access platform-specific APIs
- **CPU-intensive tasks** - Image processing, game engines

**Basic NDK usage:**
```cpp
// native-lib.cpp
#include <jni.h>
#include <string>

extern "C" JNIEXPORT jstring JNICALL
Java_com_example_myapp_MainActivity_stringFromJNI(
        JNIEnv* env,
        jobject /* this */) {
    std::string hello = "Hello from C++";
    return env->NewStringUTF(hello.c_str());
}
```

```kotlin
// MainActivity.kt
class MainActivity : AppCompatActivity() {
    
    companion object {
        init {
            System.loadLibrary("native-lib")
        }
    }
    
    external fun stringFromJNI(): String
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        val nativeString = stringFromJNI()
        textView.text = nativeString
    }
}
```

### Q: What is the difference between Serializable and Parcelable?

**Answer:**

| Aspect | Serializable | Parcelable |
|--------|--------------|------------|
| **Performance** | Slower (reflection) | Faster (no reflection) |
| **Memory usage** | Higher | Lower |
| **Implementation** | Simple (marker interface) | More complex |
| **Platform** | Java standard | Android specific |
| **Customization** | Limited | Full control |

**Serializable example:**
```kotlin
data class User(
    val id: Int,
    val name: String,
    val email: String
) : Serializable
```

**Parcelable example:**
```kotlin
@Parcelize
data class User(
    val id: Int,
    val name: String,
    val email: String
) : Parcelable

// Manual implementation (without @Parcelize)
data class User(
    val id: Int,
    val name: String,
    val email: String
) : Parcelable {
    
    constructor(parcel: Parcel) : this(
        parcel.readInt(),
        parcel.readString() ?: "",
        parcel.readString() ?: ""
    )
    
    override fun writeToParcel(parcel: Parcel, flags: Int) {
        parcel.writeInt(id)
        parcel.writeString(name)
        parcel.writeString(email)
    }
    
    override fun describeContents(): Int = 0
    
    companion object CREATOR : Parcelable.Creator<User> {
        override fun createFromParcel(parcel: Parcel): User = User(parcel)
        override fun newArray(size: Int): Array<User?> = arrayOfNulls(size)
    }
}
```

---

## Conclusion

This comprehensive guide covers the most important Android interview questions across all major topics. The questions range from basic concepts to advanced topics, providing both theoretical knowledge and practical code examples.

**Key areas to focus on for interviews:**
1. **Android fundamentals** - Activity/Fragment lifecycle, Context
2. **Architecture patterns** - MVP, MVVM, Clean Architecture
3. **Modern Android development** - Kotlin, Coroutines, Room, ViewModel
4. **Performance optimization** - Memory management, ANR prevention
5. **Testing** - Unit testing, UI testing with Espresso
6. **Advanced topics** - Custom views, NDK, security

**Tips for interview preparation:**
- Practice coding examples on Android Studio
- Understand the "why" behind each concept, not just the "how"
- Be prepared to discuss trade-offs and alternatives
- Stay updated with latest Android development practices
- Practice explaining concepts clearly and concisely

Good luck with your Android interviews! 🚀
