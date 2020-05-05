# Overview

This document goes through the workflow of installing the sample schemas for v18c provided by Oracle.

[Sample Schema Repository](https://github.com/oracle/db-sample-schemas)

[Sample Schema Docs](https://docs.oracle.com/en/database/oracle/oracle-database/19/comsc/index.html)

## Installation of HR Schema

1. Download and Unzip the correct version of the sample schemas (v18c in this example)

  ```shell
  wget "https://github.com/oracle/db-sample-schemas/archive/v18c.tar.gz" 
  tar -xf v18c.tar.gz
  ```

2. `CD` into `v18c` and update installation scripts with current working directory

    `sudo perl -p -i.bak -e 's#__SUB__CWD__#'$(pwd)'#g' *.sql */*.sql */*.dat`

3. Set the Oracle Environment

    `source /usr/local/bin/oraenv`

4. Logon as `SYS` using the `SYSDBA` privilege

    `sqlplus sys as sysdba`

5. Identify Pluggable DB Name and Service

    Pluggable DB Name and ID (Should be `XEPDB1` and `3`)
   
    `SELECT name, con_id FROM v$pdbs;`

    Pluggable DB Service (Should be `xepdb1`)

    `SELECT name FROM v$active_services WHERE con_id=3;`

6. Execute the `mksample.sql` script to build all schemas

    `@mksample`
    
    * Password for `SYSTEM` and `SYS` are the current ones
    * Passwords for `HR|OE|PM|IX|SH|BI` are all new. Recommend using lowercase schema names as password. (e.g.; HR == hr)
    * `Default Tablespace` == `users`
    * `Temporary Tablespace` == `temp`
    * `Log File Directory` == `$ORACLE_HOME/demo/schema/log`
    * `Connect String` == `localhost:1521/xepdb1`
      * This is the Service Name found in Step 5

7. Switch your session to the pluggable database

    `ALTER SESSION SET CONTAINER = xepdb1;`

8. Switch your session to the target schema (`HR` in this example)

    `ALTER SESSION SET CURRENT_SCHEMA = HR;`

9. Validate everything worked

    `SELECT * FROM REGIONS;`

    Expected Output: 

    ```
    SQL> SELECT * FROM REGIONS;
    
     REGION_ID REGION_NAME
    ---------- -------------------------
    	 1 Europe
    	 2 Americas
    	 3 Asia
    	 4 Middle East and Africa
    ```
