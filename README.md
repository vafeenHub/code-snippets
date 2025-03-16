# CodeSnippets

[AppDatabaseMigrationManager](#appdatabasemigrationmanager)

[Converters](#converters)

[ViewModelCreator](#viewmodelcreator)

[Java jar file with manifest](#java-jar)

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
