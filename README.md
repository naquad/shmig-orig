SHMIG
=====

### WARNING

This is the archived original version of shmig that is stored for the reference
purposes. Updated version resides at https://github.com/mbucc/shmig.


A database migration tool written in BASH consisting of just one file - `shmig`.

Why?
----

Currently there are lots of database migration tools such as [DBV](http://dbv.vizuina.com/), [Liquibase](http://www.liquibase.org/) and other framework-specific ones (for Ruby on Rails, Yii, Laravel, ...). But they all are pretty heavy, with lots of dependencies (or even unusable outside of their stack), some own DSLs...

I needed some simple, reliable solution with minimum dependencies and able to run in pretty much any POSIX-compatible environment against different databases (PostgreSQL, MySQL, SQLite3).

And here's the result.

Idea
----

RDMS'es are bundled along with their console clients. MySQL has `mysql`, PostgreSQL has `psql` and SQLite3 has `sqlite3`. And thats it! This is enough for interacting with database in batch mode w/o any drivers or connectors.

Using client options one can make its output suitable for batch processing with standard UNIX text-processing tools (`sed`, `grep`, `awk`, ...). This is enough for implementing simple migration system that'll store current schema version information withing database (see `SCHEMA_TABLE` variable in `shmig.conf.example`).

Usage
-----

SHMIG tries to read configuration from config file (see `shmig.conf.example`), you can also configure it from command line (command line settings have higher priority than configuration file settings so you can override stuff).

Required options are:

  1. `TYPE` or `-t`/`--type` - database type
  2. `DATABASE` or `-d`/`--database` - database to operate on
  3. `MIGRATIONS` or `-m`/`--migrations` - directory with migrations

All other options (see `shmig.conf.example` and `shmig --help`) are not necessary.
To simplify usage you should create `shmig.conf` file in your project root directory and put there configuration then just run `shmig <action> ...` in that directory.

For detailed information see `shmig.conf.example` and `shmig --help`.

Migrations
----------

Migrations are SQL files whos name starts with "`<UNIX TIMESTAMP>-`" and end with ".sql".
File contains two special markers: `-- ====  UP ====` that marks start of section that will be executed when migration is applied and `-- ==== DOWN ====` that marks start of section that will be executed when migration is revererted.

For example:

```
-- Migration: create users table
-- Created at: 2013-10-02 07:03:11
-- ====  UP  ====
CREATE TABLE `users`(
  id int not null primary key auto_increment,
  name varchar(32) not null,
  email varchar(255) not null
);

CREATE UNIQUE INDEX `users_email_uq` ON `users`(`email`);
-- ==== DOWN ====
DROP TABLE `users`;
```

Everything since `-- ==== UP ====` till `-- ==== DOWN ====` will be executed when migration is applied and everything since `-- ==== DOWN ====` till the end of file will be executed when migration is reverted. If migration is missing marker or contents of marker is empty then appropriate action will fail (i.e. if you're trying to revert migration that has no or empty `-- ==== DOWN ====` marker you'll get an error and script won't execute any migrations following script with error). Also note those semicolons terminating statements. They're required because you're basically typing that into your database CLI client.

SHMIG can generate skeleton migration for you, see `create` action.

Current state
-------------

This is very early release. I've tried it with SQLite3, PostgreSQL, MySQL databases and didn't find any bugs. If you find any then please report them along with your migrations (or similar that will allow to reproduce bug), tools versions, detailed description of steps and config file (w/o DB credentials).

Security considerations
-----------------------

Password is passed to `mysql` and `psql` via environment variable. This can be a security issue if your system allows other users to read environment of process that belogs to another user. In most Linux distributions with modern kernels this is forbidden. You can check this (on systems supporting /proc filesystem) like this: `cat /proc/1/env` - if you get permission denied error then you're secure.

Efficiency
----------

Because SHMIG is just a shell script it's not a speed champion. Every time a statement is executed new client process is spawned. I didn't experience much issues with speed, but if you'll have then please file an issue and maybe I'll get to that in detail.

Todo
----

  1. Speed. Some optimizations are definitely possible to speed things up.
  2. A way to spawn just one CLI client. Maybe something with FIFOs and SIGCHLD handler.
  3. Better documentation :\
