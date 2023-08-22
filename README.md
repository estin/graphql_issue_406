https://github.com/supabase/pg_graphql/issues/406

Versions:
- postgres14 on registry.opensource.zalan.do/acid/spilo-14:2.1-p3 
- pg_graphql v1.3.0


Start spilo leader and secondary with pg_graphql

```bash
docker compose up -d
docker compose ps
```

Output

```
NAME                      IMAGE                     COMMAND                  SERVICE             CREATED              STATUS              PORTS
demo-spilo1               graphql-stand-by-spilo1   "/bin/sh /launch.sh …"   spilo1              22 minutes ago       Up About a minute   5432/tcp, 8008/tcp, 8080/tcp
demo-spilo2               graphql-stand-by-spilo2   "/bin/sh /launch.sh …"   spilo2              22 minutes ago       Up About a minute   5432/tcp, 8008/tcp, 8080/tcp
graphql-stand-by-etcd-1   bitnami/etcd:latest       "/opt/bitnami/script…"   etcd                About a minute ago   Up About a minute   0.0.0.0:2379-2380->2379-2380/tcp, :::2379-2380->2379-2380/tcp
```

Find the spilo leader

```bash
docker compose logs | grep "the leader" | tail -n 1
```

Output

```
demo-spilo2              | 2023-08-22 17:53:18,728 INFO: no action. I am (spilo2) the leader with the lock
```

Create extension on leader

```bash
docker compose exec -i spilo2 psql -U postgres -c "create extension pg_graphql"; 
```

Check extension on leader

```bash
docker compose exec -i spilo2 psql -U postgres -c "create table users(id serial primary key, username varchar(255) not null)"
docker compose exec -i spilo2 psql -U postgres -c "insert into public.users(username) values ('aardvark@x.com')"
docker compose exec -i spilo2 psql -U postgres -c 'select graphql.resolve($$ query{usersCollection(filter: {not: { username: { in: ["test"] }}}, first: 1) { edges{ node{username} } }} $$)'
```

Output

```
                                       resolve
--------------------------------------------------------------------------------------
 {"data": {"usersCollection": {"edges": [{"node": {"username": "aardvark@x.com"}}]}}}
(1 row)
```

Check on secondary (another spilo instance)

```bash
docker compose exec -i spilo1 psql -U postgres -c 'select graphql.resolve($$ query{usersCollection(filter: {not: { username: { in: ["test"] }}}, first: 1) { edges{ node{username} } }} $$)' 
```


**Error**

```
                                         resolve
-----------------------------------------------------------------------------------------
 {"data": null, "errors": [{"message": "cannot assign TransactionIds during recovery"}]}
(1 row)
```

