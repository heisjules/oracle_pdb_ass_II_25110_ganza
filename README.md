 # **README – Oracle  Oracle CDB & Pluggable Database Configuration (CDB/PDB Setup)**

**Author:** GANZA Jules
**Student ID:** 25110
**Project:** Oracle CDB & Pluggable Database Configuration

---

## **📌 Overview**

This project documents the creation and configuration of an Oracle **Container Database (CDB)** and a dedicated **Pluggable Database (PDB)** for student schema development and PL/SQL experimentation.

You created:

* A CDB root: **CDB$ROOT**
* A new pluggable database: **GA_PDB_25110**
* A local PDB admin user: **GANZA_PLSQLAUCA_25110**

This README summarizes all SQL steps, configuration decisions, and verification commands.

---

## **🔧 1. Creating the Pluggable Database**

```sql
CREATE PLUGGABLE DATABASE ga_pdb_25110
ADMIN USER ganza_plsqlauca_25110 IDENTIFIED BY Ganza123;
```

### ✔ Result

* PDB successfully created
* Local admin user created inside the PDB automatically (no need to create again in CDB)

---

## **🔌 2. Opening the Pluggable Database**

Run in **CDB$ROOT**:

```sql
ALTER PLUGGABLE DATABASE ga_pdb_25110 OPEN;
```

(Optional: open automatically on startup)

```sql
ALTER PLUGGABLE DATABASE ga_pdb_25110 SAVE STATE;
```

---

## **👤 3. Granting Roles to the PDB Admin User**

```sql
GRANT CONNECT, RESOURCE, CREATE SESSION TO ganza_plsqlauca_25110;
```

✔ Successful

---

## **📂 4. Tablespace Configuration**

Attempt to set quota on USERS failed:

```
ORA-00959: tablespace 'USERS' does not exist
```

➡ Because the USERS tablespace exists only in the CDB, NOT in the PDB.

### ✔ Correct Fix (run inside the PDB):

1️⃣ Switch to PDB

```sql
ALTER SESSION SET CONTAINER = ga_pdb_25110;
```

2️⃣ Check tablespaces

```sql
SELECT tablespace_name FROM dba_tablespaces;
```

3️⃣ Grant quota on SYSTEM or create a dedicated tablespace:

**Option A — Use SYSTEM for now**

```sql
ALTER USER ganza_plsqlauca_25110 QUOTA UNLIMITED ON SYSTEM;
```

**Option B — Recommended: Create your own tablespace**

```sql
CREATE TABLESPACE ga_ts_25110
  DATAFILE 'ga_ts_25110.dbf'
  SIZE 100M AUTOEXTEND ON;

ALTER USER ganza_plsqlauca_25110 QUOTA UNLIMITED ON ga_ts_25110;
```

---

## **🧪 5. Validation Commands**

### Check PDBs:

```sql
SELECT name, con_id FROM v$pdbs;
```

### Confirm datafiles in the PDB:

```sql
SELECT file_name, con_id FROM cdb_data_files WHERE con_id = 3;
```

*(Your PDB likely has **CON_ID = 3**)*

### Test login:

```sql
CONNECT ganza_plsqlauca_25110/Ganza123@ga_pdb_25110
```

---

## **📁 6. Expected Directory & Datafile Structure**

Inside the Oracle base directory:

```
/opt/oracle/oradata/ORCLCDB/GA_PDB_25110/
```

You will find:

* SYSTEM datafile
* SYSAUX datafile
* Undo
* Temp
* Any created user tablespaces

---

## **🎯 7. Summary**

| Item                | Status                        |
| ------------------- | ----------------------------- |
| CDB available       | ✔ CDB$ROOT                    |
| Seed PDB            | ✔ PDB$SEED                    |
| Student PDB created | ✔ GA_PDB_25110                |
| Admin user created  | ✔ GANZA_PLSQLAUCA_25110       |
| Roles granted       | ✔                             |
| Tablespace quota    | Pending (SYSTEM or custom TS) |

