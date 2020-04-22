# pg_show_plans

`pg_show_plans` is a module which shows the query plans of all currently running SQL statements.

You can select the output format of plans: TEXT or JSON.

This module supports PostgreSQL versions from 9.5 to 12.

### Note
When the server starts, pg_show_plans makes a hashtable  on the shared-memory in order to temporarily store query plans.
The hashtable size cannot be changed, so the plans are not stored if the hashtable is full.

## Version

*Version 1.0 RC 3*

## Installation

You can install it to do the usual way shown below.

```
$ tar xvfj postgresql-12.2.tar.bz2
$ cd postgresql-12.2/contrib
$ git clone https://github.com/cybertec-postgresql/pg_show_plans.git
$ cd pg_show_plans
$ make && make install
```

You must add the line shown below in your postgresql.conf.

```
shared_preload_libraries = 'pg_show_plans'
```

After starting your server, you must issue `CREATE EXTENSION` statement shown below.

```
testdb=# CREATE EXTENSION pg_show_plans;
```

## How to use

By issuing the following query, it shows the query plan and related information of the currently running SQL statements.

```
testdb=# SELECT * FROM pg_show_plans;
  pid  | level | userid | dbid  |                                 plan                                  
-------+-------+--------+-------+-----------------------------------------------------------------------
 11473 |     0 |     10 | 16384 | Function Scan on pg_show_plans  (cost=0.00..10.00 rows=1000 width=56)
 11504 |     0 |     10 | 16384 | Function Scan on print_item  (cost=0.25..10.25 rows=1000 width=524)
 11504 |     1 |     10 | 16384 | Result  (cost=0.00..0.01 rows=1 width=4)
(3 rows)
```

If you need the query plans of running SQL statements and also the corresponding query string, you issue the following query which is combined with pg_show_plans and pg_stat_activity.

```
testdb=# \x
Expanded display is on.
testdb=# SELECT p.pid, p.level, p.plan, a.query FROM pg_show_plans p 
   LEFT JOIN pg_stat_activity a
   ON p.pid = a.pid AND p.level = 0 ORDER BY p.pid, p.level;
-[ RECORD 1 ]-----------------------------------------------------------------------------------------
pid   | 11473
level | 0
plan  | Sort  (cost=72.08..74.58 rows=1000 width=80)                                                  +
      |   Sort Key: pg_show_plans.pid, pg_show_plans.level                                            +
      |   ->  Hash Left Join  (cost=2.25..22.25 rows=1000 width=80)                                   +
      |         Hash Cond: (pg_show_plans.pid = s.pid)                                                +
      |         Join Filter: (pg_show_plans.level = 0)                                                +
      |         ->  Function Scan on pg_show_plans  (cost=0.00..10.00 rows=1000 width=48)             +
      |         ->  Hash  (cost=1.00..1.00 rows=100 width=44)                                         +
      |               ->  Function Scan on pg_stat_get_activity s  (cost=0.00..1.00 rows=100 width=44)
query | SELECT p.pid, p.level, p.plan, a.query FROM pg_show_plans p                                   +
      |    LEFT JOIN pg_stat_activity a                                                               +
      |    ON p.pid = a.pid AND p.level = 0 ORDER BY p.pid, p.level;
-[ RECORD 2 ]-----------------------------------------------------------------------------------------
pid   | 11517
level | 0
plan  | Function Scan on print_item  (cost=0.25..10.25 rows=1000 width=524)
query | SELECT * FROM print_item(1,20);
-[ RECORD 3 ]-----------------------------------------------------------------------------------------
pid   | 11517
level | 1
plan  | Result  (cost=0.00..0.01 rows=1 width=4)
query | 

```


## pg_show_plans View
 - *pid*: the pid of the process which the query is running.    
 - *level*: the level of the query which runs the query. Top level is `0`. For example, if you execute a simple select query, the level of this query's plan is 0. If you execute a function that invokes a select query, level 0 is the plan of the function and level 1 is the plan of the select query invoked by the function.
 - *userid*: the userid of the user which runs the query.
 - *dbid*: the database id of the database which the query is running.
 - *plan*: the query plan of the running query.

## Configuration Parameters
 - *pg_show_plans.startup_enable* : Boolean that defines if plans are enabled at startup, or not. Default is `true`.
 - *pg_show_plans.plan_format* : It controls the output format of query plans. It can be selected either `text` or `json`. Default is `text`.
 - *pg_show_plans.max_plan_length* : It sets the maximum length of query plans. Default is `8192` [byte]. Note that this parameter must be set to an integer.

## Functions
 - *pg_show_plans_disable()* disables the feature. Only superuser can execute it.
 - *pg_show_plans_enable()* enables the feature. Only superuser can execute it.
 - *pgsp_format_json()* changes the output format to `json`. Note that the format of the plans that are stored in the memory before executing this function cannot be changed.
 - *pgsp_format_text()* changes the output format to `text`. Note that the format of the plans that are stored in the memory before executing this function cannot be changed.


## Change Log
 - 10 Apr, 2020: Version 1.0 RC3 Released. Supported Streaming Replication. This extension can be run on the standby server since this version.
 - 26 Mar, 2020: Version 1.0 RC2 Released. Added pgsp_format_json() and pgsp_format_text(); deleted the parameter `show_level`.
 - 21 Dec, 2019: Version 1.0 RC Released. Supported versions from 9.1 to 9.4.
 - 16 Aug, 2019: Version 0.8 Released. Supported the parameter:max_plan_length.
 - 12 Aug, 2019: Version 0.3 Released. Supported garbage collection.
 - 9 Aug, 2019: Version 0.2 Released. Supported nested queries.
 - 8 Aug, 2019: Version 0.1 Released.
