# CodeSnippets

[AppDatabaseMigrationManager](#appdatabasemigrationmanager)

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