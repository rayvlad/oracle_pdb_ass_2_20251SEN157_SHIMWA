# oracle_pdb_ass_2_20251SEN157_SHIMWA
# Oracle Pluggable Database (PDB) Assignment

## Overview of Tasks

This project demonstrates core Oracle Multitenant Architecture operations using Oracle Database XE 21c, including:

1. **Task 1 – PDB Creation**: Creating a new pluggable database (PDB) inside the container database (CDB), including an admin user local to that PDB.
2. **Task 2 – PDB Verification**: Opening the PDB, saving its state so it persists across restarts, and confirming the user was created successfully inside it.
3. **Task 3 – Oracle Enterprise Manager (OEM) Verification**: Accessing Oracle EM Database Express (EM Express) to confirm the environment and PDB reflect the completed work, with the logged-in username visible.
4. **Cleanup Task – Temporary PDB**: Creating a temporary/test PDB and then fully dropping it (including its datafiles) to demonstrate PDB lifecycle management (create → verify → delete).

## Oracle Environment Used

| Item | Detail |
|---|---|
| Database Edition | Oracle Database XE (Express Edition) |
| Version | 21.3.0.0.0 |
| Platform | Microsoft Windows x86 64-bit |
| CDB Name | XE |
| Management Tool | Oracle Enterprise Manager Database Express (EM Express) |
| Connection Method | SQL*Plus (`sqlplus / as sysdba`) |


### Task 1: Creating the Pluggable Database
Connected to the CDB (`XE`) as `SYSDBA` via SQL*Plus and created a new PDB named `SH_PDB_2021SEN157`, cloned from the seed database (`PDB$SEED`), with a local admin user `SHIMWA_PLSQLAUCA_20251SEN157`. Since `DB_CREATE_FILE_DEST` was not preconfigured, the `FILE_NAME_CONVERT` clause was used to map the seed database's datafile location to the new PDB's datafile location.

```sql
CREATE PLUGGABLE DATABASE SH_PDB_2021SEN157
  ADMIN USER SHIMWA_PLSQLAUCA_20251SEN157 IDENTIFIED BY <password>
  FILE_NAME_CONVERT = ('C:\APP\USER\PRODUCT\21C\ORADATA\XE\PDBSEED\',
                        'C:\APP\USER\PRODUCT\21C\ORADATA\XE\SH_PDB_2021SEN157\');
```

### Task 2: Opening the PDB and Verifying the User
The newly created PDB was opened in read-write mode and its state was saved so it reopens automatically after a CDB restart. The session was then switched into the PDB's container to confirm the admin user existed locally within it.

```sql
ALTER PLUGGABLE DATABASE SH_PDB_2021SEN157 OPEN;
ALTER PLUGGABLE DATABASE SH_PDB_2021SEN157 SAVE STATE;

ALTER SESSION SET CONTAINER = SH_PDB_2021SEN157;
SELECT username, account_status, common
FROM dba_users
WHERE username = 'SHIMWA_PLSQLAUCA_20251SEN157';
```

Output confirmed the user existed with `COMMON = NO`, verifying it is a local (PDB-only) user rather than a common user shared across the CDB.

### Task 3: Oracle Enterprise Manager (OEM) Verification
EM Express was accessed via `https://localhost:<port>/em`, logged in as `SYS` (SYSDBA). The Database Home dashboard confirmed:
- The CDB (`XE`, version 21.3.0.0.0) and its "CDB (2 PDB(s))" status
- The newly created PDB (`SH_PDB_2021SEN157`) listed alongside the default `XEPDB1` in the Data Storage chart
- The logged-in username (`SYS`) visible on the dashboard

### Cleanup Task: Temporary PDB Lifecycle
A second, temporary PDB (`SH_to_delete_20251SEN157`) was created using the same method to demonstrate the full PDB lifecycle, then removed completely:

```sql
ALTER SESSION SET CONTAINER = CDB$ROOT;
ALTER PLUGGABLE DATABASE SH_to_delete_20251SEN157 CLOSE IMMEDIATE;
DROP PLUGGABLE DATABASE SH_to_delete_20251SEN157 INCLUDING DATAFILES;
```

`INCLUDING DATAFILES` ensured the associated `.dbf` files were removed from disk, not just the PDB's metadata entry.

## Challenges Faced and How They Were Solved

| Challenge | Cause | Resolution |
|---|---|---|
| `ORA-...: FILE_NAME_CONVERT must be specified` | `DB_CREATE_FILE_DEST` was not set on the CDB, so Oracle had no default location for the new PDB's datafiles | Queried `v$datafile` joined against `PDB$SEED`'s container to find the real seed datafile path, then supplied it explicitly via `FILE_NAME_CONVERT` |
| "Missing or invalid file name pattern" on `FILE_NAME_CONVERT` | The path used did not match the actual filesystem path/format (Linux-style forward slashes and lowercase were used instead of the actual Windows path) | Re-ran the `v$datafile` query, copied the exact Windows path (`C:\APP\USER\PRODUCT\21C\ORADATA\XE\PDBSEED\`) character-for-character into the command |
| "Operation not allowed from within a pluggable database" on `CREATE PLUGGABLE DATABASE` | The session's container was still set to a PDB (from a prior `ALTER SESSION SET CONTAINER` command) instead of the root | Ran `ALTER SESSION SET CONTAINER = CDB$ROOT;` before re-attempting the create command, since PDB creation/drop must be run from the root container |
| PDB name exceeded Oracle's 30-character identifier limit | Initial temporary PDB name (`SHSHIMWA_to_delete_pdb_20251SEN157`) was too long | Shortened the name to `SH_to_delete_20251SEN157` (within the 30-character limit) |

## Integrity Statement

I confirm that the work described in this README and the accompanying screenshots reflects tasks I personally performed on my own Oracle Database XE 21c environment. All commands were executed directly in SQL*Plus and Oracle EM Express, and all evidence provided is authentic and unaltered.

## Submission Details

- **Repository Link:** [https://github.com/rayvlad/oracle_pdb_ass_2_20251SEN157_SHIMWA/edit/main/README.md]
- **PDB Name Created:** SH_PDB_2021SEN157
- **Issues Encountered:** Yes

