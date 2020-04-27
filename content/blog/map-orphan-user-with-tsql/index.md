---
title: Mapping an orphan user in TSQL
date: "2020-04-26"
description: "Mapping an orphan user in TSQL"
---

##Mapping an orphan user 

A database user is orphan when it is not mapped to a Sql Server Login. 

This usually happens when restoring a db from backup on a diff server.

So you could have a database user named 'SomeUser' and a Sql Server login named 'SomeUser' but if the sids for those users do not match, then Sql Server does not know they are related. 

To show the orphan user in the current db run this...

```sql 

exec sp_change_users_login 'REPORT'

```

or this would give you the same results

```sql
Select	db.name, 
		db.sid
		from sys.database_principals db 
	left join sys.server_principals s
		on (db.sid = s.sid)
		where s.sid is null
		and db.authentication_type_desc = 'INSTANCE'
```


1. If you already either already have a Sql Server login with the same name as the db user name, or you would like one to be automatically created run this...

```sql

exec sp_change_users_login 'Auto_Fix', 'username'

```

2. If the Sql server login and the database user need to have different names then you can run this...

```sql
--Create the login 
create login SomeLoginName with password = 'random-string';
exec sp_change_users_login 'Update_One', 'SomeDbUser', 'SomeLoginName';

```






This sql will accomplish the same thing without using the sproc. 


