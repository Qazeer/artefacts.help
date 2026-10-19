---
title: Microsoft SQL Server
summary: 'Microsoft SQL Server is a relational database engine. Each installed instance maintains its own set of databases, log files, and optional auditing outputs under the instance data directory. \n\n The following SQL Server–specific sources are of forensics interest: \n\n - Database files (master, msdb, model, and user databases): principals, roles/permission definitions, linked servers, and job configured.\n\n - Transaction logs: for data changes since last database checkpoint.\n\n - SQL Server error logs (ERRORLOG): engine lifecycle events, enabling or disabling of features (such as sp_configure), and login failures. \n\n - SQL Trace (.trc files) and SQL Server Profiler: event tracing framework, that provide granular control over events/fields collection and event filtering events. Intented to for operationnal monitoring/troubleshooting and replaced by Extended Events. A "default trace" is enabled by default on every instance and captures a small amount of built-in history about significant instance activity.\n\n - Extended Events/XEvents sessions (.xel files): event tracing framework that replaces SQL Trace. Extended events provide similar granular control over the events and event fields that will be logged. Not enabled by default.\n\n - SQL Server Audit (.sqlaudit files and/or EVTX events 33205 / 33206): security logging framework, built on top of Extended Events. Not enabled by default.\n\n - SQL Server Agent log files and the msdb database, for scheduled job execution and backup metadataz'
keywords: Microsoft SQL Server, MSSQL, SQL Server, ERRORLOG, xp_readerrorlog, SQL Server Audit, sqlaudit, .sqlaudit, Extended Events, xel, .xel, ldf, .ldf, trc, .trc, default trace, SQLAGENT, msdb, sysjobhistory, backupset, 18456, 18454, 33205, 33206, 15457, xp_cmdshell, OLE automation, sp_configure, OLEDB, linked server, MSSQLSERVER, SQLSERVERAGENT, system_health, Always On, Database Mail, PolyBase, fn_get_audit_file, fn_xe_file_target_read_file
tags:
  - databases
  - windows_server_roles
location: 'Default instance location:\n %ProgramFiles%\Microsoft SQL Server\<MAJOR_VERSION>\<INSTANCE_NAME>\MSSQL\.\n System and user database files: <INSTANCE_FOLDER>\MSSQL\DATA\.\n Log files: <INSTANCE_FOLDER>\MSSQL\Log\.\n\n System databases: master.mdf, msdb.mdf, and model.mdf.\n\n Transaction log: .ldf files.\n\n Error log: ERRORLOG, ERRORLOG.1, ...\n\n Windows event log:\n Channel: Application. Events: 18456, 15457, 18454, etc. (sources MSSQLSERVER/MSSQL$<INSTANCE>/SQLSERVERAGENT).\n\n SQL Trace (target dependant): .trc files.\n Default trace: log_<N>.trc.\n\n Extended Events (target dependant): binary .xel files.\n\n SQL Server Audit (target dependant): binary .sqlaudit files and/or Windows event Application / Security 33205/33206.\n\n SQL Server Agent log: SQLAGENT.OUT (and numbered archives).'
last_updated: 2026-10-19
sidebar: sidebar
permalink: microsoft_sqlserver.html
folder: windows
---

### Overview

`Microsoft SQL Server` is a relational database engine. Each installed
instance maintains its own set of databases, log files, and optional auditing
outputs under the instance data directory.

The following `SQL Server`–specific sources are of forensics interest to
investigate compromise or misuse of an instance (authentication abuse,
privilege escalation through database features, data access or exfiltration,
persistence through jobs or logins, etc.):

  - Database files (`master`, `msdb`, `model`, and user databases): principals,
    roles/permission definitions, linked servers, and job configured.
  
  - Transaction logs: for data changes since last database checkpoint.

  - SQL Server `error logs` (`ERRORLOG`): for engine lifecycle events,
    enabling or disabling of features (such as `sp_configure`), and login
    failures.

  - `SQL Trace` (`.trc` files) and `SQL Server Profiler`: event tracing
    framework, that provide granular control over events/fields collection and
    event filtering events. Intented to for operationnal
    monitoring/troubleshooting and replaced by `Extended Events`.
    
    The `SQL Server Profiler` is an interface to create and manage traces, and
    to analyze and replay trace results.
  
    A "default trace" is enabled by default on every instance and captures a
    small amount of built-in history about significant instance activity
    (including security audit events, such as changes to logins, roles, and
    permissions).

  - `Extended Events`/`XEvents` sessions (`.xel` files): event tracing
    framework that replaces `SQL Trace`. Extended events provide similar
    granular control over the events and event fields that will be logged. Not
    enabled by default.

  - `SQL Server Audit` (`.sqlaudit` files and/or `EVTX` events
    `33205` / `33206`): security logging framework, built on top of
    `Extended Events`. Not enabled by default.
  
  - `SQL Server Agent` log files and the `msdb` database, for scheduled job
    execution and backup metadata.

#### Default instance path
 
`SQL Server` stores instance files by default under
`<PROGRAMFILES>\Microsoft SQL Server\<MAJOR_VERSION>\<INSTANCE_NAME>\`.

The folder name encodes the major version, for example `MSSQL16.MSSQLSERVER`
for a default instance on `SQL Server 2022`.

| Component | Typical path |
|-----------|--------------|
| System and user database data files. | `<INSTANCE_FOLDER>\MSSQL\DATA\` |
| Logs files. | `<INSTANCE_FOLDER>\MSSQL\Log\` |
| Setup / upgrade bootstrap (install-time) logs. | `%ProgramFiles%\Microsoft SQL Server\<MAJOR_VERSION>\Setup Bootstrap\LOG\` |

### Database data files

At a minimum, every SQL Server database requires at least two files: a data
file and a transaction log file (`.ldf`). Data files store the database's data
and objects, including tables, indexes, stored procedures, and views. Data files
can be organized into a primary file (`.mdf`) and optional secondary files
(`.ndf`). Secondary files can be across multiple disks by putting each file on
a different disk drive.

The following tables/views can be of forensic interest:

| Database | table | Forensics use |
|----------|-------|---------------|
| `master` | `sys.server_principals` | SQL Server principals, such as `SQL_LOGIN`, `SERVER_ROLE`, `WINDOWS_USER`, or `WINDOWS_GROUP`. <br> Stores each principal's name, SID, type, create timestamp, and last modified timestamp. <br> `Server roles` have a server-wide scope while `database roles` are database-wide in their permissions scope. There are additionally two types of roles: `fixed roles`, that have a fixed and defined set of permissions, and `user-defined roles`, that can be manually created and assigned permissions. <br> Any login can see their own login name, the system logins, and the fixed server roles. Viewing other logins requires `ALTER ANY LOGIN`, or a permission on the login. Viewing user-defined server roles requires `ALTER ANY SERVER ROLE`, or membership in the role. |
| `master` | `sys.sql_logins` | SQL Server authentication logins (`SQL_LOGIN`, i.e users). <br> Stores the hash of SQL login password. <br> Any login can see their own login name, and the `sa` login. To see other logins, the principal requires `ALTER ANY LOGIN`, `VIEW SERVER SECURITY DEFINITION`, or a permission on the login. |
| `master` | `sys.server_role_members` | SQL Server role memberships, linking the `Principal ID` of a role to a `Principal ID` of each member. |
| `master` | `sys.server_permissions ` | SQL Server permissions, that are the rights to access and modify the service configuration and databases objects, such as `CONNECT SQL`, `IMPERSONATE`, `IMPERSONATE ANY LOGIN`, `ALTER ANY LOGIN`, etc. <br> The permissions is granted, through  `Principal ID`, by `grantor_principal_id` to `princigrantee_principal_id`. <br> The permissions can apply at the server-level (`SERVER`), endpoint (`ENDPOINT`), login (`LOGIN`). |
| `master` | `sys.configurations` | Effective server-wide configuration values, such as `xp_cmdshell`, `clr enabled`, etc. |
| `master` | `sys.credentials` and `sys.database_scoped_credentials` | Respectively server-level and database scoped credentials. <br> Access to  `sys.credentials` requires either `VIEW ANY DEFINITION` permission or `ALTER ANY CREDENTIAL` permission. Access to  `sys.database_scoped_credentials` requires `CONTROL` permission on the database. |
| `master` | `sys.asymmetric_keys`, `sys.symmetric_keys`, and `sys.certificates` | Cryptographic keys and certificates at the instance level. |

#### Linked servers

The linked server mechanism allows for access to others `Object Linking and
Embedding, Database (OLE DB)` data sources outside of the present SQL Server
instance. The mechanism can be used at the database level to connect to and
query a variety of data stores including, but not limited to:
 - SQL Servers.
 - Oracle Servers.
 - Text Files.
 - Excel Files.

A server link can be configured to use the current security context of the
login, a specified Windows or SQL Server login of the linked server, or be
disabled if no credentials are provided. By default, any login that belongs to
the `PUBLIC` role can query a database through a server link and may thus use
the configured credentials (if any).

Moreover, stored procedures, such as `xp_cmdshell`, can be executed over
a server link, according to the configured login roles and permissions. Note
that outgoing RPC connections, `RPC Out`, need to be enabled on the link to
conduct reconfiguration operations to enable `xp_cmdshell` on the linked
instance.

| Database | table | Forensics use |
|----------|-------|---------------|
| `master` | `sys.servers` | Linked and remote server(s) registered (including the local server with a `server_id` of `0`). |
| `master` | `sys.linked_logins` | Linked-server-login mapping, for use by RPC and distributed queries from local server to the corresponding linked server. <br> Link a remote user name (`remote_name`) to a remote server, by `server ID` from `sys.servers`. <br> The remote user name and password are stored, with only the username accessible in catalog view interfaces. |

#### SQL Server Agent

The `SQL Server Agent` is a Windows service that executes scheduled
tasks, denominated `SQL Server Agent jobs`. `SQL Server Agent` is available is
all versions of `SQL server`, except `SQL Server Express`, **but is disabled by
default**.

In order to fulfil its function, the `SQL Server Agent` Windows service must be
run using an account having the `sysadmin` fixed server role in `SQL Server` as
well as the following Windows privileges: `SeServiceLogonRight`,
`SeAssignPrimaryTokenPrivilege`, `SeChangeNotifyPrivilege`, and
`SeIncreaseQuotaPrivilege`.

The `SQL Server Agent jobs` can be executed:
  - through a `SQL Agent schedule`, for example at a recurring interval or at a
    specific timestamp. A job can be associated with multiple schedules, and
    reciprocally, a schedule can dictate the execution of multiple jobs.
  - upon the triggering of a `SQL Agent alert`, for example in response to an
    event such as another job execution or the reaching of a system resources
    usage threshold.
  - directly by executing the `sp_start_job` stored procedure.

A `SQL Server Agent job` is composed of (at least) one or multiple steps, each
step being assigned to a specific `SQL Server Agent` subsystem. It is possible
to execute operating system commands using the following subsystems:
  - `CmdExec`: run an executable with the specified command line option, such
    as `cmd.exe /c <COMMAND>` for example.
  - `PowerShell`: run a PowerShell script, by specifying either the PowerShell
    code directly or a PowerShell script file.   
  - `ActiveX`: run an `ActiveX` script. Note that the `ActiveSscripting`
    subsystem is discontinued since `SQL Server 2016` (included).

Note that a `SQL Server Agent job` can run locally on the `SQL Server` they are
configured as well as on one or multiple remote servers.

The permissions to configure, execute, and delete `SQL Server Agent jobs` are
governed by the following fixed database roles:

| Role | Scope | Notable associated permissions |
|------|-------|--------------------------------|
| `sysadmin` | Fixed-server role. | Can administrate and execute any jobs, regardless of the job's owner. <br><br> Is the only role that can define new `proxy accounts`. <br> Additionally, can define and execute jobs that will run as the `SQL Server Agent` Windows service account. <br><br> By default, only the members of the `sysadmin` fixed server role can setup a multi-servers environment. |
| `SQLAgentUserRole` | `msdb` database fixed-database role. | Can create and execute local jobs under their own security context or using the identity of an existing `proxy account`. <br><br> Can enumerate, modify, or delete jobs they own. <br><br> By default, cannot delete the job history of the jobs they own. <br> Cannot enumerate, administrate, or execute jobs they don't own. |
| `SQLAgentReaderRole` | `msdb` database fixed-database role. | Includes the permissions of the `SQLAgentUserRole` role. <br><br> Can additionally enumerate and view the properties / history of all local or multi-servers jobs. |
| `SQLAgentOperatorRole` | `msdb` database fixed-database role. | Includes the permissions of the `SQLAgentUserRole` and `SQLAgentReaderRole` roles. <br><br> Can additionally execute, stop, and enable / disable all local jobs and their job history. <br><br> Cannot however modify or delete jobs they don't own (nor make use of multi-servers jobs). |

The following tables store  SQL Server Agent job configuration information and
past execution details:

| Database | table | Forensics use |
|----------|-------|---------------|
| `msdb`   | `dbo.sysjobs` | Scheduled job to be executed by `SQL Server Agent`. <br> Stores each job's unique ID (`job_id`), name, description, whether the job is enabled, first step id (`start_step_id`), and creation and last modification timestamps. |
| `msdb`   | `dbo.sysjobsteps` | Step(s) for scheduled jobs. <br> Link each job to its step(s), i.e command(s) to be executed in sequential order. A step can either be the final step or define different subsequent step(s), with a separate step defined for success and failure outcomes. <br> Stores each step's associated `job_id`, step ID (`step_id`), name, subsystem, command, eventual additional parameters, on success/fail action and eventual associated next steps step_id, and last run information (outcome, duration, run timestamp, etc.). |
| `msdb`   | `dbo.sysschedules` |  SQL Server Agent job schedules, defining a `schedule_id` and scheduling parameters, i.e when a job will be executed (once, at instance start, daily, weekly, monthly, etc.). |
| `msdb`   | `dbo.sysjobschedules` | Links SQL Server Agent jobs to schedules, mapping a `job_id` to a `schedule_id` and referencing when the next scheduled execution will occur (`next_run_date` + `next_run_time`). |
| `msdb`   | `dbo.sysjobhistory` | Historical executions of SQL Server Agent jobs. <br> Stores, for each past execution: the `job_id`, `step_id`, step name, the outcome status (succeeded, failed, canceled, etc.), the run date, time, and duration. |

### Transaction logs

Transaction log files (`.ldf`) record the information needed to recover all
database transactions. They thus contain substantial technical information.

Each physical transaction log file is divided into several `Virtual Log Files`
(`VLFs`). Virtual log files have no fixed size, and there's no fixed number of
virtual log files for a physical log file. In turn, each `VLF` contains one or
more `log blocks`, that act as a container of `log records`. A log block is
variable in size and is the basic unit of transaction logging when writing log
records to disk.

The transaction log operates logically as a string of log records. Each log
record is identified by an incremental `Log Sequence Number` (`LSN`). Each new
log record is written to the logical end of the active `VLF`, and with an
increased `LSN`.

#### Transaction log's Log Records

`Log records` have a defined structure with a metadata fields and varying
additional fields depending on the log record type/operation.

Each log record contains the following notable fields:
  - The log record `LSN`. Example value: `00000025:0000001a:0047`.

  - The previous log record `LSN`.

  - The log record `type`/`operation`. There are between 70 and 100 log record
    operations recorded in the transaction log, depending on the version of SQL
    Server version in use. These operations include:
      
      - The start and end of each transaction: `LOP_BEGIN_XACT` and
        `LOP_COMMIT_XACT`/`LOP_ABORT_XACT`. Each transaction starts with a
        `LOP_BEGIN_XACT` log record and concludes with a
        `LOP_COMMIT_XACT`/`LOP_ABORT_XACT` log record.

      - Every data modification, such as insert (`LOP_INSERT_ROWS`), update
        (`LOP_MODIFY_ROW`) or delete (`LOP_DELETE_ROWS`).
    
      - Creating or dropping a table or index.
    
      - Every extent and page allocation or deallocation.
  
  - A begin timestamp (`Begin Time`) for `LOP_BEGIN_XACT` log record,
    and a finish timestamp (`End Time`) for `LOP_COMMIT_XACT` log record.

  - The `Transaction ID` the log record is part of. The
    `LOP_COMMIT_XACT`/`LOP_ABORT_XACT` log records additionnaly reference
    directly the `LSN` of their associated `LOP_BEGIN_XACT` log record. It is
    thus possible to bound in time each transaction and every operations they
    contain.

  - For operations that modify data, such as `LOP_INSERT_ROWS` or
    `LOP_MODIFY_ROW`, the `Page ID` and `Slot ID`. These two fields can be used
    to locate the modified data in the database. However, as data can
    reshuffled and reorganized in a clustered table, the data doesn't
    necessarily always remain in the same `Page ID` and `Slot ID` over time.

  - For operations that modify data, the data pre and post transaction
    modification in the `RowLogX` fields, such as `RowLog0` and `RowLog1`.
    For `LOP_INSERT_ROWS` operations, `RowLog0` contains the inserted data.
    For `LOP_MODIFY_ROW` operations, `RowLog0` and `RowLog1` contain the old
    and new data respectively.
    The `RowLog` fields contain hex-encoded undocumented binary data that
    depend of the operation type. It is possible to
    [reconstruct the data change](https://www.sqlservercentral.com/articles/reading-sql-servers-transaction-log)
    by parsing the `RowLogX` fields.

#### Transaction log truncation

Transaction log truncation is the process of feeing inactive `VLFs` for reuse.
This prevents the `transaction log` from growing indefinitely and consuming all
available disk space.

The [`Recovery model`](https://learn.microsoft.com/en-us/sql/relational-databases/backup-restore/recovery-models-sql-server)
configured for the SQL Server instance governs when the transaction log
truncation process occurs. Regardless of the recovery model, a database
checkpoint must first be taken before the transaction log can be truncated.
Checkpoints are [periodically conducted by the Database Engine](https://learn.microsoft.com/en-us/sql/relational-databases/logs/database-checkpoints-sql-server).

Transaction log truncation occurs automatically after the following events:
  - Under the `simple` recovery model, after a checkpoint.
  - Under the `full` or `bulk-logged` recovery models, after a log backup and
    if a checkpoint has occurred since the previous backup. In `bulk-logged`
    recovery model the log space usage is reduced by using minimal logging for
    most bulk operations.

Log records from freed `VLFs` are not immediately overwritten, and historical
data may be carved from the space.

#### Transaction logs parsing

On a live instance, transaction logs can be retrieved using
`SELECT * FROM fn_dblog(NULL, NULL)`.

The
[`SQLServerForensics`](https://github.com/aarsakian/SQLServerForensics) utility
can be leveraged to parse and extraction the transactions from transaction log
files: 

```bash
# Process all tables from the database, exporting content and insert timestamp from Transaction logs if available.
MSSQLParser.exe -db <DB_MDF> -ldb <TRANSACT_LOG_LDF>.ldf -processtables -showcontent -log -export <OUTPUT_FOLDER> -format csv

# Process the specified table from the database, exporting content and insert timestamp from Transaction logs if available.
# Exporting resulting is optionnal.
MSSQLParser.exe -db <DB_MDF> -ldb <TRANSACT_LOG_LDF>.ldf -processtables -tables <TABLE> -showcontent -log [-export <OUTPUT_FOLDER> -format csv]

# Show transaction log raw data.
MSSQLParser.exe -db <DB_MDF> -ldb <TRANSACT_LOG_LDF>.ldf -showldf -filterlop <any|insert|begin|commit|begin_ckpt|end_ckpt>
```

### SQL Server error log

The SQL Server error log are plain-text, sequentially numbered log
files (`ERRORLOG`, then `ERRORLOG.1`, `ERRORLOG.2`, …) written by the Database
Engine. A new file is created when the log is recycled (on Database restart or
when the current file reaches its size limit).

The error log records the following notable information of forensics interest:

  - Instance startup and shutdown (`SQL Server is starting`, version and
    build strings, and listening endpoints).

  - Failed authentication attempts by default. Login auditing can be configured
    to record failed logins only (default), successful logins only, both
    successful and failed logins, or be completely disabled.

  - Changes to instance configuration through `sp_configure` (for example
    enabling `xp_cmdshell`, to allow execution of operating system commands
    through the instance, or `clr enabled` to allow loading of user controlled
    `Common Language Runtime` (`CLR`) assembly).

  - Database attach / detach, recovery, and fatal errors.

The error logs can be read directly from disk under `<INSTANCE_FOLDER>\MSSQL\Log\`.

Alternatively, on a live instance, the SQL Server error log can be directly
queried using the
[`xp_readerrorlog` extended procedure](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-readerrorlog-transact-sql):

```SQL
// Reads the current SQL Server error log.
EXECUTE sp_readerrorlog;

// Reads the previous or located at index N SQL Server error log.
EXECUTE sp_readerrorlog <1 | N>;

// Searches for the keyword "fail" in the current SQL Server error log.
EXECUTE sp_readerrorlog 1, 1, 'fail';
```

{% include note.html content="Login failure. Includes the username and remote client IP address." %}

```
2026-06-04 22:01:38.68 Logon       Login failed for user 'sa'. Reason: Password did not match that for the login provided. [CLIENT: 10.1.2.3]
```

{% include note.html content="Non default login success. Includes the username, user type (SQL server or Windows account), and remote client IP address." %}

```
2026-06-04 23:06:55.52 Logon       Login succeeded for user 'sqlserver\sqlserveradmin'. Connection made using Integrated authentication. [CLIENT: 10.1.2.3]

2026-06-04 23:09:27.57 Logon       Login succeeded for user 'sa'. Connection made using SQL Server authentication. [CLIENT: 10.1.2.3]
```

{% include note.html content="sp_configure configuration change, such as enabling 'xp_cmdshell' or 'clr enabled'"%}

```
2026-06-04 22:06:22.45 spid51      Configuration option 'clr enabled' changed from 0 to 1. Run the RECONFIGURE statement to install.

2026-06-04 22:06:57.27 spid51      Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
2026-06-04 22:06:57.27 spid51      Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
2026-06-04 22:06:57.28 spid51      Configuration option 'show advanced options' changed from 1 to 0. Run the RECONFIGURE statement to install.
```

{% include note.html content="Indicates that a SQL Server Profiler trace has started. Does NOT indicate a successful login." %}

```
2026-06-03 21:45:28.50 spid9s      SQL Trace ID 1 was started by login "sa".
```

### Windows ETW events

`SQL Server` publishes events to the `Windows` `Application` event log through
providers such as `MSSQLSERVER` (default instance),
`MSSQL$<INSTANCE_NAME>` (named instance), and `SQLSERVERAGENT`.

| Channel | Conditions | Events / information of interest |
|---------|------------|----------------------------------|
| Channel: <br> `Application` <br><br> EVTX file: <br> `Application.evtx` | Requires auditing of failed logins to be enabled (default configuration). | Event `18456`: `Login failed for user '<USERNAME>'. Reason: Password did not match that for the login provided. [CLIENT: <REMOTE_IP>]`. <br><br> Also logged in SQL Server error log. <br><br> Information of interest: <br> - username. <br> - Client/remote IP address. <br> - Reason of the login failure (invalid username, invalid password, etc.). |
| Channel: <br> `Application` <br><br> EVTX file: <br> `Application.evtx` | Requires auditing of successful logins to be enabled (non-default configuration). | Event `18454`: `Login succeeded for user '<USERNAME>'. Connection made using SQL Server authentication. [CLIENT: <REMOTE_IP>]`. <br><br> Also logged in SQL Server error log. <br><br> Information of interest: <br> - username. <br> - User type (SQL server or Windows account). <br> - Client/remote IP address. |
| Channel: <br> `Application` <br><br> EVTX file: <br> `Application.evtx` | Default configuration. | Event `15457`: `Configuration option '<OPTION_NAME>' changed from <OLD_VALUE> to <NEW_VALUE>. Run the RECONFIGURE statement to install`. <br><br> Also logged in SQL Server error log. <br><br> sp_configure configuration change, such as enabling `xp_cmdshell` or `clr enabled`. <br><br> Information of interest: <br> - Option/setting name. <br> - Old and new values of the configuration setting. <br> - For Windows logins, domain and username of the user that conducted the change. SQL server login do not appear to be logged. |
| Channel: <br> `Application` / `Security` <br><br> EVTX file: <br> `Application.evtx` / `Security.evtx` | Requires `SQL Server Auditing` to be configured, with the Windows event log as a target. The  events logged will depend on the actions/operations audited (as specified in the `Audit Specification`). | Event `33205`: `Audit event: <AUDIT_DATA>`. <br><br> For more information, refer to the [SQL Server Auditing section below](#sql-server-auditing). |

The full list of Windows ETW events emitted by the `Microsoft SQL Server` engine
can be found in the
[official Microsoft documentation](https://learn.microsoft.com/en-us/sql/relational-databases/errors-events/database-engine-events-and-errors-31000-to-41399).

### SQL Server Auditing

SQL Server Audit is a native logging SQL Server feature that records activity
occurring at both the SQL Server instance and database levels. The auditing can
log user authentications, privilege changes, setting changes, and data access
events. It is built on top of [`Extended Events`](#sql-server-extended-events).

SQL Server Audit is not enabled/configured by default. The actions/operations
logged are also fully configurable, so the data available will be dependant of
the SQL Server Audit settings configured.

#### SQL Server Audit components

SQL Server Audit is composed of multiple components:
  - The SQL Server `Audit objects`, configured at the SQL Server instance level.
    Multiple audits can be configured per SQL Server instance.
  
  - The audit target/destination, that defines where events will be stored for
    a given audit. Audit data can be written to the Windows `Security` or
    `Application` event log, or binary `.sqlaudit` audit files. An audit object
    can and must have a single target. When audit records are written to audit
    files, the audit target configuration also specifies file retention
    parameters, including the maximum number of audit files and the maximum
    size permitted for each file.  

  - An `Audit Specifications` that defines the actions logged.
  
    Audit Specification can be configured at the:
    
    - Server level, known as `server audit specification`, to log
      actions/operations such as user authentications or server-level
      configuration changes. The operations logged are defined through
      `server-level audit action groups`.

    - Database level, known as `database audit specification`, to log
      actions/operations on the database or database's objects (tables, views,
      stored procedures, etc.).
      
      Access to the database data can be logged through audit of `SELECT`,
      `UPDATE`, `INSERT`, `DELETE`, and `EXECUTE` statements at the database or
      database object. It is thus possible to audit access to the data of a
      specific or all tables in a database.
      
    - Audit level, to log changes to the audit specifications at the server
      or database level.

    An audit specification can be linked to a single Audit object and an Audit
    object can have only one audit specification at a time.

    The details of the `audit action groups` and individual
    `database audit actions` that can be logged are referenced in the
    [Microsoft documentation](https://learn.microsoft.com/en-us/sql/relational-databases/security/auditing/sql-server-audit-action-groups-and-actions). 

The `Audit objects` and `Audit Specifications` of a live SQL Server instance
can be accessed using `SQL Server Management Studio` respectively under:
 - Security -> Audits
 - Security -> "Server Audit Specification" and Databases -> <DB> -> Security
   -> "Database Audit Specification".

#### SQL Server Audit event fields

The following notable fields are consistently available in audit records:
  - `file_name`: The path and name of the audit log file that the record came
    from.

  - `event_time`: Date and time when the action was performed.

  - `action_id`: four character code that indicate the action/operation type.

    Notable values include:

    | Code | Description | Available in  SQL Server version |
    |------|-------------|----------------------------------|
    | `AUTH` | Authenticate. | Starting with SQL Server 2012. |
    | `CO` | Connect.  | Starting with SQL Server 2012. |
    | `LGIS` | Login succeeded.  | Starting with SQL Server 2012. |
    | `LGIF` | Login failed.  | Starting with SQL Server 2012. |
    | `LGSD` | `SUCCESSFUL_LOGIN_GROUP` group operations.  | Starting with SQL Server 2012. |
    | `LGFL` | `FAILED_LOGIN_GROUP` group operations.  | Starting with SQL Server 2012. |
    | `LGO` | Logout. | Starting with SQL Server 2012. |
    | `LO` |  `LOGOUT_GROUP` group operations.  | Starting with SQL Server 2012. |
    | `TXBG` | Transaction begin.  | Starting with SQL Server 2016. |
    | `TRBS` | Transaction commit.  | Starting with SQL Server 2016. |
    | `TX` | `TRANSACTION_GROUP` group operations.  | Starting with SQL Server 2016. |
    | `SL` | `SELECT` statement.  | Starting with SQL Server 2012. |
    | `FT` | `FULLTEXT` statement, logged when a named pipe connection is established between the SQL Full-text Filter Daemon Launcher/Filter Daemon and the SQL Server instance. | Starting with SQL Server 2012. |
    | `FTG` | `FULLTEXT_GROUP` group operations. | Starting with SQL Server 2012. |
    | `CR` | `CREATE` statement.  | Starting with SQL Server 2012. |
    | `DL` | `DELETE` statement.  | Starting with SQL Server 2012. |
    | `IN` | `INSERT` statement.  | Starting with SQL Server 2012. |
    | `UP` | `UPDATE` statement.  | Starting with SQL Server 2012. |
    | `AL` | `ALTER` statement. | Starting with SQL Server 2012. |
    | `VW` | `VIEW` statement.  | Starting with SQL Server 2012. |
    | `VDST` | `VIEW DATABASE STATE` statement.  | Starting with SQL Server 2012. |
    | `VSST` | `VIEW SERVER STATE` statement.  | Starting with SQL Server 2012. |
    | `DR` | `DROP` statement.  | Starting with SQL Server 2012. |
    | `DPRL` | `DROP MEMBER` statement.  | Starting with SQL Server 2012. |
    | `IMP` | `IMPERSONATE` statement.  | Starting with SQL Server 2012. |
    | `LGEA` | `ENABLE` statement.  | Starting with SQL Server 2012. |
    | `LGDA` | `DISABLE` statement.  | Starting with SQL Server 2012. |
    | `EX` | `EXECUTE` statement.  | Starting with SQL Server 2012. |
    | `XA` | External access assembly.  | Starting with SQL Server 2012. |
    | `XU` | Unsafe assembly.  | Starting with SQL Server 2012. |
    | `PWC` | Change password. | Starting with SQL Server 2012. |
    | `PWCS` | Change own password.  | Starting with SQL Server 2012. |
    | `USLG` | Change user login.  | Starting with SQL Server 2012. |
    | `BST` | Batch started. | Starting with SQL Server 2016. |
    | `BCM` | Batch completed. | Starting with SQL Server 2016. |
    | `APRL` | Add member. | Starting with SQL Server 2012. |
    | `ALST` | Alter settings. | Starting with SQL Server 2012. |
    | `TO` | Take ownership. | Starting with SQL Server 2012. |
    | `ADBO` | Bulk admin. | Starting with SQL Server 2012. |
    | `OPSV` |  `SERVER_OPERATION_GROUP` group operations.  | Starting with SQL Server 2012. |
    | `MNSO` |  `SERVER_OBJECT_CHANGE_GROUP` group operations.  | Starting with SQL Server 2012. |
    | `GRSV` |  `SERVER_PERMISSION_CHANGE_GROUP` group operations.  | Starting with SQL Server 2012. |
    | `IMSP` |  `SERVER_PRINCIPAL_IMPERSONATION_GROUP` group operations.  | Starting with SQL Server 2012. |
    | `ADSP` |  `SERVER_ROLE_MEMBER_CHANGE_GROUP` group operations.  | Starting with SQL Server 2012. |
    | `OPDB` |  `DATABASE_OPERATION_GROUP` group operations.  | Starting with SQL Server 2012. |
    | `MNDO` |  `DATABASE_OBJECT_CHANGE_GROUP` group operations.  | Starting with SQL Server 2012. |
    | `GRDB` |  `DATABASE_PERMISSION_CHANGE_GROUP` group operations.  | Starting with SQL Server 2012. |
    | `IMDP` |  `DATABASE_PRINCIPAL_IMPERSONATION_GROUP` group operations.  | Starting with SQL Server 2012. |
    | `ADDP` |  `DATABASE_ROLE_MEMBER_CHANGE_GROUP` group operations.  | Starting with SQL Server 2012. |
    
  - `succeeded`: Indicates whether the action that triggered the event
    succeeded (`1` => success and `0` => fail).

  - `session_id`: Identified of the session under which the event occurred.

  - `sequence_number`: Tracks the sequence of records within a single audit
    operation that was too large to fit in a single row.

  - `server_principal_name`, `server_principal_sid`, and `server_principal_id`:
    respectively `login`, `login ID`, and `ID` of the login server context the
    action is performed in.

  - `database_name`: The database context in which the action occurred, or NULL
    for operations that occured at the server level.

  - `database_principal_name`: `login` of the login database context the
    action is performed in (for operations that occured at the database level).

  - `object_name`: The name of the entity on which the audit occurred, NULL if
    the entity is the SQL server itself or if the operaton isn't performed at
    an object level.

The following notable additionnal fields can be available depending on the
event type:

  - `statement`: Transact SQL statement if avaible. Returns NULL if not
    applicable.

  - `client_ip`: source IP of the client application.

  - `application_name`: Name of the client application that executed the
    statement that caused the audit event.

The full reference of the SQL Server Auditing event fields can be found in the
[official Microsoft documentation](https://learn.microsoft.com/en-us/sql/relational-databases/security/auditing/sql-server-audit-records?view=sql-server-ver17).

#### SQL Server Audit logs access/parsing

On a live SQL Server instance, audit records within binary `.sqlaudit` audit
files can be read using the system function `sys.fn_get_audit_file`. Altough
the function requires a live instance, it can be leveraged to read audit files
exported from another instance.

```sql
SELECT
    event_time,
    server_principal_name,
    client_ip,
    action_id,
	  CASE action_id
		  WHEN 'AS' THEN 'ACCESS'
      WHEN 'APRL' THEN 'ADD MEMBER'
      WHEN 'ADSC' THEN 'ADD SENSITIVITY CLASSIFICATION'
      WHEN 'AL' THEN 'ALTER'
      WHEN 'ALCN' THEN 'ALTER CONNECTION'
      WHEN 'ALLR' THEN 'ALTER LEDGER'
      WHEN 'ALRC' THEN 'ALTER LEDGER CONFIGURATION'
      WHEN 'ALRS' THEN 'ALTER RESOURCES'
      WHEN 'ALSS' THEN 'ALTER SERVER STATE'
      WHEN 'ALST' THEN 'ALTER SETTINGS'
      WHEN 'ALTR' THEN 'ALTER TRACE'
      WHEN 'PWAR' THEN 'APPLICATION_ROLE_CHANGE_PASSWORD_GROUP'
      WHEN 'AUSC' THEN 'AUDIT SESSION CHANGED'
      WHEN 'AUSF' THEN 'AUDIT SHUTDOWN ON FAILURE'
      WHEN 'CNAU' THEN 'AUDIT_CHANGE_GROUP'
      WHEN 'AUTH' THEN 'AUTHENTICATE'
      WHEN 'BA' THEN 'BACKUP'
      WHEN 'BAL' THEN 'BACKUP LOG'
      WHEN 'BRDB' THEN 'BACKUP_RESTORE_GROUP'
      WHEN 'BCM' THEN 'BATCH COMPLETED'
      WHEN 'BST' THEN 'BATCH STARTED'
      WHEN 'BCMG' THEN 'BATCH_COMPLETED_GROUP'
      WHEN 'BSTG' THEN 'BATCH_STARTED_GROUP'
      WHEN 'LGB' THEN 'BROKER LOGIN'
      WHEN 'LGBG' THEN 'BROKER_LOGIN_GROUP'
      WHEN 'ADBO' THEN 'BULK ADMIN'
      WHEN 'LGDB' THEN 'CHANGE DEFAULT DATABASE'
      WHEN 'LGLG' THEN 'CHANGE DEFAULT LANGUAGE'
      WHEN 'CCLG' THEN 'CHANGE LOGIN CREDENTIAL'
      WHEN 'PWCS' THEN 'CHANGE OWN PASSWORD'
      WHEN 'PWC' THEN 'CHANGE PASSWORD'
      WHEN 'USLG' THEN 'CHANGE USERS LOGIN'
      WHEN 'USAF' THEN 'CHANGE USERS LOGIN AUTO'
      WHEN 'CP' THEN 'CHECKPOINT'
      WHEN 'CO' THEN 'CONNECT'
      WHEN 'USTC' THEN 'COPY PASSWORD'
      WHEN 'CR' THEN 'CREATE'
      WHEN 'CMLG' THEN 'CREDENTIAL MAP TO LOGIN'
      WHEN 'DBAF' THEN 'DATABASE AUTHENTICATION FAILED'
      WHEN 'DBAS' THEN 'DATABASE AUTHENTICATION SUCCEEDED'
      WHEN 'DABO' THEN 'DATABASE BULK ADMIN'
      WHEN 'DBL' THEN 'DATABASE LOGOUT'
      WHEN 'LGM' THEN 'DATABASE MIRRORING LOGIN'
      WHEN 'MNDB' THEN 'DATABASE_CHANGE_GROUP'
      WHEN 'DAGL' THEN 'DATABASE_LOGOUT_GROUP'
      WHEN 'LGMG' THEN 'DATABASE_MIRRORING_LOGIN_GROUP'
      WHEN 'ACDO' THEN 'DATABASE_OBJECT_ACCESS_GROUP'
      WHEN 'MNDO' THEN 'DATABASE_OBJECT_CHANGE_GROUP'
      WHEN 'TODO' THEN 'DATABASE_OBJECT_OWNERSHIP_CHANGE_GROUP'
      WHEN 'GRDO' THEN 'DATABASE_OBJECT_PERMISSION_CHANGE_GROUP'
      WHEN 'OPDB' THEN 'DATABASE_OPERATION_GROUP'
      WHEN 'TODB' THEN 'DATABASE_OWNERSHIP_CHANGE_GROUP'
      WHEN 'GRDB' THEN 'DATABASE_PERMISSION_CHANGE_GROUP'
      WHEN 'MNDP' THEN 'DATABASE_PRINCIPAL_CHANGE_GROUP'
      WHEN 'IMDP' THEN 'DATABASE_PRINCIPAL_IMPERSONATION_GROUP'
      WHEN 'ADDP' THEN 'DATABASE_ROLE_MEMBER_CHANGE_GROUP'
      WHEN 'DBCC' THEN 'DBCC'
      WHEN 'DBCG' THEN 'DBCC_GROUP'
      WHEN 'DL' THEN 'DELETE'
      WHEN 'D' THEN 'DENY'
      WHEN 'DWC' THEN 'DENY WITH CASCADE'
      WHEN 'LGDA' THEN 'DISABLE'
      WHEN 'DR' THEN 'DROP'
      WHEN 'DPRL' THEN 'DROP MEMBER'
      WHEN 'DRSC' THEN 'DROP SENSITIVITY CLASSIFICATION'
      WHEN 'LGEA' THEN 'ENABLE'
      WHEN 'ENLR' THEN 'ENABLE LEDGER'
      WHEN 'EX' THEN 'EXECUTE'
      WHEN 'XA' THEN 'EXTERNAL ACCESS ASSEMBLY'
      WHEN 'EADS' THEN 'EXTERNAL GOVERNANCE ATTRIBUTES DELTA SYNCHRONIZATION'
      WHEN 'EAFS' THEN 'EXTERNAL GOVERNANCE ATTRIBUTES FULL SYNCHRONIZATION'
      WHEN 'EGDP' THEN 'EXTERNAL GOVERNANCE POLICY DELTA PULL'
      WHEN 'EGFP' THEN 'EXTERNAL GOVERNANCE POLICY FULL PULL'
      WHEN 'EGTF' THEN 'EXTERNAL GOVERNANCE TURNED OFF'
      WHEN 'EGTO' THEN 'EXTERNAL GOVERNANCE TURNED ON'
      WHEN 'EGOG' THEN 'EXTGOV_OPERATION_GROUP'
      WHEN 'DAGF' THEN 'FAILED_DATABASE_AUTHENTICATION_GROUP'
      WHEN 'LGFL' THEN 'FAILED_LOGIN_GROUP'
      WHEN 'FWDL' THEN 'FIREWALL DELETE'
      WHEN 'FWUD' THEN 'FIREWALL UPDATE'
      WHEN 'FT' THEN 'FULLTEXT'
      WHEN 'FTG' THEN 'FULLTEXT_GROUP'
      WHEN 'GDLR' THEN 'GENERATE LEDGER DIGEST'
      WHEN 'LGG' THEN 'GLOBAL TRANSACTIONS LOGIN'
      WHEN 'LGGG' THEN 'GLOBAL_TRANSACTIONS_LOGIN_GROUP'
      WHEN 'G' THEN 'GRANT'
      WHEN 'GWG' THEN 'GRANT WITH GRANT'
      WHEN 'IMP' THEN 'IMPERSONATE'
      WHEN 'IN' THEN 'INSERT'
      WHEN 'OPLR' THEN 'LEDGER_OPERATION_GROUP'
      WHEN 'LGIF' THEN 'LOGIN FAILED'
      WHEN 'LGIS' THEN 'LOGIN SUCCEEDED'
      WHEN 'PWCG' THEN 'LOGIN_CHANGE_PASSWORD_GROUP'
      WHEN 'LGO' THEN 'LOGOUT'
      WHEN 'LO' THEN 'LOGOUT_GROUP'
      WHEN 'PWMC' THEN 'MUST CHANGE PASSWORD'
      WHEN 'LGNM' THEN 'NAME CHANGE'
      WHEN 'NMLG' THEN 'NO CREDENTIAL MAP TO LOGIN'
      WHEN 'OP' THEN 'OPEN'
      WHEN 'PWEX' THEN 'PASSWORD EXPIRATION'
      WHEN 'PWPL' THEN 'PASSWORD POLICY'
      WHEN 'RC' THEN 'RECEIVE'
      WHEN 'RF' THEN 'REFERENCES'
      WHEN 'PWRS' THEN 'RESET OWN PASSWORD'
      WHEN 'PWR' THEN 'RESET PASSWORD'
      WHEN 'RS' THEN 'RESTORE'
      WHEN 'R' THEN 'REVOKE'
      WHEN 'RWC' THEN 'REVOKE WITH CASCADE'
      WHEN 'RWG' THEN 'REVOKE WITH GRANT'
      WHEN 'RCM' THEN 'RPC COMPLETED'
      WHEN 'RST' THEN 'RPC STARTED'
      WHEN 'ACO' THEN 'SCHEMA_OBJECT_ACCESS_GROUP'
      WHEN 'MNO' THEN 'SCHEMA_OBJECT_CHANGE_GROUP'
      WHEN 'TOO' THEN 'SCHEMA_OBJECT_OWNERSHIP_CHANGE_GROUP'
      WHEN 'GRO' THEN 'SCHEMA_OBJECT_PERMISSION_CHANGE_GROUP'
      WHEN 'SL' THEN 'SELECT'
      WHEN 'SN' THEN 'SEND'
      WHEN 'SBC' THEN 'SENSITIVE BATCH COMPLETED'
      WHEN 'SRC' THEN 'SENSITIVE RPC COMPLETED'
      WHEN 'SBCG' THEN 'SENSITIVE_BATCH_COMPLETED_GROUP'
      WHEN 'SCCG' THEN 'SENSITIVITY_CLASSIFICATION_CHANGE_GROUP'
      WHEN 'SVCN' THEN 'SERVER CONTINUE'
      WHEN 'SVPD' THEN 'SERVER PAUSED'
      WHEN 'SVSD' THEN 'SERVER SHUTDOWN'
      WHEN 'SVSR' THEN 'SERVER STARTED'
      WHEN 'MNSO' THEN 'SERVER_OBJECT_CHANGE_GROUP'
      WHEN 'TOSO' THEN 'SERVER_OBJECT_OWNERSHIP_CHANGE_GROUP'
      WHEN 'GRSO' THEN 'SERVER_OBJECT_PERMISSION_CHANGE_GROUP'
      WHEN 'OPSV' THEN 'SERVER_OPERATION_GROUP'
      WHEN 'GRSV' THEN 'SERVER_PERMISSION_CHANGE_GROUP'
      WHEN 'MNSP' THEN 'SERVER_PRINCIPAL_CHANGE_GROUP'
      WHEN 'IMSP' THEN 'SERVER_PRINCIPAL_IMPERSONATION_GROUP'
      WHEN 'ADSP' THEN 'SERVER_ROLE_MEMBER_CHANGE_GROUP'
      WHEN 'STSV' THEN 'SERVER_STATE_CHANGE_GROUP'
      WHEN 'SPLN' THEN 'SHOW PLAN'
      WHEN 'UNDO' THEN 'STATEMENT ROLLBACK'
      WHEN 'UNDG' THEN 'STATEMENT_ROLLBACK_GROUP'
      WHEN 'LGS' THEN 'STORAGE LOGIN'
      WHEN 'LGSG' THEN 'STORAGE_LOGIN_GROUP'
      WHEN 'SUQN' THEN 'SUBSCRIBE QUERY NOTIFICATION'
      WHEN 'DAGS' THEN 'SUCCESSFUL_DATABASE_AUTHENTICATION_GROUP'
      WHEN 'LGSD' THEN 'SUCCESSFUL_LOGIN_GROUP'
      WHEN 'TO' THEN 'TAKE OWNERSHIP'
      WHEN 'C2OF' THEN 'TRACE AUDIT C2OFF'
      WHEN 'C2ON' THEN 'TRACE AUDIT C2ON'
      WHEN 'TASA' THEN 'TRACE AUDIT START'
      WHEN 'TASP' THEN 'TRACE AUDIT STOP'
      WHEN 'TRCG' THEN 'TRACE_CHANGE_GROUP'
      WHEN 'TXBG' THEN 'TRANSACTION BEGIN'
      WHEN 'TRBC' THEN 'TRANSACTION BEGIN COMPLETED'
      WHEN 'TRBS' THEN 'TRANSACTION BEGIN STARTING'
      WHEN 'TXCM' THEN 'TRANSACTION COMMIT'
      WHEN 'TRCC' THEN 'TRANSACTION COMMIT COMPLETED'
      WHEN 'TRCS' THEN 'TRANSACTION COMMIT STARTING'
      WHEN 'TRPC' THEN 'TRANSACTION PROMOTE COMPLETED'
      WHEN 'TRPS' THEN 'TRANSACTION PROMOTE STARTING'
      WHEN 'TRGC' THEN 'TRANSACTION PROPAGATE COMPLETED'
      WHEN 'TRGS' THEN 'TRANSACTION PROPAGATE STARTING'
      WHEN 'TXRB' THEN 'TRANSACTION ROLLBACK'
      WHEN 'TRRC' THEN 'TRANSACTION ROLLBACK COMPLETED'
      WHEN 'TRRS' THEN 'TRANSACTION ROLLBACK STARTING'
      WHEN 'TRSC' THEN 'TRANSACTION SAVEPOINT COMPLETED'
      WHEN 'TRSS' THEN 'TRANSACTION SAVEPOINT STARTING'
      WHEN 'TXGG' THEN 'TRANSACTION_BEGIN_GROUP'
      WHEN 'TXCG' THEN 'TRANSACTION_COMMIT_GROUP'
      WHEN 'TX' THEN 'TRANSACTION_GROUP'
      WHEN 'TXRG' THEN 'TRANSACTION_ROLLBACK_GROUP'
      WHEN 'TRO' THEN 'TRANSFER'
      WHEN 'PWU' THEN 'UNLOCK ACCOUNT'
      WHEN 'XU' THEN 'UNSAFE ASSEMBLY'
      WHEN 'UP' THEN 'UPDATE'
      WHEN 'UDAU' THEN 'USER DEFINED AUDIT'
      WHEN 'UCGP' THEN 'USER_CHANGE_PASSWORD_GROUP'
      WHEN 'UDAG' THEN 'USER_DEFINED_AUDIT_GROUP'
      WHEN 'VFLR' THEN 'VERIFY LEDGER'
      WHEN 'VW' THEN 'VIEW'
      WHEN 'VWCT' THEN 'VIEW CHANGETRACKING'
      WHEN 'VDST' THEN 'VIEW DATABASE STATE'
      WHEN 'VSST' THEN 'VIEW SERVER STATE'
      ELSE CONCAT('UNKNOWN (', action_id, ')')
	  END AS action_id_details,
    succeeded,
    database_name,
    object_name,
    statement,
    file_name
FROM sys.fn_get_audit_file('<SQLAUDIT_FILE_PATH>', DEFAULT, DEFAULT)
-- WHERE event_time >= '2024-03-15' 
ORDER BY event_time;
```

Alternatively, the
[`Convert-SqlAudit.ps1`](https://web.archive.org/web/20260415113628/https://gist.github.com/PatrickKeisler/d34f29f943151d253dc6626a1fd5c08b#file-convert-sqlaudit-ps1)
PowerShell script, leveraging the `Read-SqlXEvent` cmdlet (from the `SqlServer`
module) can be used to parse to parse binary `.sqlaudit` audit files to CSV:

```
.\Convert-SqlAudit.ps1 -FileName "<SQLAUDIT_FILE_PATH>" -OutputPath <OUTPUT_DIR>
```

### SQL Trace and SQL Server Profiler

`SQL Trace` is an event tracing framework integrated into the
`SQL Server Database Engine`. SQL Trace provides mechanisms to define the
events monitored, configure event filters, and collect associated event fields.
The events, fields, and filtering criteria can be customized to tailor data
collection to specific monitoring needs. `SQL Trace` are mostly leveraged to
gather performance insights into normal operations and in troubleshooting
scenarios.

The `SQL Server Profiler` is integrated into the SQL Server tooling and is an
interface to create and manage traces, and to analyze and replay trace results.

Following the release of `Extended Events` in SQL Server 2008, SQL Trace and
SQL Server Profiler were deprecated, with the deprecation announced in SQL
Server 2012. Both features remain available for backward compatibility, but
Extended Events is the recommended framework for new monitoring and
troubleshooting implementations.

A common use case for SQL Server Profiler is to monitor expensive or failing
queries, but it can also be used more broadly to capture query activity, stored
procedure executions, or login events. Captured events can be inspected in real
time, or saved to a file or database table for subsequent analysis.

#### SQL Trace components

The SQL Trace framework consists of several components including:

  - `Traces`, that define the event classes monitored, event fields recorded,
    filters applied, and the Destination used to store the data. A given SQL
    Server instance can have multiple traces, and each trace can be created,
    started, stopped, and configured independently.

  - `Event classes`, that define the types of server operations that can be
    monitored, such as `SQL:BatchCompleted`, `RPC:Completed`,
    `SQL:StmtCompleted`, and `Audit Login`.
    Each event class exposes a specific set of data columns, some of which are
    common across multiple event classes.
  
  - `Event categories`, that group related event classes, such as TSQL,
    `Stored Procedures`, and Security Audit.
  
  - `Events`, that are individual occurrences of an event class (for example,
    one specific batch completing). Each event is recorded as a row in the
    trace output.

  - `Filters`, that define the conditions used to restrict event collection.
    For example by database, login, application name, session ID, or query
    duration.
  
  - `Destinations`, that define how and where to store the data. Server-side
    traces created with `sp_trace_create` write to a trace file (`.trc`). SQL
    Server Profiler can additionally display events live through its rowset
    provider, and can save captured events to a trace file or to a SQL Server
    table.

#### SQL Trace event fields

Each `SQL Trace` event contains a set of data columns describing the associated
operation. The available columns depend on the `event class` selected and the
columns enabled for collection.

The following fields are commonly collected:

  - `StartTime` and `EndTime`.
  - `EventClass` and `EventSubClass`.
  - `DatabaseID` and `DatabaseName`.
  - `SPID` (SQL Server session ID).
  - `LoginName`, `NTUserName`, and `SessionLoginName`.
  - `HostName` and `ApplicationName`.
  - `TextData`, containing the SQL statement or other event-specific text.
  - `Duration`, representing the duration of the operation.
  - CPU Reads and Writes.
  - `RowCounts`, representing the number of rows affected or returned,
    depending on the event.
  - `Error`, containing an associated error number when applicable.

The fields available and their interpretation depend on the event class. For
example, `SQL:BatchCompleted` records information about a completed SQL batch,
whereas `RPC:Completed` records information about a completed remote procedure
call. Some fields may be unavailable or not populated for specific event
classes.

#### Default Trace

SQL Server includes a lightweight server-side trace, known as the default
trace, that is enabled by default on every instance. It was introduced in SQL
Server 2005 and runs with trace ID 1. This trace provides a small amount of
built-in history about significant instance activity without any prior setup,
which makes it useful for after-the-fact troubleshooting.

The default trace captures a fixed set of event classes spanning several
categories, including:

  - Object changes, such as `Object:Created`, `Object:Altered`, and
    `Object:Deleted`.
  - Database file growth and shrink operations.
  - Server and database configuration changes, such as server/database
    management operations.
  - Security audit events, such as changes to logins, roles, and permissions.
  - Selected error and warning events, such as sort warnings, hash warnings,
    and missing column statistics.

The default trace writes to rollover files named `log_<N>.trc` (with an
incrementing number), located in the same directory as the SQL Server error
log. Each file is limited to 20 MB, and a maximum of five files are retained,
so on busy instances the history can cover only a short period. A new file is
also started each time the instance restarts. Since older files are
overwritten, data that needs to be preserved should be copied or loaded into a
table.

Unlike user-defined traces, the definition of the default trace (events,
columns, filters, and destination) cannot be customized. It can only be
switched on or off with the advanced server configuration option
`default trace enabled`.

#### SQL Trace configuration and access

On a live SQL Server instance, the following query retrieves the metadata for
the existing traces on the current instance:

```sql
SELECT *
FROM sys.traces;
```

The `sys.traces` catalog view exposes information about active and configured
traces, including their trace IDs, paths, status, and associated configuration
metadata.

Existing SQL traces can be inspected using the functions `fn_trace_getinfo`,
`fn_trace_geteventinfo`, `fn_trace_getfilterinfo`, and
`fn_trace_gettable`.

Server-side trace files can be read using the system function
`fn_trace_gettable`. Although the function requires a live instance, it can be
leveraged to read trace files generated by another instance, provided the files
are accessible to the SQL Server process executing the query.

```sql
-- The DEFAULT argument instructs the function to read the subsequent rollover files associated with the specified trace file, when applicable.

SELECT *
FROM fn_trace_gettable('<TRC_FILE>', DEFAULT);
```

Alternatively, SQL Server Profiler can open and inspect saved `.trc` trace
files through its graphical interface, and can save captured events directly to
a table.

### SQL Server Extended Events

The `SQL Server Extended Events` is a performance monitoring system integrated
into the SQL Server Database Engine to enable collection of detailed server
activity. This system is mostly leveraged to gather performance insights into
normal operations and in troubleshooting scenarios.

Extended Events, also known as XEvents, provide granular control over data
collection and event filtering, allowing data collection tailored to specific
monitoring needs. The operations/actions logged and associated event fields
recorded can be fully customized. The events and event fields available will
thus be fully dependent on the Extended Events sessions configured.

Following the release of Extended Events, in SQL Server 2008, Microsoft
deprecated `SQL profiler`, another data collection and tracing framework aimed
at performance measurement and troubleshooting.

A common use case for Extended Events is to monitor expensive or failing
queries, but they can also be used more broadly to capture all query activity
or to log authentication events.

#### Extended Events components

The Extended Events engine consists of several components including:

  - `Sessions`, that defines the operations/actions monitored, event fields
    recorded, and the `Target(s)` implemented to store the data. A given SQL
    Server instance can have multiple sessions, and each session can be created,
    modified, started, or stopped independently.

  - `Targets`, that defines how and where to store the data. Targets can
    notably be `ring_buffer` for in-memory storage or `event_file` for
    persistent on-disk storage. A session must define at least one target
    and can write to multiple targets.

    If the target is of `event_file` type, XEvents are stored as binary `XEL`
    files.

The Extended Events of a live SQL Server instance can be accessed using
`SQL Server Management Studio` under: Management -> Extended Event
-> Sessions.

#### Extended Events event fields

Each extended event contains a common set of fields, including:
`timestamp_utc`, `object_name` (event name), and `event_data`. The `event_data`
field contains the overall data/fields for the event, in XML format. 

There is a common set of `Global Fields` that can be recorded/captured for all
Extended Events types (but are not enabled by default). Each event type also
has a set of specific fields (with some optionnal that can be individually
toggled).

The following global fields may be of interest, if enabled:
  - `server_instance_name`.
  - `database_id`.
  - `database_name`.
  - `session_id`.
  - `username`, `server_principal_name`, `server_principal_sid`.
  - `client_hostname`, `client_app_name`.
  - `sql_text`.
  - `num_response_rows`.
  - `task_time`.
  - `last_error`.

#### Extended Events access

On a live SQL Server instance, Extended Events `.xel` files can be read using
the system function `sys.fn_xe_file_target_read_file`. Altough the function
requires a live instance, it can be leveraged to read audit files exported
from another instance.

```sql
SELECT
    object_name AS EventName,
    timestamp_utc,
    CAST(event_data AS XML) AS EventData,
    file_name,
    file_offset
FROM sys.fn_xe_file_target_read_file('<XEL_FILE>', NULL, NULL, NULL) AS event_data; 
```

Alternatively, the `Read-SqlXEvent` cmdlet (from the `SqlServer` module) can be
used to parse to parse binary XEvents `XEL` audit files (or directly retrieve
XEvents from a live SQL Server instance):

```bash
Read-SqlXEvent -FileName <XEL_FILE>

# Retrieves the XEvents from the specified XEvent session.
Read-SQLXEvent -ConnectionString "Server=.;Database=master;Trusted_Connection=True" -SessionName <XEVENT_SESSION_NAME>
```

### SQL Server Agent log

The `SQL Server Agent` service maintains its own text log: `SQLAGENT.OUT`
(and rotated `SQLAGENT.1`, …), under the instance `MSSQL\Log` folder.

The Agent logs record agents startup, jobs, operator actions, and errors
encountered while running scheduled tasks.

---

### References

  - [Kevvie Fowler - Black Hat USA 2007 - SQL Server Database Forensics](http://www.orkspace.net/secdocs/Conferences/BlackHat/USA/2007/SQL%20Server%20Database%20Forensics%20.pdf)

  - [Microsoft - SQL Server transaction log architecture and management guide](https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-log-architecture-and-management-guide?view=sql-server-ver17#virtual-log-files-vlfs)

  - [Kraig Kerr - Reading SQL Server's Transaction Log](https://www.sqlservercentral.com/articles/reading-sql-servers-transaction-log)

  - [Paul Randal & Erin Stellato - The SQL Server Transaction Log, Part 4: Log Records](https://sqlperformance.com/2022/07/transaction-log/part-4-log-records)

  - [Microsoft - MSSQLSERVER_18456 (login failed)](https://learn.microsoft.com/en-us/sql/relational-databases/errors-events/mssqlserver-18456-database-engine-error)

  - [Microsoft - SQL Server Audit records](https://learn.microsoft.com/en-us/sql/relational-databases/security/auditing/sql-server-audit-records)

  - [Microsoft - sys.fn_get_audit_file](https://learn.microsoft.com/en-us/sql/relational-databases/system-functions/sys-fn-get-audit-file-transact-sql)

  - [Microsoft - sys.fn_xe_file_target_read_file](https://learn.microsoft.com/en-us/sql/relational-databases/system-functions/sys-fn-xe-file-target-read-file-transact-sql)

  - [Microsoft - Default trace](https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-default-trace-default-trace)

  - [Microsoft - View the SQL Server error log](https://learn.microsoft.com/en-us/sql/tools/configuration-manager/viewing-the-sql-server-error-log)

  - [Microsoft - SQL Server Agent error log](https://learn.microsoft.com/en-us/sql/ssms/agent/view-the-sql-server-agent-error-log-sql-server-management-studio)

  - [NetSPI / Scott Sutherland - SQL Server Detective Control Cheat Sheet](https://github.com/NetSPI/PowerUpSQL/wiki/SQL-Server-Detective-Control-Cheat-Sheet)

  - [Microsoft - SQL Server Audit action groups and actions](https://learn.microsoft.com/en-us/sql/relational-databases/security/auditing/sql-server-audit-action-groups-and-actions)

  - [Sql Quantum Leap - Server Audit Filter Values For [action_id]](https://sqlquantumleap.com/reference/server-audit-filter-values-for-action_id/)

  - [SQLShack - Esat Erkec - Using SQL Server Extended Events to monitor query performance](https://www.sqlshack.com/using-sql-server-extended-events-to-monitor-query-performance/)

  - [datanumen - SQL Server Extended Events: Monitor, Diagnose & Troubleshoot SQL Server](https://www.datanumen.com/sql-recovery/guides/sql-server-extended-events/#11_What_Are_SQL_Server_Extended_Events)