Introduction
============

**pg_restrict** is an extension to restrict some SQL commands on [PostgreSQL](http://www.postgresql.org/). It introduces the master role concept that is similar to superuser. Even superusers can be forbid to drop databases and roles (if it is not a master role).

Requirements
============

* PostgreSQL 9.3+

Build and Install
=================

This extension is supported on [those platforms](http://www.postgresql.org/docs/current/static/supported-platforms.html) that PostgreSQL is. The installation steps depend on your operating system.

You can also keep up with the latest fixes and features cloning the Git repository.

```
$ git clone https://github.com/eulerto/pg_restrict.git
```

Unix based Operating Systems
----------------------------

Before use this extension, you should build it and install it.

```
$ git clone https://github.com/eulerto/pg_restrict.git
$ cd pg_restrict
# Make sure your path includes the bin directory that contains the correct `pg_config`
$ PATH=/path/to/pg/bin:$PATH
$ make
$ make install
```

Windows
-------

There are several ways to build **pg_restrict** on Windows. If you are build PostgreSQL too, you can put **pg_restrict** directory inside contrib, change the contrib Makefile (variable SUBDIRS) and build it following the [Installation from Source Code on Windows](http://www.postgresql.org/docs/current/static/install-windows.html) instructions. However, if you already have PostgreSQL installed, it is also possible to compile **pg_restrict** out of the tree. Edit `pg_restrict.vcxproj` file and change `c:\postgres\pg113` to the PostgreSQL prefix directory. The next step is to open this project file in MS Visual Studio and compile it. Final step is to copy `pg_restrict.dll` to the `pg_config --pkglibdir` directory.

Configuration
=============

In order to function, this extension must be loaded via `shared_preload_libraries` in `postgresql.conf`.

There are several configuration parameters that control the behavior of **pg_restrict**. The default behavior is to restrict drop databases `postgres`, `template1`, and `template0` and disallow removal of role `postgres`. Role `postgres` can drop any restricted database/role (because it is a master role, by default).

* `pg_restrict.alter_system` (boolean): restrict ALTER SYSTEM command to master roles (`pg_restrict.master_roles` parameter). Default is _false_.
* `pg_restrict.alter_table`  (boolean): restrict ALTER TABLE command to master roles (`pg_restrict.master_roles` parameter). Default is _false_.
* `pg_restrict.create_table` (boolean): restrict ALTER TABLE command to master roles (`pg_restrict.master_roles` parameter). Default is _false_.
* `pg_restrict.copy_program` (boolean): restrict COPY ... PROGRAM command to master roles (`pg_restrict.master_roles` parameter). Default is _false_.
* `pg_restrict.master_roles` (string): Roles that are allowed to execute the restricted commands. If there is more than one role, separate them with comma. Default is _postgres_.
* `pg_restrict.nonremovable_databases` (string): restrict DROP databases listed here to a master role (even if the current role is the database owner or superuser). Default is _postgres, template1, template0_.
* `pg_restrict.nonremovable_roles` (string): restrict DROP roles listed here to a master role (even if the current role has CREATEROLE privilege or is a superuser). Default is _postgres_.

These parameters are set in `postgresql.conf`. Typical usage might be:

```
shared_preload_libraries = 'pg_restrict'
pg_restrict.alter_system = on
pg_restrict.alter_table  = on
pg_restrict.create_table = on
pg_restrict.copy_program = off
pg_restrict.master_roles = 'euler, admin'
pg_restrict.nonremovable_databases = 'prod, bi, mydb, postgres, template1, template0'
pg_restrict.nonremovable_roles = 'admin, euler, fulano'
```

Example
=======

The following parameters are set in `postgresql.conf`.

```
shared_preload_libraries = 'pg_restrict'
pg_restrict.master_roles = 'euler, postgres'
pg_restrict.nonremovable_databases = 'prod, bi, postgres, template1, template0'
pg_restrict.nonremovable_roles = 'admin, euler'
```

Let's create a new superuser called `fulano` and connect as `fulano`. Role `fulano` **is not** a master role. If `fulano` tries to drop a role `euler` (`euler` is listed as non-removable by non-master role), it errors out. Even though `fulano` creates a database called `prod`, it **can not** remove it because `prod` is listed as non-removable and `fulano` is not a master role.

```
postgres=# CREATE ROLE fulano SUPERUSER LOGIN;
CREATE ROLE
postgres=# \c - fulano
You are now connected to database "euler" as user "fulano".
postgres=# SELECT current_role;
 current_role 
--------------
 fulano
(1 row)

postgres=# SHOW pg_restrict.master_roles;
 pg_restrict.master_roles 
--------------------------
 euler, postgres
(1 row)

postgres=# SHOW pg_restrict.nonremovable_roles;
 pg_restrict.nonremovable_roles 
--------------------------------
 admin, euler
(1 row)

postgres=# DROP ROLE euler;
psql: ERRO:  cannot drop role "euler"
postgres=# CREATE DATABASE prod;
CREATE DATABASE
postgres=# SELECT current_role;
 current_role 
--------------
 fulano
(1 row)

postgres=# SHOW pg_restrict.nonremovable_databases;
    pg_restrict.nonremovable_databases        
------------------------------------------
 prod, bi, postgres, template1, template0
(1 row)

postgres=# DROP DATABASE prod;
psql: ERRO:  cannot drop database "prod"


postgres=# show pg_restrict.alter_table ;
 pg_restrict.alter_table
-------------------------
 on
(1 row)

postgres=# alter table foo add column foo1 integer;
ERROR:  cannot execute ALTER TABLE
postgres=#


postgres=# show pg_restrict.create_table ;
 pg_restrict.create_table
--------------------------
 on
(1 row)

postgres=# create table foo5 (foo integer);
ERROR:  cannot execute CREATE TABLE
postgres=#

```

License
=======

> Copyright (c) 2019, Euler Taveira de Oliveira
> All rights reserved.

> Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

> Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.

> Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.

> Neither the name of the Euler Taveira de Oliveira nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission.

> THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
