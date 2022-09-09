# PostgreSQL Logical Replication with Supabase
How to set up a read-only replica using PostgreSQL Logical Replication with Supabase

### Step 1: Set up a new project in Supabase to host your replica database
[https://supabase.com](https://supabase.com)

### Step 2: Migrate the database schema to the new (replica) project

1. Run `ALTER ROLE postgres SUPERUSER` in the old project's SQL editor
2. Run `pg_dump --clean --if-exists --schema-only --quote-all-identifiers -h [OLD_DB_HOST] -U postgres > schema_dump.sql` from your terminal
3. Run `ALTER ROLE postgres NOSUPERUSER` in the old project's SQL editor
4. Run `ALTER ROLE postgres SUPERUSER` in the new project's SQL editor
5. Run `psql -h [NEW_DB_HOST] -U postgres -f schema_dump.sql` from your terminal
6. Run `ALTER ROLE postgres NOSUPERUSER` in the new project's SQL editor

#### Notes for this step
- You must use the [Supabase Dashboard SQL Editor](https://app.supabase.com/project/_/sql) to change the postgres user from `NOSUPERUSER` to `SUPERUSER` and vice-versa.  The dashboard runs with the proper privileges to do this.  Connecting to the database with any other tool using the `postgres` user will not work.
- To find `[OLD_DB_HOST]` and `[NEW_DB_HOST]`, go to your [Supabase Settings Page](https://app.supabase.com/project/_/settings/database) and look under Connection Info / Host.  It will have the format of `db.zzzzzzzzzzzzzzzzzzzz.supabase.co` where `zzzzzzzzzzzzzzzzzzzz` is your project reference number.
- It's important to use the `--schema-only` option here, as you only want to dump the schema, and not the data.

### Step 3: Create a publication on the production database

```sql
CREATE PUBLICATION my_publication FOR ALL TABLES;
```

#### Notes for this step
- If you only want to replicate specific tables, you can use:
`CREATE PUBLICATION my_publication FOR TABLE table1, table2, table3;`
- The schema for each table in in your publication must exist in the replica database before you move on to create the subscription. 

### Step 4: Create a subscription on the replica database

```sql
CREATE SUBSCRIPTION my_subscription
CONNECTION 'postgresql://postgres:[PASSWORD]@[OLD_DB_HOST]:5432/postgres' 
PUBLICATION my_publication;
```

#### Notes for this section

- `PASSWORD` is your `postgres` password, i.e. the password you created when you set up your project.  (You can also reset your password from the [Supabase Dashboard](https://app.supabase.com/project/_/settings/database) under `Dashboard` / `Settings` / `Database` / `Reset Database Password`) 
- `OLD_DB_HOST` is your primary database host name, used in the steps above
- be sure to use port **5432** to connect to your PostgreSQL server, and not **6543**, which is the pg_bouncer connection pooling port.

##  Debugging your replication
See [Debugging PostgreSQL Logical Replication](./debugging.md)

### Notes regarding database migrations / schema changes

- Be careful with schema changes, they don't propagate to the replicas automatically, and will cause the replica to stop syncing.

- If you use `DROP CASCADE` on the `public` schema when attempting to resync schemas, it can cause the `realtime.subscription` to drop.

## Acknowlegements
Thanks to Colin from Zverse for pointing out some of these great debugging techniques that help solve issues related to datbase migrations.
