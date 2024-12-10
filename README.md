1. Покдлючение к сети интеренет
```xml
<uses-permission android:name="android.permission.INTERNET" />
```
2. Испорт всех нужных компонентов
```kotlin
    implementation("io.github.jan-tennert.supabase:gotrue-kt:1.3.2")
    implementation("io.github.jan-tennert.supabase:compose-auth:1.3.2")
    implementation("io.github.jan-tennert.supabase:compose-auth-ui:1.3.2")
    implementation("io.github.jan-tennert.supabase:storage-kt:1.3.2")
    implementation("io.github.jan-tennert.supabase:postgrest-kt:1.3.2")
    implementation("io.ktor:ktor-client-cio:2.3.4")
    implementation("androidx.compose.runtime:runtime-livedata:1.5.1")
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.6.2")
    implementation("io.coil-kt:coil-compose:2.4.0")
    implementation("androidx.navigation:navigation-compose:2.8.4")
```
```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.jetbrains.kotlin.android)
    id("org.jetbrains.kotlin.plugin.serialization")
}
```
```kotlin
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.jetbrains.kotlin.android) apply false
    kotlin("plugin.serialization") version "1.8.10"
}
```
3. Добавление, удаление, обновление бд
```kotlin
class IdeasViewModel : ViewModel() {
    private val _ideas = MutableStateFlow<List<Idea>>(emptyList()) // Используем StateFlow
    val ideas: StateFlow<List<Idea>> get() = _ideas // Данные только для чтения
    val supabaseClient = createSupabaseClient(
        supabaseUrl = "https://pdlptygcttplacpoycqn.supabase.co",
        supabaseKey = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InBkbHB0eWdjdHRwbGFjcG95Y3FuIiwicm9sZSI6ImFub24iLCJpYXQiOjE3MzM4MjY2NzMsImV4cCI6MjA0OTQwMjY3M30.9NIRrSJ5M1FfJZBeOUvoowCaNtnmbwXnpO8BZYIHGL4"
    ) {
        install(Postgrest)
    }
    fun fetchIdeas() {
        viewModelScope.launch {
            val response = supabaseClient.postgrest["ideas"]
                .select().decodeList<Idea>()
            try {
                _ideas.emit(response) 
            }
            catch (e:Exception){
                Log.d("Orimous2", e.message.toString())
            }
        }
    }
    fun addIdea(title: String, description: String?) {
        viewModelScope.launch {
            val newIdea = mapOf(
                "title" to title,
                "description" to (description ?: "")
            )
            val response = supabaseClient.postgrest["ideas"]
                .insert(newIdea)
                .decodeSingle<Idea>()
            fetchIdeas()
        }
    }
    fun updateIdea(updatedIdea: Idea) {
        viewModelScope.launch {
            val updatedData = mapOf(
                "title" to updatedIdea.title,
                "description" to (updatedIdea.description ?: "")
            )
            supabaseClient.postgrest["ideas"]
                .update(updatedData) {
                    Idea::id eq updatedIdea.id
                }
                .decodeSingle<Idea>()
            fetchIdeas()
        }
    }
    fun deleteIdea(idea: Idea) {
        viewModelScope.launch {
            supabaseClient.postgrest["ideas"]
                .delete {
                    Idea::id eq idea.id
                }
                .decodeSingle<Idea>()
            fetchIdeas()
        }
    }
}
```
4. Навигация
```kotlin
NavHost(navController = navController, startDestination = "screen1") {
    composable("screen1") {
        Screen1(navController)
    }
    composable("screen2") {
        Screen2()
    }
}
```
```kotlin
Scaffold(
    bottomBar = {
        BottomNavigationBar(navController = navController)
    },
    modifier = Modifier
        .fillMaxSize()
) { itPadding ->
    Box(
        Modifier
            .padding(itPadding)
            .fillMaxSize()
    ) {
        NavGraph(navController = navController)
    }
}
```