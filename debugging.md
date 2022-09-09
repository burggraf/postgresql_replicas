# Debugging PostgreSQL Logical Replication

### Check for errors related to replication in the logs of the primary database

- Go to the [Logs Explorer](https://app.supabase.com/project/_/logs-explorer) in the Supabase Dashboard
- Select Templates / Errors
- Look for errors related to your replication

### Errors related to not having enough replication slots
A common issue that comes up is a lack of available replication slots or available worker processes.  By default, Supabase projects are set up with a max of 4 worker processes (`max_worker_processes`) and 5 replication slots (`max_replication_slots`).

#### View current settings for worker processes or replication slots

```sql
show max_worker_processes;
show max_replication_slots;
```

#### Change settings for worker processes or replication slots

```sql
alter system set max_worker_processes to '8';
alter system set max_replication_slots to '8';
```

These new settings will not take effect until you restart your database. You can do this with the [Restart Server](https://app.supabase.com/project/_/settings/general) button in the Supabase Dashboard. 


### Refresh the subscription

On the replica:

```sql
ALTER SUBSCRIPTION my_subscription REFRESH PUBLICATION;
```

### Check the replication slot status(es) on the primary server

```sql
select * from pg_replication_slots;
```

### Drop a subscription (safely)
To drop a subscription, disable it, drop the slot, then drop the subscription.

```sql
ALTER SUBSCRIPTION my_subscription REFRESH PUBLICATION;
ALTER SUBSCRIPTION my_subscription DISABLE;
ALTER SUBSCRIPTION my_subscription SET (slot_name=NONE);
DROP SUBSCRIPTION my_subscription;
```

### Dropping a slot on the primary server
When re-recreating the subscription on the replica, you may receive an error saying that you have a duplicate slot.  In that case, drop the slot on the primary server:

```sql
select * from pg_replication_slots;
select pg_drop_replication_slot('my_subscription');
```

