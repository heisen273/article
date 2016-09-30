# How to grant read-only access to a database in Postgres.

So, lets say you have a customer who needs read-only access in your DB, – so he could run SELECT queries but won't have a chance to INSERT/UPDATE/ALTER anything. You'll need to do just a several steps, in order to  achieve this. 

First of all, – we need to run commands using DB owner user. Do figure out who owns DB, – simply login into DB using

```python
psql -d database_name
```

and then do:

```python
database_name=# \l
                              List of databases
   Name    | Owner    | Encoding |   Collate   |    Ctype    | Access privileges
-----------+----------+----------+-------------+-------------+-------------------
 db_name   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
(1 row)

database_name=#
```

So, we can see that our DB owner is user 'postgres'. Thus, we need to run commands using postgres privileges in linux. 
Next step would be creating readonly user and giving him right privileges. 
Run following command in order to figure out where is postgres located on your machine
```python
which psql
```
In my case it is /usr/local/bin/psql.
Then you'll need to do:
```python
sudo -u postgres /usr/local/bin/createuser -U postgres -D -R -S readonly_user
```
This command will create user with limited permissions. 
Key -D means that this user cannot create db's; 
-R means  that this user cannot create any roles;
-S means that this user won't be superuser.

Okay, so we got ourself a DB user, now lets give him Select permissions and login passwrd .
```python
sudo -u postgres /usr/local/bin/psql -U postgres database_name -c "ALTER USER readonly_user WITH ENCRYPTED PASSWORD 'supper_strong_pass'"

sudo -u postgres /usr/local/bin/psql -U postgres database_name -c "GRANT CONNECT ON DATABASE database_name TO readonly_user"

sudo -u postgres /usr/local/bin/psql -U postgres database_name -c "GRANT USAGE ON SCHEMA public TO readonly_user"

sudo -u postgres /usr/local/bin/psql -U postgres database_name -c "GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_user"
```
This block of commands will give readonly user password, then connectionn, usage & select access on our DB.
So, once you done that, – you'll need to edit your pg_hba.conf file, so we force our user to use password when he connects to  db.
After line "# "local" is for Unix domain socket connections only" you'll need to add following :
```python
local database_name readonly_user md5
```
And restart DB afterwards. 

Then, if you need to connect to DB using readonly user, – simply do
```pythono
psql -d database_name -U readonly_user
```



