# postgresql_replicas
How to set up a read-only replica in PostgreSQL using Supabase

### Step 1: Set up a new project in Supabase to host your replica database
[https://supabase.com](https://supabase.com)

### Step 2: Duplicate the schema on the new (replica) project
[Migrate the Database Schema](https://supabase.com/docs/guides/database#migrate-the-database)

**NOTE:** (Use the `-s` command line option of `pg_dump` to just dump the schema without data)

### Step 3: Create a publication on the production database

```sql
CREATE PUBLICATION my_publication FOR ALL TABLES;
```

### Step 4: Create a subscription on the replica database

```sql
CREATE SUBSCRIPTION my_subscription
CONNECTION 'postgresql://postgres:[PASSWORD]@[db_location]:5432/postgres' 
PUBLICATION my_publication;
```
Where:

`PASSWORD` is your `postgres` password, i.e. the password you created when you set up your project.  (You can also reset your password from the Supabase dashboard here: `Dashboard` / `Settings` / `Database` / `Reset Database Password`)

`db_location` is your production database host name, found under `Dashboard` / `Settings` / `Database` / `Connection Info` / `Host', i.e. `db.xxxxxxxxxxxxxxxxxxx.supabase.co`

### Optional: to refresh the publication after any issues that might come up:

```sql
ALTER SUBSCRIPTION my_subscription REFRESH PUBLICATION;
```
