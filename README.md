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

Please note, that if you want to open remote access to your database for specific IP, you'll need to add following line to your pg_hba.conf(where 10.10.10.10 is user's IP):
```python
host database_name readonly_user 10.10.10.10/32 md5
```
If you want to open remote access to ALL users from that specific IP adress, you'll need to simply add such kind of line to your pg_hba.conf file:
```python
host database_name all 10.10.10.10/32 md5
```

Then, if you need to connect to DB using readonly user, – simply do
```python
psql -d database_name -U readonly_user
```

Also, after this you need to change listen_address(by default its 'localhost') value inside postgresql.conf file:
```python
listen_addresses='*'
```

And don't forget to restart the DB after you changed pg_hba.conf, otherwise changes won't be applied. 





This specific guide gives client access to DataBase via DB  user. The idea is that you don't want to let the guy login onto your serv via ssh(using Unix user) and then login into DB using DataBase-user. 
I believe this is more secure, cause who knows what is in his mind: you might end up with your server hang down even if he doesn't have superuser privelegies, – there are many ways to end up with such result.
