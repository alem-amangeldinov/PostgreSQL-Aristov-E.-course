# Replication



#### open PG port and check open ports on firewall (if PG port is not available)
```
firewall-cmd --permanent --zone=public --add-port=5432/tcp && firewall-cmd --reload && firewall-cmd --list-all
```

#### Create user with REPLICATION permission:
```
psql -c "CREATE USER replica WITH REPLICATION ENCRYPTED PASSWORD 'pass@123';"
```


#### create replication slot:
```
create replication slot:
psql -c "SELECT * FROM pg_create_physical_replication_slot('my_slot');"

list created replication slots:
psql -c "select * from pg_replication_slots;"

drop replication slot:
psql -c "select pg_drop_replication_slot('my_slot');"
```

#### add the created user in pg_hba file:
```
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host      all           replica     192.168.56.222/16         scram-sha-256
```



#### set listen_address postgresql.conf file:

```
listen_addresses = '*'
or 
only add replica host
```





#### Run pg_basebackup with R option
```
/usr/pgsql-14/bin/pg_basebackup -h 192.168.56.221 -U postgres -p 5432 -D /db/repl -P -X stream -R -c fast -S my_slot
```

```
/usr/pgsql-14/bin/pg_basebackup -h master_host -U user_name -p cluster_port -D /cluster-directory -P -X stream -R -c fast -S slot_name
```


#### start cluster after pg_basebackup is complited


```
/usr/pgsql-14/bin/pg_ctl start -D /db/repl/
```
```
error on start cluster:

/usr/pgsql-14/bin/pg_ctl start -D /db/repl/
waiting for server to start....2024-02-05 09:35:00.563 GMT [20078] FATAL:  data directory "/db/repl" has invalid permissions
2024-02-05 09:35:00.563 GMT [20078] DETAIL:  Permissions should be u=rwx (0700) or u=rwx,g=rx (0750)

solution:
 chmod 0700 /db/repl
```

#### create sh file and run pg_basebackup with nohup

```
cat > ~/basebackup.sh << EOL
#!/bin/bash
date
/usr/pgsql-14/bin/pg_basebackup -P -R -X stream -c fast -S my_slot -h master_host -p 5432 -U user_name -D /path_to_cluster_directory
date
echo 'end basebackup'
/usr/pgsql-14/bin/pg_ctl start -D /path_to_cluster_directory
echo 'start replica cluster'
date
EOL
```

```
run sh file:
nohup sh ~/basebackup.sh &


check:
tail -f nohup.out
```

