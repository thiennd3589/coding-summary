# Dump and restore DB

## Dump db

```mongodump --archive=<backup_name>.gz --gzip --db=<database_name>```

## Restore db

```mongorestore --gzip <path/to/extracted/bson/files>```