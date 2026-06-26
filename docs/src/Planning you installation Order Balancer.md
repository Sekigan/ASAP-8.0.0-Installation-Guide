### Planning Your Installation

You can plan your installation based on the available resources and your business requirements.

**WebLogic Server**
**You can install Order Balancer on an existing ASAP WebLogic Server or on a separate WebLogic Server. If you choose to install Order Balancer on an existing ASAP WebLogic Server, install it in a separate domain. If you choose to use a separate WebLogic Server, install WebLogic Server and create a domain for Order Balancer.**

**Database User**
**You can use the same database as the ASAP database or a different database. In both the cases, create a separate tablespace and a user.**


For example:
```bash
create tablespace "OBDATA_TS" logging datafile '/u01/app/oracle/oradata/ASAP72/OBDATA_TS1.dbf' size 31999M reuse autoextend on next 5120K maxsize unlimited default storage (initial 256K next 256K minextents 1 maxextents 2147483645 pctincrease 0);
```

```bash
CREATE USER "OB_SYS_USR" PROFILE "DEFAULT" IDENTIFIED BY "OB_SYS_USR" DEFAULT TABLESPACE "OBDATA_TS" TEMPORARY TABLESPACE "TEMP" ACCOUNT UNLOCK;
```

```bash
GRANT UNLIMITED TABLESPACE TO "OB_SYS_USR" WITH ADMIN OPTION; 
GRANT "CONNECT" TO "OB_SYS_USR" WITH ADMIN OPTION; 
GRANT "RESOURCE" TO "OB_SYS_USR" WITH ADMIN OPTION; 
GRANT CREATE VIEW TO "OB_SYS_USR" WITH ADMIN OPTION; 
GRANT EXECUTE ON SYS.UTL_FILE TO "OB_SYS_USR" WITH GRANT OPTION; 
GRANT CREATE PROCEDURE TO "OB_SYS_USR" WITH ADMIN OPTION; 
GRANT CREATE TRIGGER TO "OB_SYS_USR" WITH ADMIN OPTION; 
GRANT CREATE TABLE TO "OB_SYS_USR" WITH ADMIN OPTION; 
GRANT CREATE TYPE TO "OB_SYS_USR" WITH ADMIN OPTION; 
GRANT CREATE SEQUENCE TO "OB_SYS_USR" WITH ADMIN OPTION;
GRANT CREATE SESSION TO "OB_SYS_USR" WITH ADMIN OPTION;
```
