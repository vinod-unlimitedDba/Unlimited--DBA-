

<details> 
<summary> SQL Server Backup Terminologies and Advanced Examples </summary>

This document compiles key terminologies, concepts, and advanced examples related to SQL Server backups. It is structured for clarity, with definitions, explanations, pros/cons, commands, and demo scripts. The content is based on standard SQL Server practices for backup and recovery.
Key Terminologies
Media Set
A media set is an ordered collection of backup media, tapes, or disk files to which one or more backup operations have written. A media set is a collection of backup set(s).
Backup Set
A backup set contains the backup from a single, successful backup operation. It can be a FULL, Differential, or Transaction Log (TLog) backup.
Media Set Family
Backups created on a single non-mirrored device or a set of mirrored devices in a media set constitute a media family. Backups created in a single media set are called media set family.
Backup Device
A backup device is either a tape or disk file provided by the operating system. It is a stored path.
Checksum
Checksum is additional bit values added to ensure data consistency. Checksum is verified to check if data is corrupted or not.
Recovery Models
Recovery models are designed to control transaction log maintenance. They control the behavior of the log file. The recovery models in SQL Server are Full, Bulk Logged, and Simple.
Full Recovery Model: Every transaction is logged into the Transaction Log file. Growth of the TLog is very fast, so regular TLog backups are recommended. When a TLog backup is taken, all committed and uncommitted log records are backed up, but only committed log records are truncated from the Transaction Log file.


Pros: Recommended for production environments; point-in-time recovery possible; almost no data loss.


Cons: Faster log file growth requiring additional storage; costly due to disk space.


Bulk Logged Recovery Model: Every transaction is logged except Bulk Insert statements, which are minimally logged (only LSN number, Page/Extent details, and Transaction ID). Point-in-time recovery not possible if Bulk Inserts are run. Log file growth is moderate. Used during Bulk Insert operations only. Recommended to take Full backups before and after switching to this model.


Simple Recovery Model: Every transaction is logged, but the log file is truncated at regular intervals (on checkpoint).


Pros: Used in non-critical environments (testing/development); automatic log maintenance; controlled log growth.


Cons: TLog backups not possible; point-in-time recovery not possible.


Verify Option
Once a backup is complete, validate the backup.
Types of Backups in SQL Server
Backup is the process of creating a copy of the database. Major types include:
Full Database Backup: Captures the entire database content, including paths/structures of MDF, NDF, LDF files and their sizes, all pages, entire active log, and last LSN number. Contents of the active portion of the log file are backed up. Full backup NEVER truncates log file(s).


Command: BACKUP DATABASE DBName TO DISK='F:\DBName.bak'


Differential Backup: Tracks and backs up changes since the last Full backup. Cumulative; refers to Differential Changed Map (DCM) page. Active log is backed up.


Command: BACKUP DATABASE DBName TO DISK=N'F:\DBName_Diff.bak' WITH DIFFERENTIAL


Transaction Log Backup: Backs up log records from the last Full (first time) or last TLog backup. Incremental; truncates log records after backup. Not possible in Simple recovery model.


Command: BACKUP LOG DBName TO DISK=N'F:\DBName1.trn'


File and Filegroup Backup: Backs up individual files/filegroups, useful for very large databases.


Commands:


BACKUP DATABASE FGTest FILEGROUP='Primary' TO DISK=N'F:\FGTest.bak'


BACKUP DATABASE FGTest FILE='FGTest3' TO DISK=N'F:\FGTest1.bak'


Advanced Backup Types
Striped Backup: Splits backups into different locations for space constraints. Maximum 64 striped backup sets.


Command: BACKUP DATABASE DBname TO DISK=N'F:\DBNamePart1.bak', DISK=N'E:\DBNamePart2.bak'


Mirror Backup: Creates mirrored copies for safety. Maximum 4 mirrored sets. Must use WITH FORMAT.


Command: BACKUP DATABASE DBname TO DISK=N'F:\DBName_Mirr1.bak' MIRROR TO DISK=N'E:\DBName_Mirr2.bak' WITH FORMAT


COPY_ONLY Backups: Backups without affecting the backup cycle (introduced in SQL Server 2005). For Full and TLog only. COPY_ONLY Full does not disturb differential chain; COPY_ONLY TLog does not truncate logs.


Commands:


BACKUP DATABASE DBName TO DISK=N'f:\DBName.bak' WITH COPY_ONLY


BACKUP LOG DBName TO DISK=N'F:\DBName.trn' WITH COPY_ONLY


Note: If DIFFERENTIAL and COPY_ONLY are both specified, COPY_ONLY is ignored. Not available via GUI in 2005, but in 2008+ it is.


Tail Log Backup: TLog backup during crash; backs up active log.


Command: BACKUP LOG DBName TO DISK=N'f:\DBName_Tail.trn' WITH NO_TRUNCATE


Partial Backup: Excludes read-only filegroups.


Command: BACKUP DATABASE DBName READ_WRITE_FILEGROUPS,FILEGROUP='FG2' TO DISK=N'F:\DBName_PBKP.bak'


Partial Differential Backup: Differential for partial backup.


Command: BACKUP DATABASE DBName READ_WRITE_FILEGROUPS TO DISK=N'F:\DBName_PBKP.bak' WITH DIFFERENTIAL


Cold Backups: Take database offline, copy MDF/LDF files.


ALTER DATABASE DBName SET OFFLINE WITH ROLLBACK IMMEDIATE


Backup Parameters
NOINIT/INIT: Append (default) or overwrite backup sets.


NOFORMAT/FORMAT: Keep (default) or create new media set.


NOSKIP/SKIP: Check (default) or ignore expiration.


BUFFERCOUNT: Number of I/O buffers.


MAXTRANSFERSIZE: Transfer unit size (multiples of 64KB up to 4MB).


BLOCKSIZE: Physical block size (default based on device).


STATS [=percentage]: Progress messages (default 10%).


To confirm differential base: Use RESTORE HEADERONLY and compare CheckpointLSN with DatabaseBackupLSN.
Advanced Demos
COPY_ONLY Full Backup Demo
text
BACKUP DATABASE CODB TO DISK=N'C:\OurBackups\CODB_Full1.bak'
BACKUP DATABASE CODB TO DISK=N'C:\OurBackups\CODB_Diff1.bak' WITH DIFFERENTIAL
BACKUP DATABASE CODB TO DISK=N'C:\OurBackups\CODB_Diff2.bak' WITH DIFFERENTIAL
BACKUP DATABASE CODB TO DISK=N'C:\OurBackups\CODB_Full2.bak' WITH COPY_ONLY
BACKUP DATABASE CODB TO DISK=N'C:\OurBackups\CODB_Diff3.bak' WITH DIFFERENTIAL
BACKUP DATABASE CODB TO DISK=N'C:\OurBackups\CODB_Diff4.bak' WITH DIFFERENTIAL
RESTORE HEADERONLY FROM DISK=N'C:\OurBackups\CODB_Full1.bak'
-- (Repeat for all files)

COPY_ONLY Log Backup Demo
text
BACKUP DATABASE CODB TO DISK=N'C:\OurBackups\CODB_Full1.bak'
USE CODB GO CREATE TABLE T1 (Sno int, Sname varchar(50))
INSERT INTO T1 VALUES (1,'Avengers') BACKUP LOG CODB TO DISK=N'C:\OurBackups\CODB_Tlog1.trn'
-- (Additional inserts and backups)
RESTORE HEADERONLY FROM DISK=N'C:\OurBackups\CODB_Full1.bak'
-- (Repeat for all files)

Another COPY_ONLY Demo
text
BACKUP DATABASE Hello TO DISK='C:\OurBackups\Hello_Full.bak'
BACKUP DATABASE Hello TO DISK='C:\OurBackups\Hello_Diff1.bak' WITH DIFFERENTIAL
BACKUP LOG Hello TO DISK='C:\OurBackups\Hello_Tlog1.trn'
-- (Additional backups including COPY_ONLY)

Restore Scenarios
Complete Database Restore (Point of Failure)
Involves restoring Full, Differential, TLogs, and Tail Log. Example backup schedule: Full on Sunday, Diff daily, TLog hourly.
Backup Steps Example:
text
CREATE DATABASE FRIENDSHIP
USE FRIENDSHIP GO
CREATE TABLE T1(Sno int, Sname varchar(50)) GO
INSERT INTO T1 VALUES (1,'T1') GO 100001
BACKUP DATABASE Friendship TO DISK=N'C:\OurBackups\Friendship_Full.bak'
-- (Additional inserts and backups)

Restore Steps:
text
BACKUP LOG FriendShip TO DISK='C:\OurBackups\FriendShip_Tail.trn' WITH NO_TRUNCATE
RESTORE DATABASE [Friendship] FROM DISK = N'D:\Backup\Friendship_Full.bak' WITH MOVE ..., NORECOVERY, REPLACE, STATS = 10 GO
-- (Restore Diff, TLogs, Tail with NORECOVERY, final with RECOVERY)

Point-in-Time Restore
Restore to a specific time or LSN for recovering dropped objects.
Example:
text
CREATE DATABASE PITDB
-- (Create table, inserts, backups)
-- Find LSN for drop: select ... from fn_dblog(null, null) where [Operation] = 'LOP_BEGIN_XACT' and [Transaction Name]='DROPOBJ'
-- Or from backup: SELECT ... FROM fn_dump_dblog(...)

Restore Commands:
text
RESTORE DATABASE [PITDB_Copy] FROM DISK = N'C:\OurBackups\PIT_Full.bak' WITH ... NORECOVERY
-- (Restore Diff, TLogs)
RESTORE LOG [PITDB_Copy] FROM DISK = N'C:\OurBackups\PIT_Tlog3.trn' WITH RECOVERY, STOPAT = N'Jul 10, 2017 07:06:06:136 AM'
-- Or STOPBEFOREMARK with LSN

File/Filegroup Restore
Restore specific files/filegroups.
Example Setup:
text
CREATE DATABASE [FFG] ON PRIMARY (...) FILEGROUP [FG1] (...) -- etc.
-- Create tables, inserts, backups

Restore:
text
BACKUP LOG FFG TO DISK=N'C:\OurBackups\FFG_Tail.trn' WITH NO_TRUNCATE
RESTORE DATABASE [FFG] FILEGROUP='PRIMARY' FROM ... WITH NORECOVERY,REPLACE GO
-- (Restore others, final with RECOVERY)

Piecemeal Restore
Restore in stages, starting with Primary filegroup (SQL Server 2005+).
Example:
text
RESTORE DATABASE FFGRESTORE FILEGROUP='PRIMARY' FROM ... WITH REPLACE,NORECOVERY,PARTIAL
-- (Restore other filegroups separately, apply logs)

Page Restore
Restore damaged pages.
Example:
text
CREATE DATABASE TestPageLevelRestore
-- (Create table, inserts, backups)
-- Corrupt page, check with DBCC CHECKDB
BACKUP LOG ... WITH NORECOVERY
RESTORE DATABASE TestPageLevelRestore PAGE='1:312' FROM ... WITH NORECOVERY
-- (Restore Diff, TLogs, final with RECOVERY)

Phases of Restore/Recovery
Data Copy Phase: Initializes contents.


Roll Forward (Redo): Redoes logged changes.


Recovery Point: Target state.


Roll Backward (Undo): Undoes uncommitted transactions.


Types of Recovery
Restart Recovery: On instance restart or database offline/online.


Restore Recovery: Manual recovery per backup sequence.


Third-Party Backup Tools
Redgate SQL Backup


Quest Litespeed


Veritas Net Backup


Tivoli Storage Manager (IBM)


And others like Legato Networker, Symantec BackupExec, etc.


Compressed Backups
Introduced in SQL Server 2008 Enterprise (Standard+ in 2008 R2). Enable with sp_configure 'backup compression default',1.
Command: BACKUP DATABASE GroupTest TO DISK=N'f:\GT.bak' WITH COMPRESSION


Calculate ratio: SELECT backup_size/compressed_backup_size FROM msdb..backupset


Backup Metadata
Query MSDB tables: SELECT * FROM dbo.backupset, dbo.backupmediafamily, dbo.restorehistory.
Script to Query Backup Info:
text
SELECT CONVERT(CHAR(100), SERVERPROPERTY('Servername')) AS Server, ... FROM msdb.dbo.backupmediafamily INNER JOIN msdb.dbo.backupset ... ORDER BY ...

SELECT CONVERT(CHAR(100), SERVERPROPERTY('Servername')) AS Server, ... FROM msdb.dbo.backupmediafamily INNER JOIN msdb.dbo.backupset ... ORDER BY ...



Backup Terminologies:
Recovery Models
Media Set: A media set is an ordered collection of backup media, tapes or disk files, to which one or more backup operations have written.
Media set is a collection of backupset(s).
Backup Set: A backup set contains the backup from a single, successful backup operation. It can be a FULL/Diff/Tlog backup.
Media Set Family: Backups created on a single nonmirrored device or a set of mirrored devices in a media set constitute a media family.
Backups created in a single media set are called mediaset family.
Backup Device: A backup device is either a tape or disk file that is provided by the operating system. Its a stored path.
Checksum: Checksum is a additional bit values added to ensure the data is consistent. Checksum is verified to ensure if data is corrupted or not.
Recovery Models: Recovery models are designed to control transaction log maintenance.
Recovery models control the behavior of the log file.
The recovery models in SQL Server are Full, Bulk Logged and Simple.
Full Recovery Model: Every Transaction in Full Recovery Model is logged into the Transaction Log file.
Growth of the Tlog is very fast and drastic in this recovery model, so it is a recommended Practice to take regular Tlog Backups.
When tlog backup is taken all the committed and uncommitted log records will be backed up and only committed log records will be truncated from Transaction Log file.
Pros: 1)Full Recovery Model is recommended in Production Environment. 2)Point-in-time recovery is possible only in this recovery model. 3)Almost no data loss
Cons:
Growth of log file is faster and requires additional storage
Costly due to disk space requirements
Bulk Logged Recovery Model: Every transaction is logged into the transaction log file except Bulk Insert statements which are minimally logged.
Minimally logged operation means only few details are available in the log file like LSN number, Page/Extent Details and Transaction ID when compared to Full Recovery Model.
Point-in-time recovery not possible. Possible if no BULK INSERT commands are run.
Log file growth is not too large, but some amount of growth would be there.
Used especially during BULK INSERT statements ONLY.
It is always recommended practice to take a FULL backup pre and post changing recovery model to BULK LOGGED.
Simple Recovery Model: Every transaction is logged into the Transaction Log file. But at regular intervals Transaction Log file gets Truncated.
Pros:
This recovery model is used in Non-Critical environments (Testing and Development)
Automatic Maintenance of Log file, log file growth can be controlled.
In simple recovery model, Log File Truncation occurs when ever Checkpoint occurs.
Cons: 1)It is not possible to take a Tlog backup in Simple Recovery Model.
2)Point-in-time recovery is not possibleVerify Option, once backup is complete Validate the backup.
Backup is a process of creating a copy of the database. There are in major 4 types of backups in SQL Server.
Full Backup
Differential Backup
Transaction Log Backup
File and Filegroup Backup
Full Database Backup:- A full database backup captures the entire database content.
A full database backup copies all the pages from a database onto a backup device/media set.
Full backup contains: a) The path/location and structures of all the MDF, NDF and LDF and their sizes
b) Copy all the pages to the media set. Entire Active Log and Last LSN number
c) Contents of the Active Portion of the Log file (i.e. Log Records) are backed up by Full Backup.
Command: BACKUP DATABASE DBName TO DISK='F:\DBName.bak'
NOTE:- FULL Backup NEVER truncates log file(s).
Differential Backup: Differential backup tracks and does backup of all the changes that have happenned from Last Full Backup till the backup completion time.
All differential backups are cumulative. Even if one backup in the mid of the week is lost, the recent backup can be used.
During differential backup, SQL Server refers DCM(Differential Changed Map) Page to track all the extents modified and copying the pages inside them into the BAK file. Also the active log is backed up as part of Differential backup.
Command: BACKUP DATABASE DBName TO DISK=N'F:\DBName_Diff.bak' WITH DIFFERENTIAL
Transaction Log Backups: Backups all the transaction log records from the last FULL backup [First Time Log Backup Only] (or) last Transaction Log backup.
Log backups are Incremental. Once log backup is completed, it truncates the log records from log file.
For SIMPLE recovery model database, it is not possible to take log backup.
Command: BACKUP LOG DBName TO DISK=N'F:\DBName1.trn'
File and Filegroup Backup: File and Filegroup backups are individual backups of Files/Filegroups in a database.
F&FG backups are helpful to devise backup strategies for Very Large DBs.
Individual Files and Filegroups are backed up using this method, also tables can be placed in FG and can be backed up individually.
Command: BACKUP DATABASE FGTest FILEGROUP='Primary' TO DISK=N'F:\FGTest.bak'
BACKUP DATABASE FGTest FILE='FGTest3' TO DISK=N'F:\FGTest1.bak'
Striped Backup: During space constraints backups can be split into different locations.
Command: BACKUP DATABASE DBname TO DISK=N'F:\DBNamePart1.bak', DISK=N'E:\DBNamePart2.bak'
Maximum 64 striped backupsets are possible.
Mirror Backup: For additional safety to backup files it is possible to create mirrored copies of database backups. If one file gets corrupted we have another as safety measure.
BACKUP DATABASE DBname TO DISK=N'F:\DBName_Mirr1.bak' MIRROR TO DISK=N'E:\DBName_Mirr2.bak' WITH FORMAT
Mirrored backupsets cannot reside with other NonMirrored backupsets, hence WITH FORMAT option is mandatory to mention.
Maximum 4 mirrored sets are possible.
COPY_ONLY Backups: Backups that are taken without affecting the Backup Cycle. COPY_ONLY. This concept was implemented from SQL Server 2005 onwards.
COPY_ONLY full backups do not disturb the differential backup chain.
COPY_ONLY log backups do not disturb the log chain (because COPY ONLY Log backups don't truncate the log records).
Command: BACKUP DATABASE DBName TO DISK=N'f:\DBName.bak' WITH COPY_ONLY BACKUP LOG DBName TO DISK=N'F:\DBName.trn' WITH COPY_ONLY
Note:
COPY_ONLY backups are possible for FULL and TLogs Only.
If COPY_ONLY, DIFFERENTIAL is mentioned in the backup command then COPY_ONLY is ignored.
Backup database DBName to disk=N'f:\DBName.bak' WITH DIFFERENTIAL,COPY_ONLY
COPY_ONLY backup cannot be taken through GUI and is possible through commands only (2005 Only). In 2008 it is possible to take COPY_ONLY through GUI.
Summary:- COPY_ONLY full backup will not disturb the differential backup chain. COPY_ONLY log backup will not disturb the log chain.
Command: BACKUP LOG Friendship to disk=N'D:\Backup\Friendship_CopyOnly_Log1.trn' WITH COPY_ONLY

COPYONLY Full Backup Demo:- BACKUP DATABASE CODB TO DISK=N'C:\OurBackups\CODB_Full1.bak'
BACKUP DATABASE CODB TO DISK=N'C:\OurBackups\CODB_Diff1.bak' WITH DIFFERENTIAL
BACKUP DATABASE CODB TO DISK=N'C:\OurBackups\CODB_Diff2.bak' WITH DIFFERENTIAL
BACKUP DATABASE CODB TO DISK=N'C:\OurBackups\CODB_Full2.bak' WITH COPY_ONLY
BACKUP DATABASE CODB TO DISK=N'C:\OurBackups\CODB_Diff3.bak' WITH DIFFERENTIAL
BACKUP DATABASE CODB TO DISK=N'C:\OurBackups\CODB_Diff4.bak' WITH DIFFERENTIAL
RESTORE HEADERONLY FROM DISK=N'C:\OurBackups\CODB_Full1.bak' RESTORE HEADERONLY FROM DISK=N'C:\OurBackups\CODB_Diff1.bak' RESTORE HEADERONLY FROM DISK=N'C:\OurBackups\CODB_Diff2.bak' RESTORE HEADERONLY FROM DISK=N'C:\OurBackups\CODB_Full2.bak' RESTORE HEADERONLY FROM DISK=N'C:\OurBackups\CODB_Diff3.bak' RESTORE HEADERONLY FROM DISK=N'C:\OurBackups\CODB_Diff4.bak'

COPYONLY Log Backup Demo:- BACKUP DATABASE CODB TO DISK=N'C:\OurBackups\CODB_Full1.bak'
Use CODB GO CREATE TABLE T1 (Sno int, Sname varchar(50))
INSERT INTO T1 VALUES (1,'Avengers') BACKUP LOG CODB TO DISK=N'C:\OurBackups\CODB_Tlog1.trn'
INSERT INTO T1 VALUES (2,'Jersey') BACKUP LOG CODB TO DISK=N'C:\OurBackups\CODB_Tlog2.trn'
INSERT INTO T1 VALUES (3,'Kanchana3') BACKUP LOG CODB TO DISK=N'C:\OurBackups\CODB_Tlog3.trn'
INSERT INTO T1 VALUES (4,'Kavaludaari') BACKUP LOG CODB TO DISK=N'C:\OurBackups\CODB_Tlog4_CO.trn' WITH COPY_ONLY
INSERT INTO T1 VALUES (5,'Bharat') BACKUP LOG CODB TO DISK=N'C:\OurBackups\CODB_Tlog5.trn'
RESTORE HEADERONLY FROM DISK=N'C:\OurBackups\CODB_Full1.bak' RESTORE HEADERONLY FROM DISK=N'C:\OurBackups\CODB_Tlog1.trn' RESTORE HEADERONLY FROM DISK=N'C:\OurBackups\CODB_Tlog2.trn' RESTORE HEADERONLY FROM DISK=N'C:\OurBackups\CODB_Tlog3.trn' RESTORE HEADERONLY FROM DISK=N'C:\OurBackups\CODB_Tlog4_CO.trn' RESTORE HEADERONLY FROM DISK=N'C:\OurBackups\CODB_Tlog5.trn'

Demo:- BACKUP DATABASE Hello TO DISK='C:\OurBackups\Hello_Full.bak'
BACKUP DATABASE Hello TO DISK='C:\OurBackups\Hello_Diff1.bak' WITH DIFFERENTIAL
BACKUP LOG Hello TO DISK='C:\OurBackups\Hello_Tlog1.trn'
BACKUP DATABASE Hello TO DISK='C:\OurBackups\Hello_Diff2.bak' WITH DIFFERENTIAL
BACKUP LOG Hello TO DISK='C:\OurBackups\Hello_Tlog2.trn'
BACKUP DATABASE Hello TO DISK='C:\OurBackups\Hello_Full_CopyOnly.bak' WITH COPY_ONLY
BACKUP DATABASE Hello TO DISK='C:\OurBackups\Hello_Diff3.bak' WITH DIFFERENTIAL
BACKUP LOG Hello TO DISK='C:\OurBackups\Hello_Tlog3.trn'
BACKUP DATABASE Hello TO DISK='C:\OurBackups\Hello_Diff4.bak' WITH DIFFERENTIAL
BACKUP LOG Hello TO DISK='C:\OurBackups\Hello_Tlog4.trn'
BACKUP LOG Hello TO DISK='C:\OurBackups\Hello_Tlog_CopyOnly.trn' WITH COPY_ONLY
BACKUP LOG Hello TO DISK='C:\OurBackups\Hello_Tlog5.trn'

Tail Log Backup: Tail log backup is a Tlog backup, that can be attempted during a crash situation. Tail log also takes backup of Active Log.
Command: BACKUP LOG DBName TO DISK=N'f:\DBName_Tail.trn' WITH NO_TRUNCATE
Tail log can fail if Log file gets corrupted.
Partial Backup: Partial backups are useful whenever we want to exclude read-only filegroups. A partial does not contain all the filegroups.
Command: BACKUP DATABASE DBName READ_WRITE_FILEGROUPS,FILEGROUP='FG2' TO DISK=N'F:\DBName_PBKP.bak'
Partial Differential Backup: Is a differential backup for Partial backup.
BACKUP DATABASE DBName READ_WRITE_FILEGROUPS TO DISK=N'F:\DBName_PBKP.bak' WITH DIFFERENTIAL
Cold Backups in SQL Server: If backup is being taken for one specific database then.
ALTER DATABASE DBName SET OFFLINE WITH ROLLBACK IMMEDIATE
Copy MDF & LDF files of this database in Data Directory. This files backup taken can be attached to another instance if required to restore.
Parameters for CUI based backups:
NOINIT/INIT: NOINIT appends backupsets into the media set. (Default) INIT overwrites existing backupsets into the media set.
NOFORMAT/FORMAT: NOFORMAT will keep the existing media set. (Default) FORMAT will create a new media set and overwrites all the contents in the file.
NOSKIP/SKIP: NOSKIP instructs the BACKUP statement to check the expiration date of all backup sets on the media before allowing them to be overwritten. (Default) SKIP ignores the backup set expiration.
BUFFERCOUNT Specifies the total number of I/O buffers to be used for the backup operation. You can specify any positive integer.
The total space used by the buffers is determined by: BUFFERCOUNT * MAXTRANSFERSIZE
MAXTRANSFERSIZE Specifies the largest unit of transfer in bytes to be used between SQL Server and the backup media. The possible values are multiples of 65536 bytes (64 KB) ranging up to 4194304 bytes (4 MB).
BLOCKSIZE Specifies the physical block size, in bytes. The supported sizes are 512, 1024, 2048, 4096, 8192, 16384, 32768, and 65536 (64 KB) bytes. The default is 65536 for tape devices and 512 otherwise.
BLOCKSIZE is selected automatically based on the device chosen.
STATS [ =percentage ] Displays a message each time another percentage completes, and is used to gauge progress. If percentage is omitted, SQL Server displays a message after each 10 percent is completed.
To confirm the differential base (FULL BACKUP) of a differential backup use the command
RESTORE HEADERONLY FROM DISK='C:\MyFiles\KDSSDB.bak'
RESTORE HEADERONLY FROM DISK='C:\MyFiles\KDSSDB_Diff1.bak'
RESTORE HEADERONLY FROM DISK='C:\MyFiles\KDSSDB_Diff1.bak'
Compare the CheckpointLSN of Full Backup with DatabaseBackupLSN of differential backups, if they are same then the base is valid. If not may mean someone has triggered a full backup in mid of the week.
Restore Scenarios:-
Complete Database Restore (Point of Failure) ***
Point In Time Restore
File/Filegroup Restore
Piece Meal Restore
Page Restore
Complete Database Restore (Point of Failure): A complete database restore involves restoring a full database backup followed by restoring and recovering a differential backup. Point of Failure restore can be done when database is crashed and we will have to perform the restores with all available backups including tail log we have taken before starting sequence of restores.
Full Backup - Sunday 9:00 Diff Backup - EveryDay 9:00 Tlog Backup - 1 Hour
Wednesday 15:30
Take Tail Log Backup
Restore Full Backup
Restore recent Diff Backup (Wednesday 9:00)
Restore all the Log Backups from 9:00 Wednesday to 15:00.
Restore Tail Log Backup (WITH RECOVERY)
Backup Steps:- CREATE DATABASE FRIENDSHIP
USE FRIENDSHIP GO
CREATE TABLE T1(Sno int, Sname varchar(50)) GO
INSERT INTO T1 VALUES (1,'T1') GO 100001
--1,00,001 Records before Full Backup BACKUP DATABASE Friendship TO DISK=N'C:\OurBackups\Friendship_Full.bak'
INSERT INTO T1 VALUES (1,'T1') INSERT INTO T1 VALUES (1,'T1')
--1,00,003 Records before Diff1 Backup, so Diff1 Backup takes --backup of two records only. BACKUP DATABASE Friendship TO DISK=N'C:\OurBackups\Friendship_Diff1.bak' WITH DIFFERENTIAL
INSERT INTO T1 VALUES (1,'T1') INSERT INTO T1 VALUES (1,'T1')
--1,00,005 Records before Tlog1 Backup, so Log1 Backup takes --backup BACKUP LOG Friendship TO DISK=N'C:\OurBackups\Friendship_Log1.trn'
INSERT INTO T1 VALUES (1,'T1') INSERT INTO T1 VALUES (1,'T1')
----1,00,007 Records -> Last two Log Records after Tlog1 and before Tlog2 Backup will be backup up. BACKUP LOG Friendship TO DISK=N'C:\OurBackups\Friendship_Log2.trn'
INSERT INTO T1 VALUES (1,'T1') INSERT INTO T1 VALUES (1,'T1')
BEGIN TRAN INSERT INTO T1 VALUES (1,'T1') INSERT INTO T1 VALUES (1,'T1')
--Corrupt the Database. ALTER DATABASE Friendship SET OFFLINE WITH ROLLBACK IMMEDIATE
Corrupted the MDF file @ OS level.
ALTER DATABASE Friendship SET ONLINE
---:Restore Steps:--- BACKUP LOG FriendShip TO DISK='C:\OurBackups\FriendShip_Tail.trn' WITH NO_TRUNCATE
RESTORE DATABASE [Friendship] FROM DISK = N'D:\Backup\Friendship_Full.bak' WITH MOVE N'Friendship' TO N'D:\Backup\Friendship.mdf', MOVE N'Friendship_log' TO N'D:\Backup\Friendship.ldf', NORECOVERY, REPLACE, STATS = 10 GO
RESTORE DATABASE [Friendship] FROM DISK = N'D:\Backup\Friendship_Diff1.bak' WITH NORECOVERY, STATS = 10 GO
RESTORE LOG [Friendship] FROM DISK=N'D:\Backup\Friendship_Log1.trn' WITH NORECOVERY, STATS = 10 GO
RESTORE LOG [Friendship] FROM DISK=N'D:\Backup\Friendship_Log2.trn' WITH NORECOVERY, STATS = 10 GO
RESTORE LOG [Friendship] FROM DISK=N'D:\Backup\Friendship_Tail.trn' WITH NORECOVERY, STATS = 10 GO
RESTORE DATABASE [Friendship] WITH RECOVERY
Point-In-Time restore Point in time restores can be done when database objects have been dropped and there is request to restore specific objects. We will firt restore database and the object to another/same instance and restore all backups using STOPAT, STOPBEFOREMARK and finally export/import objects to the instance.
CREATE DATABASE PITDB
USE PITDB GO
create table Table1 (sno int, sname varchar(50)) insert into Table1 values (1,'Pit1') insert into Table1 values (2,'Pit2') insert into Table1 values (3,'Pit3') insert into Table1 values (4,'Pit4')
--10:16AM (Full Backup) backup database PITDB to disk=N'C:\OurBackups\PIT_Full.bak'
--10:17AM 4 Inserts after Full insert into Table1 values (5,'Pit5') insert into Table1 values (6,'Pit6') insert into Table1 values (7,'Pit7') insert into Table1 values (8,'Pit8')
--10:18AM (Diff Backup) backup database PITDB to disk=N'C:\OurBackups\PIT_Diff.bak' with differential
--10:19AM 4 Inserts after Diff insert into Table1 values (9,'Pit9') insert into Table1 values (10,'Pit10') insert into Table1 values (11,'Pit11') insert into Table1 values (12,'Pit12')
--10:20AM (TLog1 Backup) backup log PITDB to disk=N'C:\OurBackups\PIT_Tlog1.trn'
--10:21AM 4 Inserts after Tlog1 insert into Table1 values (13,'Pit13') insert into Table1 values (14,'Pit14') insert into Table1 values (15,'Pit15') insert into Table1 values (16,'Pit16')
--10:22AM (TLog2 Backup) backup log PITDB to disk=N'C:\OurBackups\PIT_Tlog2.trn'
--10:23AM 4 Inserts after Tlog2 insert into Table1 values (17,'Pit17') insert into Table1 values (18,'Pit18') insert into Table1 values (19,'Pit19') insert into Table1 values (20,'Pit20')
begin tran insert into Table1 values (21,'Pit21') insert into Table1 values (22,'Pit22') insert into Table1 values (23,'Pit23') insert into Table1 values (24,'Pit24') commit
--10:24 Drop the Table. DROP TABLE Table1
create table Table2 (sno int, sname varchar(50)) insert into Table2 values (1,'Pit1') insert into Table2 values (2,'Pit2') insert into Table2 values (3,'Pit3') insert into Table2 values (4,'Pit4')
--10:25AM (TLog3 Backup) backup log PITDB to disk=N'C:\OurBackups\PIT_Tlog3.trn'

After restoring based on timestamp provided by Application team member, if we restore all backups and if Application Owner is not happy with the returned table and number of records in it. They may provide one more estimated timeline. It is better to check from DBA end the exact LSN number of Table Drop.
select [Current LSN], [Operation], [Transaction ID], [Parent Transaction ID], [Begin Time], [Transaction Name], [Transaction SID] from fn_dblog(null, null) where [Operation] = 'LOP_BEGIN_XACT' and [Transaction Name]='DROPOBJ'
To find person who dropped the table use: select suser_sname(0x010500000000000515000000BF3E9BCF15EC2EA66C398C33E8030000)
If log file was truncated, then we can use fn_dump_dblog to retrieve LSN information from Log Backup.
SELECT Operation,[Current LSN],[Transaction Name] FROM fn_dump_dblog ( DEFAULT, DEFAULT, DEFAULT, DEFAULT, 'C:\OurBackups\PIT_Tlog3.trn', DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT, DEFAULT ) where OPERATION='LOP_BEGIN_XACT' and [Transaction Name]='DROPOBJ' GO
--For Point in Time recovery mention STOPAT parameter with appropriate timeline (or) STOPBEFOREMARK with LSN Number.
USE [master] RESTORE DATABASE [PITDB_Copy] FROM DISK = N'C:\OurBackups\PIT_Full.bak' WITH FILE = 1, MOVE N'PITDB' TO N'C:\Program Files\Microsoft SQL Server\MSSQL11.INST2K12_2\MSSQL\DATA\PITDB_Copy.mdf', MOVE N'PITDB_log' TO N'C:\Program Files\Microsoft SQL Server\MSSQL11.INST2K12_2\MSSQL\DATA\PITDB_Copy_log.ldf', NORECOVERY, NOUNLOAD, STATS = 5 RESTORE DATABASE [PITDB_Copy] FROM DISK = N'C:\OurBackups\PIT_Diff.bak' WITH NORECOVERY
RESTORE LOG [PITDB_Copy] FROM DISK = N'C:\OurBackups\PIT_Tlog1.trn' WITH NORECOVERY
RESTORE LOG [PITDB_Copy] FROM DISK = N'C:\OurBackups\PIT_Tlog2.trn' WITH NORECOVERY
RESTORE LOG [PITDB_Copy] FROM DISK = N'C:\OurBackups\PIT_Tlog3.trn' WITH RECOVERY, STOPAT = N'Jul 10, 2017 07:06:06:136 AM'
--Restores to specific LSN Number. RESTORE LOG [PITDB_Copy] FROM DISK = N'C:\OurBackups\PIT_Tlog3.trn' WITH STOPBEFOREMARK = 'lsn:0x00000020:00000161:0001' GO
--Restores to specific Time provided. RESTORE LOG [PITDB_Copy] FROM DISK = N'C:\OurBackups\PIT_Tlog3.trn' WITH STOPATMARK = N'lsn:0x00000020:00000161:0001' GO
--Prefix 0x in the LSN number which is identified and prefix --it with 'lsn:0x' for hexadecimal format.

File/Filegroup Restore: Restore one or more damaged read-only files, without restoring the entire database. File restore is available only if the database has at least one read-only filegroup
Create a database with two filegroups and create two tables in each filegroup.
CREATE DATABASE [FFG] ON PRIMARY ( NAME = N'FFG', FILENAME = N'C:\Program Files\Microsoft SQL Server\MSSQL13.KDSSGB28_2K162\MSSQL\DATA\FFG.mdf' , SIZE = 4096KB , FILEGROWTH = 1024KB ), FILEGROUP [FG1] ( NAME = N'FG1', FILENAME = N'C:\Program Files\Microsoft SQL Server\MSSQL13.KDSSGB28_2K162\MSSQL\DATA\FG1.ndf' , SIZE = 4096KB , FILEGROWTH = 1024KB ), FILEGROUP [FG2] ( NAME = N'FG2', FILENAME = N'C:\Program Files\Microsoft SQL Server\MSSQL13.KDSSGB28_2K162\MSSQL\DATA\FG2.ndf' , SIZE = 4096KB , FILEGROWTH = 1024KB ), FILEGROUP [FG3] ( NAME = N'FG3', FILENAME = N'C:\Program Files\Microsoft SQL Server\MSSQL13.KDSSGB28_2K162\MSSQL\DATA\FG3.ndf' , SIZE = 4096KB , FILEGROWTH = 1024KB ) LOG ON ( NAME = N'FFG_log', FILENAME = N'C:\Program Files\Microsoft SQL Server\MSSQL13.KDSSGB28_2K162\MSSQL\DATA\FFG_log.ldf' , SIZE = 1024KB , FILEGROWTH = 10%)
use FFG GO
create table Table1 (Sno int, sname varchar(50)) on FG1
create table Table2 (Sno int, sname varchar(50)) on FG2
insert into Table1 values (1,'One') insert into Table1 values (2,'Ek') insert into Table1 values (3,'Ondu')
insert into Table2 values (1,'Two') insert into Table2 values (2,'Do') insert into Table2 values (3,'Eradu')
--Taking a full database backup of FFG. BACKUP DATABASE FFG TO DISK=N'C:\OurBackups\FFG_FULL.BAK'
--3 Records are being duplicated into both tables. insert into FFG.dbo.Table1 select * from FFG.dbo.Table1 insert into FFG.dbo.Table2 select * from FFG.dbo.Table2
--Differential Backup. BACKUP DATABASE FFG TO DISK=N'C:\OurBackups\FFG_DIFF1.BAK' WITH DIFFERENTIAL
--6 Records are being duplicated into both the tables. insert into FFG.dbo.Table1 select * from FFG.dbo.Table1 insert into FFG.dbo.Table2 select * from FFG.dbo.Table2
--Log Backup of FFG database. BACKUP LOG FFG TO DISK=N'C:\OurBackups\FFG_TLOG1.TRN'
--12 Records are being duplicated into both the tables. insert into FFG.dbo.Table1 select * from FFG.dbo.Table1 insert into FFG.dbo.Table2 select * from FFG.dbo.Table2
For testing, corrupt the database MDF/NDF file.
----:RESTORE COMMANDS:----
BACKUP LOG FFG TO DISK=N'C:\OurBackups\FFG_Tail.trn' WITH NO_TRUNCATE
USE [master] RESTORE DATABASE [FFG] FILEGROUP='PRIMARY' FROM DISK = N'C:\OurBackups\FFG_FULL.BAK' WITH NORECOVERY,REPLACE GO
RESTORE DATABASE [FFG] FROM DISK = N'C:\OurBackups\FFG_Diff1.BAK' WITH NORECOVERY GO
RESTORE LOG [FFG] FROM DISK = N'C:\OurBackups\FFG_TLog1.trn' WITH NORECOVERY GO
RESTORE LOG [FFG] FROM DISK = N'C:\OurBackups\FFG_Tail.trn' WITH RECOVERY GO
If we try to bring database online WITH RECOVERY option it will error out. Because only Primary filegroup was restored and remaining FG1 and FG2 filegroups were not restored. So it is mandatory in File and Filegroup restore that all filegroups including Primary once restored only then database will come online.
RESTORE DATABASE [FFG] FILEGROUP='Primary',FILEGROUP='FG1',FILEGROUP='FG2' FROM DISK = N'C:\OurBackups\FFG_FULL.BAK' WITH NORECOVERY,REPLACE GO
RESTORE DATABASE [FFG] FROM DISK = N'C:\OurBackups\FFG_Diff1.BAK' WITH NORECOVERY GO
RESTORE LOG [FFG] FROM DISK = N'C:\OurBackups\FFG_TLog1.trn' WITH NORECOVERY GO
RESTORE LOG [FFG] FROM DISK = N'C:\OurBackups\FFG_Tail.trn' WITH RECOVERY GO
Now database comes online as all filegroups are restored.
File restore Restores a file or filegroup in a multi-filegroup database.
We must use file and filegroup backup and restore operations in conjunction with transaction log backups. After you restore the files, you must then restore the transaction log backups that were created since the file backups were created to bring the database to a consistent state.
Piecemeal restore Restores the database in stages, beginning with the primary filegroup and one or more secondary filegroups. A piecemeal restore begins with a RESTORE DATABASE using the PARTIAL option and specifying one or more secondary filegroups to be restored.
Piecemeal Restore Piecemeal Restore is a feature of SQL Server 2005 and using Piecemeal restore we can restore file/filegroups as per requirement of Application team. It is mandatory to restore Primary filegroup first before restore all other filegroups.
RESTORE DATABASE FFGRESTORE FILEGROUP='PRIMARY' FROM DISK=N'D:\BACKUP\FFGRESTORE_FULL_041520150653.BAK' WITH REPLACE,NORECOVERY,PARTIAL
RESTORE DATABASE FFGRESTORE FILEGROUP='FG1' FROM DISK=N'D:\BACKUP\FFGRESTORE_FULL_041520150653.BAK' WITH NORECOVERY
RESTORE DATABASE FFGRESTORE FILEGROUP='PRIMARY', FILEGROUP='FG1' FROM DISK=N'D:\BACKUP\FFGRESTORE_DIFF1_041520150656.BAK' WITH NORECOVERY
RESTORE LOG FFGRESTORE FROM DISK=N'D:\BACKUP\FFGRESTORE_TLOG1_041520150657.TRN' WITH RECOVERY
---Restoring FG2 seperately. RESTORE DATABASE FFGRESTORE FILEGROUP='FG2' FROM DISK=N'D:\BACKUP\FFGRESTORE_FULL_041520150653.BAK' WITH NORECOVERY
RESTORE DATABASE FFGRESTORE FILEGROUP='FG2' FROM DISK=N'D:\BACKUP\FFGRESTORE_DIFF1_041520150656.BAK' WITH RECOVERY
RESTORE LOG FFGRESTORE FROM DISK=N'D:\BACKUP\FFGRESTORE_TLOG1_041520150657.TRN' WITH RECOVERY
Page Restore: Restores one or more damaged pages. An unbroken chain of log backups must be available, up to the current log file, and they must all be applied to bring the page up to date with the current log file.
create database TestPageLevelRestore
use TestPageLevelRestore go create table Shift (sno int,sname varchar(50))
insert into Shift values (1,'Hello') insert into Shift values (2,'Hi') insert into Shift values (3,'How Are You')
BACKUP DATABASE TestPageLevelRestore TO DISK=N'C:\OurBackups\TestPageLevelRestore_Full.bak'
insert into Shift values (4,'I am') insert into Shift values (5,'Doing') insert into Shift values (6,'Good, How are you doing?')
BACKUP DATABASE TestPageLevelRestore TO DISK=N'C:\OurBackups\TestPageLevelRestore_Diff.bak' WITH DIFFERENTIAL
insert into Shift values (7,'Great to hear') insert into Shift values (8,'I am Doing Great') insert into Shift values (9,'Where are you going for vacation?')
BACKUP LOG TestPageLevelRestore TO DISK=N'C:\OurBackups\TestPageLevelRestore_Tlog.trn'
Use TestPageLevelRestore GO Select * from sys.indexes where OBJECT_NAME(object_id)='Shift'
DBCC IND('TestPageLevelRestore','Shift',0)
DBCC TRACEON(3604,-1) DBCC PAGE(8,1,276,3)

dbcc page ( {'dbname' | dbid}, filenum, pagenum [, printopt={0|1|2|3} ])
0 print just the page header 1 page header plus per-row hex dumps and a dump of the page slot array (unless its a page that doesn t have one, like allocation bitmaps) 2 page header plus whole page hex dump 3 page header plus detailed per-row interpretation

ALTER DATABASE TestPageLevelRestore SET OFFLINE WITH ROLLBACK IMMEDIATE
--Corrupt the pages using HexEditor or DBCC WRITEPAGE commands --When corrupting page, multiply Page Number*8192. Press Ctrl+G to go page number in decimal format.
ALTER DATABASE TestPageLevelRestore SET ONLINE
use TestPageLevelRestore go select * from Shift
Msg 8928, Level 16, State 1, Line 1 Object ID 2105058535, index ID 0, partition ID 72057594038779904, alloc unit ID 72057594039697408 (type In-row data): Page (1:153) could not be processed. See other errors for details. Msg 8939, Level 16, State 98, Line 1 Table error: Object ID 2105058535, index ID 0, partition ID 72057594038779904, alloc unit ID 72057594039697408 (type In-row data), page (1:153). Test (IS_OFF (BUF_IOERR, pBUF->bstat)) failed. Values are 12716041 and -4.
Run DBCC CHECKDB, to identify if just a single page is corrupt or if multiple pages are corrupt.
--Take Backup with NORECOVERY, this will put database in --RESTORING State.
BACKUP LOG TestPageLevelRestore TO DISK=N'C:\OurBackups\TestPageLevelRestore_NoRecovery.trn' WITH NORECOVERY
--Start the restore sequence.
restore database TestPageLevelRestore PAGE='1:312' from disk=N'C:\OurBackups\TestPageLevelRestore_Full.bak' WITH NORECOVERY
restore database TestPageLevelRestore from disk=N'C:\OurBackups\TestPageLevelRestore_Diff.bak' WITH NORECOVERY
restore log TestPageLevelRestore from disk=N'C:\OurBackups\TestPageLevelRestore_Tlog.trn' WITH NORECOVERY
restore log TestPageLevelRestore from disk=N'C:\OurBackups\TestPageLevelRestore_NoRecovery.trn' WITH RECOVERY
Phases of Restore/Recovery: A restore is a multiphase process. The possible phases of a restore include the data copy, roll forward, Recovery Point, roll backward.
1)Data Copy Phase: The first phase in any restore process is the data copy phase. The data copy phase initializes the contents of the database, files, or pages being restored.
2)Roll Forward (Redo): Redo (or roll forward) is the process of redoing logged changes to the data in the roll forward set to bring the data forward in time.
3)Recovery Point: The goal of roll forward is to return the data to its original state at the recovery point. The recovery point is the point to which the user specifies that the set of data be recovered.
4)Rollbackward (Undo): After the redo phase has rolled forward all the log transactions, a database typically contains changes made by transactions that are uncommitted at the recovery point.
The recovery process opens the transaction log to identify uncommitted transactions. Uncommitted transactions are undone by being rolled back. This step, is called the undo (or roll back) phase.
Types of Recovery:-
Restart Recovery: Everytime an instance is restarted/started the consistency of all the databases including master,model,msdb and tempdb is checked. This process is an internal operation and initiated just to keep the entire instance clean and with integrity.
Taking database offline/online involves restart recovery for that database.
Restore Recovery As per backup strategy whenever a restore is started and recovery is done per backup sequence. This entire process of recovery initiated manually is called as Restore recovery.
Third Party Backup Tools:
Redgate SQL Backup (Redgate)
Quest Litespeed
Veritas Net Backup
Tivoli Storage Manager (IBM)
Legato Networker (EMC)
Symantec BackupExec
Acroyns
Avamar (HP)
Hyperbac 10)SQL SafeLite (Idera) 11)Evault 12)Sonasafe for SQL Server (SonaSoft) 13)Backup and FTP
Compressed Backups: Backup compression was introduced in SQL Server 2008 Enterprise. SQL Server 2008 R2 it is possible in Standard and higher editions.
Restrictions:
Compressed and uncompressed backups cannot co-exist in a media set.
Previous versions of SQL Server cannot read compressed backups.
Before attempting to take a backup with compression, we need to enable backup compression option at instance level if needed.
sp_configure 'show advanced options',1 reconfigure with override sp_configure 'backup compression default',1 reconfigure with override sp_configure
To run backup command is as below
backup database GroupTest to disk=N'f:\GT.bak' with compression
By default, compression significantly increases CPU usage, and the additional CPU consumed by the compression process might adversely impact concurrent operations.
Note: Creating compressed backups is supported only in SQL Server 2008 Enterprise and later, but every edition of SQL Server 2008 and later can restore a compressed backup.
Calculate the Compression Ratio of a Compressed Backup:
To calculate the compression ratio of a backup, use the values for the backup in the backup_size and compressed_backup_size columns of the backupset history table, as follows:
backup_size:compressed_backup_size
For example, a 3:1 compression ratio indicates that you are saving about 66% on disk space.
SELECT backup_size/compressed_backup_size FROM msdb..backupset
Backup Metadata:- use msdb go select * from dbo.backupset select * from dbo.backupmediafamily where media_set_id=2327
Restore Metadata:- select * from dbo.restorehistory
To simplify the script to query backup information in SQL Server:-
SELECT CONVERT(CHAR(100), SERVERPROPERTY('Servername')) AS Server, msdb.dbo.backupset.database_name, msdb.dbo.backupset.backup_start_date, msdb.dbo.backupset.backup_finish_date, msdb.dbo.backupset.expiration_date, CASE msdb..backupset.type WHEN 'D' THEN 'Database' WHEN 'L' THEN 'Log' END AS backup_type, msdb.dbo.backupset.backup_size, msdb.dbo.backupmediafamily.logical_device_name, msdb.dbo.backupmediafamily.physical_device_name, msdb.dbo.backupset.name AS backupset_name, msdb.dbo.backupset.description FROM msdb.dbo.backupmediafamily INNER JOIN msdb.dbo.backupset ON msdb.dbo.backupmediafamily.media_set_id = msdb.dbo.backupset.media_set_id WHERE (CONVERT(datetime, msdb.dbo.backupset.backup_start_date, 102) >= GETDATE() - 7) ORDER BY msdb.dbo.backupset.database_name, msdb.dbo.backupset.backup_finish_date
Backup Internals:-
Backup command gets parsed. It validates Syntax and existence of Database.
Backup command is passed to Execution Area, important point to note Backup Buffers are created in MTL Portion (till SQL Server 2012).
MTL (Mem-To-Leave) is a portion in Memory where Large Pages are stored. Backup Buffers, Linked Servers data, Extended Stored Procedure and lots of other information external to SQL Server is stored.
To track the buffer size we need to enable trace 3605 and 3213.
Whenever a backup is initiated entire file structure of the database is copied to the BAK file.
As Backup related pages and information to be backed up are stored in MTL portion, hence all the content from MDF/NDF files is copied into BAK file through MTL portion.
BACKUP Command Additional Parameters:
NOINIT | INIT Controls whether the backup operation appends to or overwrites the existing backup sets on the backup media. NOINIT -> Append INIT -> Overwrite
Default option is NOINIT.
NOSKIP | SKIP Controls whether a backup operation checks the expiration date and time of the backup sets on the media before overwriting them. NOSKIP -> Will verify expiration of backups before overwriting backupsets. SKIP -> Will not verify expiration of backupsets
Default option is NOSKIP
NOFORMAT | FORMAT Specifies whether the media header should be written on the volumes used for this backup operation, overwriting any existing media header and backup sets.
NOFORMAT -> Backup preserves existing media set. FORMAT -> Backup creates a new media set.
Default option is NOFORMAT.
BLOCKSIZE: Specifies the physical block size, in bytes. The supported sizes are 512, 1024, 2048, 4096, 8192, 16384, 32768, and 65536 (64 KB) bytes. The default is 65536 for tape devices and 512 otherwise. Typically, this option is unnecessary because BACKUP automatically selects a block size that is appropriate to the device.
BUFFERCOUNT: Specifies the total number of I/O buffers to be used for the backup operation. You can specify any positive integer; however, large numbers of buffers might cause "out of memory" errors because of inadequate memory in the Sqlservr.exe process.
MAXTRANSFERSIZE: Specifies the largest unit of transfer in bytes to be used between SQL Server and the backup media. The possible values are multiples of 65536 bytes (64 KB) ranging up to 4194304 bytes (4 MB).
The total space used by the buffers is determined by: buffercount*maxtransfersize.
To analyze and understand backup parameters: Enable traceflag 3213 to get actual backup parameters SQL Server internally choses to complete backup operation.
DBCC TRACEON(3213,-1) DBCC TRACEON(3605,-1) --Writes to Error log DBCC TRACEON(3604,-1) --Writes backup information on screen
BACKUP DATABASE KDSSGIndexTest to disk=N'KDSSGIndexTest.bak' WITH BUFFERCOUNT=14,MAXTRANSFERSIZE=4194304
The content copied by the reader thread is written to MTL portion in form of Backup Buffers. The writer thread will write this to the BAK file either on tape or on disk.
If the database files are spread into multiple drives (example 3) then we will have 3 reader threads and if we are taking striped backup (example 2 files) then two writer threads for backup operation.
How to improve Backup Performance:
Striped Backup to seperate drives. When backup is performed on seperate drives dedicated WRITER thread will be allocated per volume.
BLOCKSIZE When block size is increased it MAY improve peformance.
BUFFERCOUNT and MAXTRANSFERSIZE This will decide the buffer size and buffer count and definitely improves performance.
The total space that will be used by the buffers is determined by: buffercount * maxtransfersize.
The output shows this is correct; with 49 buffers * 1024 KB = 49 MB total buffer space is in use.
Third Party Backup tools also will improve performance of backup.
VSS Writer: VSS is shadow copy (also known as a snapshot or a point-in-time copy) of the data we want to backup. We can then use those shadow copies as backup, without affecting the running application at that point.
There are four basic parts of a VSS solution that need to be in place for a complete system to work: the VSS coordination service, the VSS requester, the VSS writer and the VSS provider.How to improve Backup Performance:
Striped Backup to seperate drives. When backup is performed on seperate drives dedicated WRITER thread will be allocated per volume.
BLOCKSIZE When block size is increased it MAY improve peformance.
BUFFERCOUNT and MAXTRANSFERSIZE This will decide the buffer size and buffer count and definitely improves performance.
The total space that will be used by the buffers is determined by: buffercount * maxtransfersize.
The output shows this is correct; with 49 buffers * 1024 KB = 49 MB total buffer space is in use.
Third Party Backup tools also will improve performance of backup.
VSS Writer: VSS is shadow copy (also known as a snapshot or a point-in-time copy) of the data we want to backup. We can then use those shadow copies as backup, without affecting the running application at that point.
There are four basic parts of a VSS solution that need to be in place for a complete system to work: the VSS coordination service, the VSS requester, the VSS writer and the VSS provider.
Encryption: SQL Server 2005 and SQL Server 2008 provide Encryption as a new feature to protect data against hackers’ attacks.
Hackers might be able to penetrate the database or tables, but owing to encryption they would not be able to understand the data or make use of it.
Encryption hierarchy is marked by three-level security.
Windows Level – Highest Level – Uses Windows DP API for encryption
SQL Server Level - Moderate Level – Uses Services Master Key for encryption
Database Level – Lower Level – Uses Database Master Key for encryption
There are two kinds of keys used in encryption:
Symmetric Key – In Symmetric cryptography system, the sender and the receiver of a message share a single, common key that is used to encrypt and decrypt the message.
Asymmetric Key – Asymmetric cryptography, also known as Public-key cryptography, is a system in which the sender and the receiver of a message have a pair of cryptographic keys – a public key and a private key – to encrypt and decrypt the message.
Yet another way to encrypt data is through certificates.
Asymmetric and Symmetric Keys: Public Key Cryptography (PKI) is a form of message secrecy in which a user creates a public key and a private key. The private key is kept secret, whereas the public key can be distributed to others. Although the keys are mathematically related, the private key cannot be easily derived by using the public key. The public key is used to encrypt data and the private key is used to decrypt data. A message that is encrypted by using the public key can only be decrypted by using the correct private key. Since there are two different keys, these keys are asymmetric.
Certificates and asymmetric keys are both ways to use asymmetric encryption. Certificates are often used as containers for asymmetric keys because they can contain more information such as expiry dates and issuers. There is no difference between the two mechanisms for the cryptographic algorithm, and no difference in strength given the same key length. Generally, you use a certificate to encrypt other types of encryption keys in a database, or to sign code modules.
Certificates and asymmetric keys can decrypt data that the other encrypts.
Certificate: A certificate is a digitally signed security object that contains a public (and optionally a private) key for SQL Server. You can use externally generated certificates or SQL Server can generate certificates.
Certificates can be used to help secure connections, in database mirroring, to sign packages and other objects, or to encrypt data or connections.
Asymmetric Keys: Asymmetric keys are used for securing symmetric keys. They can also be used for limited data encryption and to digitally sign database objects. An asymmetric key consists of a private key and a corresponding public key.
Asymmetric keys can be imported from strong name key files, but they cannot be exported. They also do not have expiry options. Asymmetric keys cannot encrypt connections.
Encryption is the process of encoding messages or information. It converts plain text into Cipher text using some algorithm.
Types of Encryption: Private Key (Symmetric): Same key is used to encrypt and decrypt.
Public Key (ASymmetric): Encryption is performed by a public key which is accessible for everyone, but decryption is performed using a different key.

--Backup Encryption
CREATE DATABASE BKPENCRYPT
CREATE TABLE TAB1 (sno int, sname varchar(50))
INSERT INTO TAB1 VALUES (1,'Hello') INSERT INTO TAB1 VALUES (2,'KDSSG')
select * from sys.certificates
use master go BACKUP SERVICE MASTER KEY TO FILE = 'C:\OurBackups\SQL2K16_Service_Master_Key.key' ENCRYPTION BY PASSWORD = 'Admin143$$'; GO
use master go CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'Admin143$$'; GO
BACKUP MASTER KEY TO FILE = 'C:\OurBackups\SQL2016_Master_Key.key' ENCRYPTION BY PASSWORD = 'Admin143$$'; GO
Use Master go CREATE CERTIFICATE BKPEncrypt2K17_2 WITH SUBJECT = 'SQL Server 2017_2 Backup Encryption' GO
BACKUP CERTIFICATE BKPEncrypt2K17_2 TO FILE = 'C:\OurBackups\SQL2017_2_BKPEncrypt_certificate.cer' WITH PRIVATE KEY ( FILE = 'C:\OurBackups\SQL2017_2_BKPEncrypt_certificate_private_key.key', ENCRYPTION BY PASSWORD = 'Admin143$$' ) GO
BACKUP DATABASE BKPEncrypt TO DISK = 'C:\OurBackups\BKPEncrypt_Full_Encrypted.bak' WITH ENCRYPTION (ALGORITHM = AES_256, SERVER CERTIFICATE = BKPEncrypt2K17_2)
----------Restore in destination instance.
RESTORE HEADERONLY FROM DISK = 'C:\OurBackups\BKPEncrypt_FULL_ENCRYPTED.bak'
Use Master GO RESTORE MASTER KEY FROM FILE = N'C:\OurBackups\SQL2014_Master_Key.key' DECRYPTION BY PASSWORD = 'Admin143$$' ENCRYPTION BY PASSWORD = 'Admin143$$'
--We may get an error here, so make sure your service account has --permissions on the Backup location path.
OPEN MASTER KEY DECRYPTION BY PASSWORD='Admin143$$'
CREATE CERTIFICATE BKPEncrypt2K16 FROM FILE='C:\OurBackups\SQL2K16_BKPEncrypt_certificate.cer' with private key (file='C:\OurBackups\SQL2K16_BKPEncrypt_certificate_private_key.key', decryption by password='Admin143$$')
restore verifyonly from disk='C:\OurBackups\BKPEncrypt_Full_Encrypted.bak'
restore database BKPEncrypt from disk=N'C:\OurBackups\BKPEncrypt_Full_Encrypted.bak' with Move 'BKPEncrypt' to 'C:\Program Files\Microsoft SQL Server\MSSQL12.INST2K14_5\MSSQL\Data\BKPEncrypt.mdf', move 'BKPEncrypt_Log' to 'C:\Program Files\Microsoft SQL Server\MSSQL12.INST2K14_5\MSSQL\Data\BKPEncrypt.ldf' ,recovery
USE [master] GO CREATE CREDENTIAL [BackupCred] WITH IDENTITY = N'amar', SECRET=N'7okxqi88GiubUY3v44a+TcnHGge9XtZ9ZpVbmM6gyKhEeLjeVhqHyx9p8UIdfiStoXEMk3PdXWytQEN2vGTW6A=='
BACKUP DATABASE [SuspectDB] TO URL = N'https://kdssg.blob.core.windows.net/backupb22/SuspectDB_Full_2016_08_15_0810.bak' WITH CREDENTIAL = N'BackupCred', STATS = 10
Description: Storage Account: The storage account is the starting point for all storage services. To access the Microsoft Azure Blob storage service, first create a Windows Azure storage account.
Container: A container provides a grouping of a set of blobs, and can store an unlimited number of blobs. To write a SQL Server backup to the Microsoft Azure Blob storage service, you must have at least the root container created. You can generate a Shared Access Signature token on a container and grant access to objects on a specific container only.
Blob: A file of any type and size. There are two types of blobs that can be stored in the Microsoft Azure Blob storage service: block and page blobs.
kdssg
lUBTjxyvBGUVDEDYkODGQDFx/l8xuac5El0IoLRIaVVNl0Mn/i8zGArN3++hlcH7wUO/RwSRZ894wbL2VBeKIQ==
https://kdssg.blob.core.windows.net/backupb22
Smart Backups:- SQL Server Managed Backup to Windows Azure manages and automates SQL Server backups to the Windows Azure Blob storage service. SQL Server Managed Backup to Windows Azure supports point in time restore for the retention time period specified.
SQL Server Managed Backup to Windows Azure can be enabled at the database level or at the instance level to manage all the databases on the instance of SQL Server.
We can enable Smart backup using stored procedure.
smart_admin.sp_set_db_backup: System stored procedure for enabling and configuring SQL Server Managed Backup to Windows Azure for a database.
EXEC smart_admin.sp_set_db_backup @database_name = 'KDSSG', @retention_days = 10, @credential_name = 'Cred1', @encryption_algorithm = 'AES_128', @encryptor_type = 'Certificate', @Encryptor_name ='Backup_Certificate', @enable_backup = 1;
Smart Full Backup: SQL Server Managed Backup to Windows Azure agent schedules a full database backup if
A database is SQL Server Managed Backup to Windows Azure enabled for the first time
The log growth since last full database backup is equal to or larger than 1 GB.
The maximum time interval of one week has passed since the last full database backup
The log chain is broken.
Smart Log Backup: SQL Server Managed Backup to Windows Azure schedules a log backup if
There is no log backup history that can be found.
The transaction log space used is 5 MB or larger
The maximum time interval of 2 hours since the last log backup is reached.
Any time the transaction log backup is lagging behind a full database backup. The goal is to keep the log chain ahead of full backup.
Retention Period is 30 days for Smart Backups. (Min: 1 day and Max: 30 days)
1TB restriction for Database Backup on Cloud.
Refer: http://www.scarydba.com/2013/12/19/how-to-set-up-managed-backups-in-sql-server-2014/
BACKUP TO URL:-
CREATE CREDENTIAL MyCredentialName WITH IDENTITY = 'KDSSG', SECRET = 'bVIEBvX8HZPbYGGq75y7NSjVflpN6DgmWuIChGGCTK1KvAuXeQiVbLr7jUkhJy373DmAVAHz6/LeGnYwIQjCbg==';
BACKUP DATABASE Model TO URL = N'https://kdssg.blob.core.windows.net/Container1/Model_07022016.bak' WITH CREDENTIAL = N'MyCredentialName', STATS = 10;
Old Interface: https://manage.windowsazure.com
New Interface: https://portal.azure.com
Microsoft provides tools and utilities that will generate certificates and strong name key files. These tools offer a richer amount of flexibility in the key generation process than the SQL Server syntax.
makecert - Creates Certificates sn - Creates strong names for symmetric keys.
Try this demo only on SQL 2005/2008/2008R2:-
BACKUP DATABASE [master] TO DISK='C:\OurBackups\Master.bak' WITH PASSWORD='Admin143$$'
Beginning with SQL Server 2012, the PASSWORD and MEDIAPASSWORD options are discontinued for creating backups.
Pseudo Simple Recovery Model: It is a fact that when we switch a database into FULL recovery model, it actually behaves as if it is in the SIMPLE recovery model until the log backup chain is established (this is commonly called being in 'pseudo-SIMPLE') with a Full Backup.
When a database is created, SQL Server runs that database in a "pseudo simple recovery model" till a Full backup is initiated.
The general premise is that when you switch from Simple to Full recovery model, the model does not change until the log backup chain has been established, with a backup. This is called pseudo-simple mode, and can be recognised by a database that is in Full recovery model showing a NULL value for last_log_backup_lsn in sys.database_recovery_status
This is complicated by the fact that when you switch a database into the FULL recovery mode, it actually behaves as if it's in the SIMPLE recovery mode until the log backup chain is established (this is commonly called being in 'pseudo-SIMPLE').
When a database is created, SQL Server runs that database in a "pseudo simple recovery model". What that means is that SQL Server automatically clears inactive records from the transaction log once it knows that it no longer needs them. It no longer needs them to be stored in the log because no one is using the log. However, once you do start to do backups (and, people generally start by doing a full database backup), then SQL Server looks to your recovery model to determine what to do with log records. If the recovery model is set to full (and, yes, this is the default), then SQL Server gives you the "full feature set" with regard to backup/restore.
Corrutptions --------
MSDB Corrupt:
Verify the reason of failure in the error logs and troubleshoot accordingly. If database is really corrupt then look out for a available valid backup. If backup is available restore MSDB as a normal user database and it would be restored.
RESTORE DATABASE [msdb] FROM DISK='C:\OurBackups\MSDB.bak' WITH REPLACE
If backup is not available, then stop the instance and start the instance with /t3608 startup parameters.
net stop "SQL Server (INST2K12)" net start "SQL Server (INST2K12)" /m /t3608
Connect to the Query window and detach MSDB database and delete the OS level files.
sp_detach_db 'MSDB' --Remove MSDB data/log files.
Execute the script in %Root Directory%\Install\instMSDB.sql file.
This would recreate the entire structure of MSDB database and might give some errors/warnings.
Errors/Warnings:-
Msg 15281, Level 16, State 1, Procedure xp_cmdshell, Line 1 SQL Server blocked access to procedure 'sys.xp_cmdshell' of component 'xp_cmdshell' because this component is turned off as part of the security configuration for this server. A system administrator can enable the use of 'xp_cmdshell' by using sp_configure.
Msg 15129, Level 16, State 1, Procedure sp_configure, Line 150 '-1' is not a valid value for configuration option 'Agent XPs'.
Solution: sp_configure 'show advanced options',1 reconfigure with override
sp_configure 'xp_cmdshell',1
sp_configure 'Agent XPs',1 reconfigure with override
sp_configure
Master Corrupt: Master is the most crucial database in an instance, if it is corrupt entire instance gets affected.
If master database is corrupt, it is either completely corrupt or partially corrupt. If partially corrupt (only some pages are corrupt) instance will start with -m;-t3608 and if it is completely corrupt instance wouldn't start.
Completely Corrupt:
Method1:-
Master database doesn't start with /m /t3608 and hence we need to rebuild the master database.
Rebuild master SQL Server 2005: start /wait setup.exe /qb INSTANCENAME=MSSQLSERVER REINSTALL=SQL_Engine REBUILDDATABASE=1 SAPWD=Admin143$$
SQL Server 2008: setup.exe /QUIETSIMPLE /ACTION=REBUILDDATABASE /INSTANCENAME="MSSQLSERVER" /SQLSYSADMINACCOUNTS="KDSSG\KDSSGDBATEAM" /SAPWD="Admin143$$"
SQL Server 2008R2/2012/2014/2016/2017: setup.exe /QS /ACTION=REBUILDDATABASE /INSTANCENAME="INST2K17" /SQLSYSADMINACCOUNTS="KDP-PC\KDP" /SAPWD="Admin143$$" /IAcceptSQLServerLicenseTerms
Start instance with /m /t3608 net stop "SQL Server (MSSQLSERVER)" net start "SQL Server (MSSQLSERVER)" /m /t3608
Restore master database WITH REPLACE option restore database master from disk=N'F:\Master.bak' WITH REPLACE
Rebuilding master also rebuilds Model, MSDB. So restore model and msdb backups too.
Method2:- Restore master database using files in Binn\Templates directory. In this scenario no need to rebuild the instance and also Model and MSDB databases are left untouched.
As instance is corrupt, so stop SQL Server instance. net stop "SQL Server (MSSQLSERVER)"
Copy the files in Binn\Templates in instance root directory to DATA directory.
Now start the instance with /m and /t3608 net start "SQL Server (MSSQLSERVER)" /m /t3608
Now instance starts but master has incorrect path references to all the other databases. So a restore would not work. So first we will have to check and change all path references using below commands.
select * from sys.sysdatabases select * from sys.sysaltfiles
Changing incorrect path references using below commands.
alter database [mssqlsystemresource] modify file (name='Data',filename='C:\Program Files\Microsoft SQL Server\MSSQL13.INST2K16\MSSQL\Binn\mssqlsystemresource.mdf')
alter database [mssqlsystemresource] modify file (name='Log',filename='C:\Program Files\Microsoft SQL Server\MSSQL13.INST2K16\MSSQL\Binn\mssqlsystemresource.ldf')
alter database [model] modify file (name='modeldev',filename='C:\Program Files\Microsoft SQL Server\MSSQL13.INST2K16\MSSQL\DATA\model.mdf')
alter database [model] modify file (name='modellog',filename='C:\Program Files\Microsoft SQL Server\MSSQL13.INST2K16\MSSQL\DATA\modellog.ldf')
alter database [msdb] modify file (name='MSDBData',filename='C:\Program Files\Microsoft SQL Server\MSSQL13.INST2K16\MSSQL\DATA\MSDBData.mdf')
alter database [msdb] modify file (name='MSDBLog',filename='C:\Program Files\Microsoft SQL Server\MSSQL13.INST2K16\MSSQL\DATA\MSDBLog.ldf')
alter database [tempdb] modify file (name='tempdev',filename='C:\Program Files\Microsoft SQL Server\MSSQL13.INST2K16\MSSQL\DATA\tempdb.mdf')
alter database [tempdb] modify file (name='templog',filename='C:\Program Files\Microsoft SQL Server\MSSQL13.INST2K16\MSSQL\DATA\templog.ldf')
--Create login as its a new master and there will be no logins in the instance.
CREATE LOGIN [KDSSG\KDSSGDBATeam] FROM WINDOWS sp_addsrvrolemember 'KDSSG\KDSSGDBATeam','sysadmin'
After all references are changes, restart the instance once with /m /t3608 for all changes to take affect. net stop "SQL Server (MSSQLSERVER)" net start "SQL Server (MSSQLSERVER)" /m /t3608
Now restore the backup of master. restore database [master] from disk=N'Master.bak' with replace.
Method3:- Resolving Master database corruption through Restoring it as a user database in another instance.
Restore master database as a user database in another instance. Restore database [master_copy] from disk=N'C:\Media\Master_Full.bak' WITH MOVE 'Master' to 'C:\Media\Master_Copy.mdf', MOVE 'Master_log' to 'C:\Media\Master_Copy.ldf'
After restoring master database a user database. Detach the database. sp_detach_db 'Master_Copy'
Rename the files C:\Media\Master_Copy.mdf and C:\Media\Master_Copy.ldf as master database files(master.mdf and master_log.ldf) and replace them with existing Master database in Data Directory.
Stop the instance and start the instance normally.
Partially Corrupt:- Partial corruption will allow to start the instance. Restore the master database to get back to the original state before corruption.
Master Corruption without BACKUP:- If no backup of master database is available, then we will have to
Attach all the user database MDF files to the newly created master
Recreate all the logins by contacting Application team for credentials and also permissions
Proper user mappings have to be done after confirming from application team.
All instance settings needs to be verified if they are in sync with the environment.
Recreate if any extended stored procedures are required.
Recreate linked server, endpoints, certificates if any.
Note: In SQL Server 2008/2008 R2/2012/2014 when master rebuilt is performed all system databases like Master, Model, MSDB, Tempdb are rebuilt. There is no choice to rebuild a specific system database.
In SQL Server 2005/2000 when master rebuilt is performed only MASTER database is rebuilt.
In SQL Server 2000 master rebuilt is possible through a tool called rebuildm.exe
Model Database Corruption: Model database being one of the crucial database for new database creations and also for Tempdb recreation on every restart.
If model database is corrupt it is going to affect instance functionality.
Steps:
Verify if Model is corrupt or not in Eventviewer and SQL Server Error Logs.
Confirm if a valid database backup exists or not using restore verifyonly/headeronly.
As instance isn't starting, rebuilding entire instance is correct answer but may not be a correct approach just for the sake of model database. Copy only the BINN\TEMPLATE model database files to DATA folder.
This would start the instance. Once instance starts restore Model just as a user database.
Restore the Model database from backup.
restore database model from disk=N'F:\Model.bak' WITH REPLACE
Resource Database Corruption: Resource database is a binary and a system database and its corruption would not allow instance to start.
If resource database is corrupt it is going to affect instance functionality.
Steps:
Verify if RDB is corrupt or not, in Eventviewer and SQL Server Error Logs.
Confirm if a valid database backup exists. RDB backups are more of OS level file copy backups.
As instance isn't starting, if backup is available copy and paste the Resource database files into BINN directory from the backup.
Start the instance.
If RDB backup is not available, we will have to do REPAIR installation of SQL Server.
a) REPAIR will perform entire repair of instance and also Master, Model,MSDB and TEMPDB are rebuilt.
Approach1: Appwiz.cpl -> SQL Server 2012 (64Bit) -> Right Click -> Uninstall/Change -> Repair -> It would ask to map to Media directory -> Perform Repair installation
Approach2: Open Setup.exe from Media -> Maintenance -> Repair.
Using both approach is similar and it would perform rebuild of resource and system databases.
After rebuilding Resource database, restore Master, Model and MSDB backups (if they are available).
b) Alternate option is to COPY RDB files from another instance and overwrite existing RDB files in the instance. This should meet one criteria i.e. the files being copied should belong to SAME VERSION and BUILD.

Resource Database corruption: If resource database gets corrupted, its more of a binary corruption. Hence a copy can be initiated preferably from same build, if not from a different build.
If resource is corrupt, try to start instance with /t3608 and /m to confirm if atleast instance starts with Master only mode.
Review the event viewer. Ideally if resource corruption identified, copy the files from colocated instances.

Resource Corruption: If resource database is corrupted, rebuilding master will rebuild the resource database.
Page Corruption: Page corruption is individual pages getting corrupted in a Data file. Restoring entire database backup for a single page corruption is not feasible, hence SQL Server provides an option to restore single or multiple pages when corrupted from existing backups.
Steps:
Create a database and a table within it. Database recovery model should be FULL.
Take a full backup of the database and any additional log backups if needed.
Find the page number of the table which stores the data using below commands
Use CorruptMe go Select * from sys.indexes where OBJECT_NAME(object_id)='Tab'
Use CorruptMe go Select * from sys.indexes where OBJECT_NAME(object_id)='Tab'
DBCC IND ('CorruptMe','Tab',0)
DBCC TRACEON (3604,-1); GO DBCC PAGE('CorruptMe',1,153,3);

DBCC TRACEON(3604,-1)
Switches on the specified trace flags globally.
dbcc page ( {'dbname' | dbid}, filenum, pagenum [, printopt={0|1|2|3} ])
DBCC PAGE ('PageTest',1,153,3) 1 = FileNum (MDF) 153 = Page Number 3 = Printoptions
Printoptions: 0-print just the page header 1-page header plus per-row hex dumps and a dump of the page slot array 2- page header plus whole page hex dump 3- page header plus detailed per-row interpretation

Once the page number is identified, multiply the page number with 8192 to get the offset value.
Make modifications to the page in the Hex Editor tool.
Change the database state to Offline before editing in Hex Editor Tool.
Ctrl+G should be used to find the hexadecimal value for the Offset value. (Address->Go To)
Replace the text in that offset rather than deleting the text. Replace will create Checksum
Bring database to Online state.
Query the table which would clearly show corruption in the specific page/object/database/fileid.
For detailed information DBCC CHECKDB can be run. Take a log backup of the database WITH NORECOVERY.
backup log CorruptMe to disk=N'E:\CorruptMelastlog.trn' with norecovery
Perform the restore of the page using restore command as below (Full and if any Tlogs)
Restore DATABASE CorruptMe Page='1:153' FROM DISK='E:\CorruptMe_FullBackup.bak' with norecovery
restore log CorruptMe from disk='E:\CorruptMelastlog.trn' with recovery
If multiple pages are corrupted i.e. > feasible number like 30 or 40. It is better to restore entire database than single page level restores.
Suspect State:
Suspect is a state where database becomes inaccessible due to different reasons
Data and Log files missing
Corruption of pages in the Data and Log files.
Synchronization issues between data and log files
Suspect Flag is enabled
File level permissions on data and log files
Issues that are caused during Recovery/Restoring process
NOTE: NEVER DETACH A DATABASE WHICH IS IN SUSPECT STATE TILL EXACT REASON IS KNOWN.
Steps to Resolve:
Identify if database is really in suspect state or not. select databasepropertyex('SuspectDB','status')
Attempt to reset the suspect flag using sp_resetstatus sp_resetstatus 'SuspectDB'
Set the EMERGENCY mode "ON" for the database for further troubleshooting. Emergency mode is a READ_ONLY state and gives some base for identifying the cause of the issue.
ALTER DATABASE [SuspectDB] SET EMERGENCY,SINGLE_USER
As backups are not possible in Emergency state, atleast Export/Import can be used to capture the important object(s) if backup is not available.
Put database in Single User mode, to avoid connection conflicts.
Run DBCC CHECKDB on the database to identify if the issue is with Data files or Log files.
Running checkdb finds any consistency and allocation errors and if there are no errors found then Data file is considered to be clean. The issue might exist with Log file.
Output should say: CHECKDB found 0 allocation errors and 0 consistency errors in database 'test'.
If backup is not available and issues found with data file/log file use
DBCC CheckDB ('SuspectDB', REPAIR_ALLOW_DATA_LOSS)
This command repairs the database but has the risk of data loss. Take proper approvals before performing this step.
If checkdb finds issues in the MDF/NDF files, then before restoring from the good available backup attempt Detach/Attach.
If data files are clean and consistent, the issue could be mostly with Log file. If Log file corrupt:
Method1 Without detaching the database we can rebuild the log file. First rename/delete the old log file and create a new one using command below.
ALTER DATABASE SuspectDB REBUILD LOG ON (NAME=SuspectDB_log,FILENAME='C:\Program Files\Microsoft SQL Server\MSSQL11.MSSQLSERVER\MSSQL\SuspectDB_log.LDF')
Alter database SuspectTest set multi_user
Method2 (Method works only when you have single log file)
Put database into OFFLINE state and delete existing log file and bring it back to ONLINE state.
OFFLINE to ONLINE rebuild process would not work if there are multiple log files.
Method3 DBCC CHECKDB('SuspectDB',REPAIR_ALLOW_DATA_LOSS)
Risk involved with this method is all log files will be removed and one single log file is created during rebuild process.
Method4 (Method works even if you have multiple log files)
CREATE DATABASE [SuspectDB] ON (FILENAME = 'C:\Program Files\Microsoft SQL Server\MSSQL11.B182K12A\MSSQL\DATA\SuspectDB.mdf') FOR ATTACH_REBUILD_LOG
Method5 (Method works only when we have single log file) USE master; GO
EXEC sp_detach_db @dbname = 'DBSRC2';
EXEC sp_attach_single_file_db @dbname = 'DBSRC2', @physname = N'C:\Program Files\Microsoft SQL Server\MSSQL10_50.MSSQLSERVER\MSSQL\Data\DBSRC2_Data.mdf';
CREATE DATABASE PageTest ON PRIMARY (FILENAME = 'F:\Program Files\Microsoft SQL Server\MSSQL.1\MSSQL\Data\PageTest.mdf') FOR ATTACH GO
This method works only when there is single log file
Method6 Perform Detach/Attach through GUI.
databasepropertyex
sp_resetstatus
EMERGENCY Mode and Single User Mode
DBCC CHECKDB
If corruptions found, check Backup is available or not. If available take approval and restore. If not available attempt to perform Import/Export. But in some cases DB sizes are very huge, then upon approval attempt DBCC CHECKDB ('Suspect',REPAIR_ALLOW_DATA_LOSS) If no positive results, then after Import/Export is completed attempt a detach/attach.
If Log file is corrupt, perform all the steps.
If database has single log file and if log file is corrupt during a suspect database state:


DBCC CHECKDB ('PageTest',REPAIR_ALLOW_DATA_LOSS)
Risk involved with this method is all log files will be removed and one single log file is created during rebuild process.
sp_detach_db PageTest
CREATE DATABASE PageTest ON PRIMARY (FILENAME = 'F:\Program Files\Microsoft SQL Server\MSSQL.1\MSSQL\Data\PageTest.mdf') FOR ATTACH GO
This method works only when there is single log file
sp_detach_db PageTest
sp_attach_single_file_db 'PageTest', 'F:\Program Files\Microsoft SQL Server\MSSQL.1\MSSQL\Data\PageTest.mdf'
This method also works only when there is single log file in a database.
If database has multiple log file and if one log file is corrupt during a suspect database state, it can be rebuilt using below command:
Don't detach the database before executing this command
ALTER DATABASE SuspectTest REBUILD LOG ON (NAME=SuspectTest_Log1,FILENAME='E:\Program Files\Microsoft SQL Server\SuspectTest_Log1.ldf')
Alter database suspectTest set multi_user

CREATE DATABASE PageTest ON (FILENAME = 'F:\Program Files\Microsoft SQL Server\MSSQL.1\MSSQL\Data\PageTest.mdf') FOR ATTACH_REBUILD_LOG

If database has single log file and if log file is corrupt during a suspect database state:


DBCC CHECKDB ('PageTest',REPAIR_ALLOW_DATA_LOSS)
Risk involved with this method is all log files will be removed and one single log file is created during rebuild process.
sp_detach_db PageTest
CREATE DATABASE PageTest ON PRIMARY (FILENAME = 'F:\Program Files\Microsoft SQL Server\MSSQL.1\MSSQL\Data\PageTest.mdf') FOR ATTACH GO
This method works only when there is single log file
sp_detach_db PageTest
sp_attach_single_file_db 'PageTest', 'F:\Program Files\Microsoft SQL Server\MSSQL.1\MSSQL\Data\PageTest.mdf'
This method also works only when there is single log file in a database.
If database has multiple log file and if one log file is corrupt during a suspect database state, it can be rebuilt using below command:
Don't detach the database before executing this command
ALTER DATABASE SuspectTest REBUILD LOG ON (NAME=SuspectTest_Log1,FILENAME='E:\Program Files\Microsoft SQL Server\SuspectTest_Log1.ldf')
Alter database suspectTest set multi_user

CREATE DATABASE PageTest ON (FILENAME = 'F:\Program Files\Microsoft SQL Server\MSSQL.1\MSSQL\Data\PageTest.mdf') FOR ATTACH_REBUILD_LOG

Moving database from one location to another:
Offline/Online Method:-
Execute alter database commands to modify the new paths in metadata.
alter database DBname modify file (name='logicalname',filename='physical path')
alter database AdventureWorks modify file (name='AdventureWorks_Data',filename='F:\Program Files\Microsoft SQL Server\MSSQL.1\MSSQL\AdventureWorks_Data.mdf')
alter database AdventureWorks modify file (name='AdventureWorks_Log',filename='F:\Program Files\Microsoft SQL Server\MSSQL.1\MSSQL\AdventureWorks_Log.ldf')
Take the database offline and move the files physically.
Bring the database online.
Note: Ensure SQL Server service account has proper permissions on the new location folder.
/* select 'alter database '+db_name(dbid)+' modify file(name='+''''+name+''''+',filename='+''''+filename+''''+')' from sys.sysaltfiles */
--select * from sys.sysaltfiles
alter database master modify file(name='master',filename='C:\Media\master.mdf') alter database master modify file(name='mastlog',filename='C:\Media\mastlog.ldf') alter database tempdb modify file(name='tempdev',filename='C:\Media\tempdb.mdf') alter database tempdb modify file(name='templog',filename='C:\Media\templog.ldf') alter database model modify file(name='modeldev',filename='C:\Media\model.mdf') alter database model modify file(name='modellog',filename='C:\Media\modellog.ldf') alter database msdb modify file(name='MSDBData',filename='C:\Media\MSDBData.mdf') alter database msdb modify file(name='MSDBLog',filename='C:\Media\MSDBLog.ldf') alter database UDB modify file(name='UDB',filename='C:\Media\UDB.mdf') alter database UDB modify file(name='UDB_log',filename='C:\Media\UDB_log.ldf') alter database ShreyaDB modify file(name='ShreyaDB',filename='C:\Media\ShreyaDB.mdf') alter database ShreyaDB modify file(name='ShreyaDB_log',filename='C:\Media\ShreyaDB_log.ldf')
Method2:- Detach/Attach Method
Detach the Database
Move the files physically
Attach the database (MDF file to be attached and modify the file names of NDF/LDF files)
Difference between Offline-Online/Detach-Attach method:
If database is offline, the database link to the instance is still available. For detach, the database is disconnected from the instance.
Offline/Online method can be used in the same instance but not for instance in another machine. If moving database files between different instances we prefer either Backup/Restore method or Detach/Attach method.
For a replicated database to be detached, it must be unpublished.
When a database is detached, all its metadata is dropped. If the database was the default database of any login accounts, master becomes their default database. Also cross database ownership chaining is broken.
Before detaching a database, you must drop all of its snapshots.
If database is being mirrored it cannot be detached until the database mirroring session is terminated.
Additional Methods: 3) Copy Database Wizard. Perform Copy Database Wizard to move database content from one database to another. Copy Database Wizard is nothing but an SSIS Package in the background.
Shrink File Method.
USE [master] GO CREATE DATABASE [TestDB] CONTAINMENT = NONE ON PRIMARY ( NAME = N'TestDB', FILENAME = N'E:\MSSQL\TestDB.mdf' , SIZE = 4096KB , MAXSIZE = UNLIMITED, FILEGROWTH = 1024KB ), ( NAME = N'TestDB_1', FILENAME = N'E:\MSSQL\TestDB_1.ndf' , SIZE = 4096KB , MAXSIZE = UNLIMITED, FILEGROWTH = 1024KB ), ( NAME = N'TestDB_2', FILENAME = N'E:\MSSQL\TestDB_2.ndf' , SIZE = 4096KB , MAXSIZE = UNLIMITED, FILEGROWTH = 1024KB ) LOG ON ( NAME = N'TestDB_log', FILENAME = N'E:\MSSQL\TestDB.ldf' , SIZE = 4096KB , MAXSIZE = 2048GB , FILEGROWTH = 1024KB) GO ALTER DATABASE [TestDB] SET RECOVERY SIMPLE
USE TestDB GO IF OBJECT_ID('dbo.SampleTable', 'U') IS NOT NULL DROP TABLE dbo.SampleTable GO CREATE TABLE dbo.SampleTable ( ID INT IDENTITY(1, 1) , Data DATETIME PRIMARY KEY CLUSTERED ( ID ) ) GO
USE TestDB GO INSERT INTO dbo.SampleTable ( Data ) VALUES ( GETDATE() ) GO 1000000
USE [master] GO ALTER DATABASE [TestDB] ADD FILE ( NAME = N'TestDB_1_New', FILENAME = N'G:\MSSQL\TestDB_1_New.ndf' , SIZE = 4096KB , FILEGROWTH = 1024KB ) TO FILEGROUP [PRIMARY] GO ALTER DATABASE [TestDB] ADD FILE ( NAME = N'TestDB_2_New', FILENAME = N'G:\MSSQL\TestDB_2_New.ndf' , SIZE = 4096KB , FILEGROWTH = 1024KB ) TO FILEGROUP [PRIMARY] GO
USE [master] GO ALTER DATABASE [TestDB] MODIFY FILE ( NAME = N'TestDB', FILEGROWTH = 0) GO ALTER DATABASE [TestDB] MODIFY FILE ( NAME = N'TestDB_1', FILEGROWTH = 0) GO ALTER DATABASE [TestDB] MODIFY FILE ( NAME = N'TestDB_2', FILEGROWTH = 0) GO
USE TestDB GO DBCC SHRINKFILE('TestDB_1', EMPTYFILE) GO DBCC SHRINKFILE('TestDB_1', TRUNCATEONLY) GO DBCC SHRINKFILE('TestDB_2', EMPTYFILE) GO DBCC SHRINKFILE('TestDB_2', TRUNCATEONLY) GO DBCC SHRINKFILE('TestDB', EMPTYFILE) GO DBCC SHRINKFILE('TestDB', TRUNCATEONLY) GO CHECKPOINT DBCC SHRINKFILE('TestDB_log', TRUNCATEONLY) GO
Tempdb Corruption: Tempdb corruption can occur and when tempdb corruption occurs, the best remedy is to restart SQL Server instance. Restarting SQL Server would recreate the new LDF files.
MSDB Corruption: MSDB corruption can lead to situation where all the jobs, alerts, backups/restore history is lost. MSDB loss effects SQL Server agent directly, so this would lead to two situations i.e. Backup available and Backup not Available.
Backup Available: If backup is available perform a direct restore of MSDB while instance is running. MSDB is the only system database that can be restored when instance is up and running.
Backup Not Available:
Method1: If backup is not available then MSDB requires a rebuild/recreation. MSDB can also be restored/attached from another instance but not a recommended method. Steps to be performed when MSDB corrupted without backup are a) Stop and Start the instance with -t3608 net start "Instance Name" /m /t3608 (or) Make changes to Startup Parameters in Configuration Manager
b) Detach MSDB database sp_detach_db MSDB
c) Remove the old corrupted physical data/log files of MSDB
d) Execute the script \Install\instmsdb.sql using SQLCMD or opening the script in SSMS
sqlcmd -S "ComputerName\InstanceName" -E -i "C:\Program Files\Microsoft SQL Server\MSSQL.1\MSSQL\Install\instmsdb.sql" -o "c:\Output.txt"
e) New MSDB would be created with all the internal objects like System Tables, Views and all other objects. Ignore any errors that are related to XP_CMDSHELL as it is built in the system and script that this option is disabled. Script will take care of the issue.
f) Once script execution is complete, stop and start the instance normally without the /m and /t3608
g) New MSDB gets created and if required perform additional few things sp_configure 'Agent XPs',1 go reconfigure with override go
sp_configure 'Show Advanced Options',1 reconfigure with override go sp_configure 'XP CMDSHELL',1 reconfigure with override go
h) Start the instance and verify if it starts normally
Method2: a) Stop the instance net stop MSSQL$INST2K12
b) Delete MSDB Data and Log files from DATA folder.
c) Copy the new MSDBdata.mdf and MSDBlog.ldf files from Binn\Templates folder to DATA directory.
d) Start SQL Server instance. net start MSSQL$INST2K12
Model Corruption: Model database gets corrupted is a threat to the instance, because if model is corrupt it wouldn't allow the tempdb to recreated while starting the instance. If model is corrupt then it would lead to two scenarios WITH Backup or WITHOUT backup.
If there are backups available then we can follow below process and perform restore of the backup.
a) Start the instance with -t3608, and dont start with -m as Only master can be restored in single user mode and not other system databases. b) net start "Instance Service Name" /t3608 This starts instance in Master only mode and which would allow us to perform Model database backup restore.
c) Restore the model database from the backup available. restore database Model from disk=N'Path\Model.bak' WITH REPLACE
d) Once model has been restored, start the instance normally without any parameters and the issue would be fixed.
e) If backup is not available then the issue leads to entire Instance Master database rebuild which would create a new model. This process is explained in Master rebuild situation.
Master Corruption: Master database corruption causes entire instance damage. There are two ways if master is completely corrupt or partially corrupt. If it is partially corrupt then instance would start with /m /t3608 traces and that would allow us to perform restore if backup is available.
First identify if instance is starting or not, if instance is starting then we can perform master database backup restore as below.
a) Start the instance with -t3608 and -m , master can be restored only with -m option mentioned. b) net start "Instance Service Name" /t3608 /m This starts instance in Master only mode, this allow to perform master database restore c) Restore the master database from the backup available. restore database Master from disk=N'Path\Master.bak' WITH REPLACE d) Once master has been restored, start the instance normally without any parameters and the issue would be fixed.
Second if instance is not starting partially then the only choice is to perform a rebuild of entire master database which is equal to entire instance rebuild (uninstall/reinstall).
a) Rebuilding master database can be done using rebuildm.exe in SQL Server 2000 and browse Master.mdf file which would be rebuilt. But this process has changed in SQL Server 2005 onwards and below command has to be fired. b) Copy the SQL 2005 setup files from the CD to the hard disk. In the command prompt, navigate to the folder which contains the setup.exe file.
start /wait setup.exe /qn INSTANCENAME="MSSQLSERVER" REINSTALL=SQL_Engine REBUILDDATABASE=1 SAPWD="NewP@ssword"
Sometimes it might through error about installation of Database Services failing. It could be because when RTM was installed it was done from some other path and when again instance is being rebuilt is doing from another path. So get the path changed with below command first before running above command.
a)start /wait setup.exe /qn INSTANCENAME="MSSQLSERVER" REINSTALL=SQL_Engine REBUILDDATABASE=1 SAPWD="NewP@ssword" REINSTALLMODE=v
This would set the path to the new location and next time the installation would continue without any errors.
b) After executing the above query the rebuild process would take sometime and new master data/log files would be created.
c) Start the instance and verify it is working fine or not. Once the instance starts. If master & Model & MSDB backups are available then perform all the restores one by one and finally start the instance.
net start "Instance Service Name" /m /t3608 restore database master from disk='Path\Master.bak' with replace net stop "Instance Service Name" net start "Instance Service Name" /t3608 restore database model from disk='Path\Model.bak' with replace restore database msdb from disk='Path\Msdb.bak' with replace
d) Verify all user databases are accessible or not through Management Studio and also through query. select * from sys.databases where state=Online.
This recreates the entire master database if it is completely corrupt.

<details><summary> pdf</summary>
  [SQL-Server-Backup-Recovery-and-Corruption-An-Advanced-Guide (1).pdf](https://github.com/user-attachments/files/22196866/SQL-Server-Backup-Recovery-and-Corruption-An-Advanced-Guide.1.pdf)

    </details>  


</details>



