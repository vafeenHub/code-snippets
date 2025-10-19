# CodeSnippets

[AppDatabaseMigrationManager](#appdatabasemigrationmanager)

[Converters](#converters)

[ViewModelCreator](#viewmodelcreator)

[Java jar file with manifest](#java-jar)

[Libs don't work in release kotlin multiplatform](#lib-in-jar)

[Settings manager](#settings-manager)

## AppDatabaseMigrationManager

```kt
/**
 * Application database migration manager.
 *
 * This class manages database migrations, ensuring the update of table structures
 * when the database version changes.
 */
internal class AppDatabaseMigrationManager {
    private fun createMigration(
        startVersion: Int, endVersion: Int, migrate: (db: SupportSQLiteDatabase) -> Unit
    ): Migration = object : Migration(startVersion, endVersion) {
        override fun migrate(db: SupportSQLiteDatabase) = migrate(db)
    }

    private val migrations = listOf<Migration>()

    /**
     * Retrieves an array of migrations for use in the database.
     */
    fun migrations(): Array<Migration> = migrations.toTypedArray()
}

```

Usecase:

```kt
Room.databaseBuilder(
            context = get(), klass = AppDatabase::class.java, name = DBInfo.NAME
        ).addMigrations(*AppDatabaseMigrationManager().migrations()).build()
```

## Converters

```kt
/**
 * Interface for converting objects of one type to another.
 *
 * @param A The type of the source object.
 * @param B The type of the target object.
 */
interface OneWayBaseConverter<A, B> {
    /**
     * Converts an object of type A to an object of type B.
     *
     * @param a The source object of type A.
     * @return The converted object of type B.
     */
    fun convert(a: A): B

    /**
     * Converts a list of objects of type A to a list of objects of type B.
     *
     * @param listA A list of objects of type A.
     * @return A list of converted objects of type B.
     */
    fun convertList(listA: List<A>): List<B> = listA.map {
        convert(it)
    }
}
```

```kt
/**
 * Interface for bidirectional conversion between two types.
 *
 * @param A The type of the first object.
 * @param B The type of the second object.
 */
interface TwoWayBaseConverter<A, B> {

    /**
     * Converts an object of type A to an object of type B.
     *
     * @param a The source object of type A.
     * @return The converted object of type B.
     */
    fun convertAB(a: A): B

    /**
     * Converts an object of type B to an object of type A.
     *
     * @param b The source object of type B.
     * @return The converted object of type A.
     */
    fun convertBA(b: B): A

    /**
     * Converts a list of objects of type A to a list of objects of type B.
     *
     * @param listA The list of objects of type A.
     * @return The list of converted objects of type B.
     */
    fun convertABList(listA: List<A>): List<B> = listA.map {
        convertAB(it)
    }

    /**
     * Converts a list of objects of type B to a list of objects of type A.
     *
     * @param listB The list of objects of type B.
     * @return The list of converted objects of type A.
     */
    fun convertBAList(listB: List<B>): List<A> = listB.map {
        convertBA(it)
    }
}

```

## ViewModelCreator

```kt

/**
 * A factory class for creating ViewModel instances.
 *
 * @param VM The type of ViewModel that this factory creates.
 * @property viewModelCreator A lambda function that returns an instance of VM.
 */
class ViewModelFactory<VM : ViewModel>(
    private val viewModelCreator: () -> VM
) : ViewModelProvider.Factory {

    /**
     * Creates a ViewModel instance based on the provided model class.
     *
     * @param T The type of ViewModel to be created.
     * @param modelClass The class of the ViewModel to create.
     * @return An instance of the specified ViewModel type.
     */
    override fun <T : ViewModel> create(modelClass: Class<T>): T = viewModelCreator() as T
}

/**
 * Extension function for Fragment to create a lazy ViewModel instance.
 *
 * @param VM The type of ViewModel to be created.
 * @param creator A lambda function that returns an instance of VM.
 * @return A Lazy instance of the specified ViewModel type.
 */
inline fun <reified VM : ViewModel> Fragment.viewModelCreator(noinline creator: () -> VM): Lazy<VM> =
    viewModels { ViewModelFactory(creator) }

/**
 * Extension function for AppCompatActivity to create a lazy ViewModel instance.
 *
 * @param VM The type of ViewModel to be created.
 * @param creator A lambda function that returns an instance of VM.
 * @return A Lazy instance of the specified ViewModel type.
 */
inline fun <reified VM : ViewModel> AppCompatActivity.viewModelCreator(noinline creator: () -> VM): Lazy<VM> =
    viewModels { ViewModelFactory(creator) }
```

## java-jar

```kt
tasks.jar {
    manifest {
        attributes["Main-Class"] = "org.example.MainKt"
    }
    from(configurations.runtimeClasspath.get().map { if (it.isDirectory) it else zipTree(it) })
    duplicatesStrategy = DuplicatesStrategy.INCLUDE
}
```

## lib-in-jar

Add this line to your composeApp/build.gradle.kts if your custom libraries don't work in release distr

```kt
compose.desktop {
    application {
        mainClass = "org.example.project.MainKt"

        nativeDistributions {
            targetFormats(TargetFormat.Dmg, TargetFormat.Msi, TargetFormat.Deb)
            packageName = "SplitKitCat"
            packageVersion = "1.0.0"
        }
        buildTypes.release.proguard {
            isEnabled = true
            obfuscate = true
            optimize = true
            joinOutputJars = true
            configurationFiles.from(
                file("proguard-rules.pro")
            )
        }
    }
}
```
## Settings Manager

```kotlin

/**
 * Интерфейс для управления настройками приложения.
 */
interface SettingsManager {

    /**
     * Поток состояний, представляющий текущие настройки приложения.
     */
    val settingsFlow: StateFlow<Settings>

    /**
     * Сохраняет изменения в настройках.
     *
     * @param saving Функция, принимающая текущие настройки и возвращающая измененные настройки.
     */
    fun save(saving: (Settings) -> Settings)
}
```

```kotlin
/**
 * Реализация интерфейса [SettingsManager] для управления настройками приложения с использованием [SharedPreferences].
 * Этот класс предоставляет реактивный способ наблюдения за изменениями настроек через [StateFlow].
 *
 * @property sharedPreferences Экземпляр [SharedPreferences], используемый для хранения и загрузки настроек.
 */
internal class SettingsManagerImpl(private val sharedPreferences: SharedPreferences) :
    SettingsManager {

    /**
     * Текущие настройки, загруженные из SharedPreferences.
     */
    private var settings = sharedPreferences.getSettingsOrCreateIfNull()

    /**
     * Внутренний StateFlow для отслеживания изменений настроек.
     */
    private val _settingsFlow = MutableStateFlow(settings)

    /**
     * Публичный [StateFlow] для подписки на изменения настроек.
     * С помощью этого потока можно получать уведомления об обновлениях настроек.
     */
    override val settingsFlow: StateFlow<Settings> = _settingsFlow.asStateFlow()

    /**
     * Регистрация слушателя для отслеживания изменений в [SharedPreferences].
     * При обнаружении изменения обновленные настройки отправляются в [StateFlow].
     */
    init {
        sharedPreferences.registerOnSharedPreferenceChangeListener { sharedPreferences, key ->
            // Эмитим новые настройки, если в SharedPreferences произошло изменение
            _settingsFlow.value = sharedPreferences.getSettingsOrCreateIfNull()
        }
    }

    /**
     * Сохраняет настройки, применяя функцию преобразования к текущим настройкам.
     * Обновленные настройки сохраняются в [SharedPreferences] и автоматически передаются
     * подписчикам через [StateFlow].
     *
     * @param saving Функция, которая принимает текущие настройки и возвращает обновленные.
     */
    @Synchronized
    override fun save(saving: (Settings) -> Settings) {
        // Обновляем настройки в памяти
        settings = saving(settings)
        // Сохраняем обновленные настройки в SharedPreferences
        sharedPreferences.save(settings)
        // Обновляем flow
        _settingsFlow.value = sharedPreferences.getSettingsOrCreateIfNull()
    }

    /**
     * Сохраняет настройки в SharedPreferences.
     *
     * @param settings Объект [Settings], который нужно сохранить.
     */
    private fun SharedPreferences.save(settings: Settings) = saveInOrRemoveFromSharedPreferences {
        putString(SharedPreferencesValue.Settings.key, settings.toJsonString())
    }

    /**
     * Получает настройки из SharedPreferences или создает новые, если их нет.
     * Эта функция проверяет наличие настроек в SharedPreferences и возвращает их.
     * Если настройки не найдены, создаются новые и сохраняются в SharedPreferences.
     *
     * @return Объект [Settings], полученный из SharedPreferences или созданный по умолчанию.
     */
    private fun SharedPreferences.getSettingsOrCreateIfNull(): Settings {
        val settings = getFromSharedPreferences {
            getString(SharedPreferencesValue.Settings.key, "").let {
                if (it != "") Gson().fromJson(it, Settings::class.java)
                else null
            }
        }
        return if (settings != null) settings
        else {
            val newSettings = Settings() // Создание новых настроек по умолчанию.
            saveInOrRemoveFromSharedPreferences {
                putString(SharedPreferencesValue.Settings.key, newSettings.toJsonString())
            }
            newSettings
        }
    }

    /**
     * Сохраняет данные в SharedPreferences или удаляет их, в зависимости от переданного блока.
     *
     * Эта функция позволяет выполнить операции редактирования SharedPreferences,
     * используя переданный блок кода.
     *
     * @param save Блок кода, который будет выполнен в контексте [SharedPreferences.Editor].
     */
    private fun SharedPreferences.saveInOrRemoveFromSharedPreferences(save: SharedPreferences.Editor.() -> Unit) {
        edit().apply {
            save() // Выполнение переданного блока.
            apply() // Применение изменений.
        }
    }

    /**
     * Получает данные из SharedPreferences с использованием переданного блока.
     *
     * @param get Блок кода, который будет выполнен в контексте [SharedPreferences].
     * @return Результат выполнения блока.
     */
    private fun <T> SharedPreferences.getFromSharedPreferences(get: SharedPreferences.() -> T): T =
        get()


}
```
