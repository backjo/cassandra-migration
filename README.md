# Cassandra Migration

## Purpose
This library can be used to implement migrations for the Cassandra database schema.
The usage is oriented on the popular tools for relation databases like flyway or liquibase.

## Usage
Using this library is quite simple. Given that you have a configured instance of the
cluster object all you need to do is integrate a the next lines in your projects startup code:

```
Database database = new Database(cluster, "nameOfMyKeyspace");
MigrationTask migration = new MigrationTask(database, new MigrationRepository());
migration.migrate();
```

This assumes that all migration scripts can be found on the classpath in a folder '/cassandra/migration'.
If you put your scripts on a different classpath location you just need to pass the path in the constructor
like this:

```java
new MigrationRepository("/my/path/here");
```

## Naming
Scripts should be named in the following schema:

```
<version>_<name>.cql
```

If the '.clq' extension is missing the file will be ignored. The 'version' is required to figure out the latest version of the scripts and relates to the version that is stored in the database schema information table. It should be an integer number. Leading zeros are accepted. The name is something that is just for the developers purpose and should be something descriptive.

## Script content
The script format is rather simple. It allows one statement per line and lines should be finished with a ';' character. Every line that is not empty will be executed against the Cassandra instance, therefore comments are currently not supported.

## Migrations
Migrations are executed with the Quorum consistency level to make sure that always a majority of nodes share the same schema information.
Error handling is not really implemented (and as far as I know not really possible from a database point of view). If one script fails the migration is stopped and an exception is thrown with an exception message that contains the name of the failing script as well as the failing script statement. If that happens, an entry will be put into the 'migration_schema' table stating that this script failed. You can then fix the script and retry the migration. It should normally not be necessary to remove failed migrations from the 'migration_schema' table.

However, in case you have multiple statements in one script and one of them failed you need to make sure that the statements before the failing one are safe to be executed again. You either need to manually revert the actions or, the preferred approach, make use of Cassandras "IF EXISTS" or "IF NOT EXISTS" mechanism to ensure that the same script can be run multiple times without failing.
