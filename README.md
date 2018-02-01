# Oracle Data Pump utilities 
Use case: we want to do export and import one big Oracle database via `Data Pump` tool. This big database has hundreds tables. Some are really big and some are not. We need to divide tables of this database into groups so that we can do parallel export and import for each individual group. 


## 1 data pump export 

### 1.1 Get all table names using queries below

```
SELECT OWNER || '.' || OBJECT_NAME FROM ALL_OBJECTS WHERE OWNER = 'SCHEMA_NAME' AND  OBJECT_TYPE = 'TABLE';
```

OR 
```
SELECT 'SCHEMA_NAME' || '.' || OBJECT_NAME FROM USER_OBJECTS WHERE  OBJECT_TYPE = 'TABLE';
```

replace `SCHEMA_NAME` here with real schema-name. 

### 1.2 Divide tables into groups
We divide tables into groups and each group will have some portion of tables. You are trying to create balanced groups. Group with big table will have small portion of tables. Name conventions for each group are defined as below. For example, `group 1` related are defined as below. 

```
# export par file 
expdp_{schema_name}_grp1.par 
# export dump file 
expdp_{schema_name}_grp1%u.dump
# export log file 
expdp_{schema_name}_grp1.log
# export job name 
expdp_{schema_name}_grp1
```

The export `par` file of `group1`- `expdp_{schema_name}_grp1.par` is defined as below.

```
userid='/ as sysdba'
CONTENT=ALL
DIRECTORY=dpump_dir
parallel=3
TABLES=
SCHEMA_NAME.TABLE1,
SCHEMA_NAME.TABLE2,
SCHEMA_NAME.TABLE3,
SCHEMA_NAME.TABLE4,
SCHEMA_NAME.TABLE5,
SCHEMA_NAME.TABLE6,
SCHEMA_NAME.TABLE7,
SCHEMA_NAME.TABLE8,
SCHEMA_NAME.TABLE9,
SCHEMA_NAME.TABLE10,
SCHEMA_NAME.TABLE11,
SCHEMA_NAME.TABLE12,
SCHEMA_NAME.TABLE13,
SCHEMA_NAME.TABLE14,
SCHEMA_NAME.TABLE15,
SCHEMA_NAME.TABLE16,
SCHEMA_NAME.TABLE17,
SCHEMA_NAME.TABLE18

dumpfile=expdp_SCHEMA_NAME_grp1%u.dmp
JOB_NAME=expdp_SCHEMA_NAME_grp1
logfile=expdp_SCHEMA_NAME_grp1.log

flashback_time="TO_TIMESTAMP (TO_CHAR (SYSDATE, 'YYYY-MM-DD HH24:MI:SS'), 'YYYY-MM-DD HH24:MI:SS')"
filesize=4G
```

Replace with real `SCHEMA_NAME` and `TABLE_NAME` you got from last step. 

### 1.3 Run real data pump export 

```
expdp parfile=expdp_{schema_name}_grp1.par  &
expdp parfile=expdp_{schema_name}_grp2.par  &
expdp parfile=expdp_{schema_name}_grp3.par  &
expdp parfile=expdp_{schema_name}_grp4.par  &
expdp parfile=expdp_{schema_name}_grp5.par  &
```

## 2 data pump import 

Name conventions of import files: 
```
# impdp par file 
impdp_{schema_name}_grp1.par 
# exported dump file 
expdp_{schema_name}_grp1%u.dump
# import log file 
impdp_{schema_name}_grp1.log
# import job name 
impdp_{schema_name}_grp1
```

The import `par` file of `group1`- `impdp_{schema_name}_grp1.par` is defined as below.

```
userid='/ as sysdba'
CONTENT=DATA_ONLY
DIRECTORY=dpump_dir
parallel=3
TABLES=
SCHEMA_NAME.TABLE1,
SCHEMA_NAME.TABLE2,
SCHEMA_NAME.TABLE3,
SCHEMA_NAME.TABLE4,
SCHEMA_NAME.TABLE5,
SCHEMA_NAME.TABLE6,
SCHEMA_NAME.TABLE7,
SCHEMA_NAME.TABLE8,
SCHEMA_NAME.TABLE9,
SCHEMA_NAME.TABLE10,
SCHEMA_NAME.TABLE11,
SCHEMA_NAME.TABLE12,
SCHEMA_NAME.TABLE13,
SCHEMA_NAME.TABLE14,
SCHEMA_NAME.TABLE15,
SCHEMA_NAME.TABLE16,
SCHEMA_NAME.TABLE17,
SCHEMA_NAME.TABLE18

dumpfile=expdp_{schema_name}_grp1%u.dump
JOB_NAME=imp_SCHEMA_NAME_grp1
logfile=impdp_SCHEMA_NAME_grp1.log

logtime=all
TABLE_EXISTS_ACTION=TRUNCATE
```

### 2.1 Run import 
```
impdp parfile=impdp_{schema_name}_grp1.par  &
impdp parfile=impdp_{schema_name}_grp2.par  &
impdp parfile=impdp_{schema_name}_grp3.par  &
impdp parfile=impdp_{schema_name}_grp4.par  &
impdp parfile=impdp_{schema_name}_grp5.par  &
```


## 3 import individual tables
In this cases, we just want to import individual tables across all groups, not all of them. 

create the `table_dmp_file.mapping` file, 
```
expdp_SCHEMA_NAME_grp1.par:TABLE1
expdp_SCHEMA_NAME_grp1.par:TABLE2
expdp_SCHEMA_NAME_grp1.par:TABLE3
expdp_SCHEMA_NAME_grp1.par:TABLE4
expdp_SCHEMA_NAME_grp1.par:TABLE5
expdp_SCHEMA_NAME_grp1.par:TABLE6
expdp_SCHEMA_NAME_grp1.par:TABLE7
expdp_SCHEMA_NAME_grp1.par:TABLE8
expdp_SCHEMA_NAME_grp1.par:TABLE9
expdp_SCHEMA_NAME_grp1.par:TABLE10
expdp_SCHEMA_NAME_grp2.par:TABLE11
expdp_SCHEMA_NAME_grp2.par:TABLE12
expdp_SCHEMA_NAME_grp2.par:TABLE13
expdp_SCHEMA_NAME_grp2.par:TABLE14
expdp_SCHEMA_NAME_grp2.par:TABLE15
expdp_SCHEMA_NAME_grp2.par:TABLE16
expdp_SCHEMA_NAME_grp3.par:TABLE17
expdp_SCHEMA_NAME_grp3.par:TABLE18
expdp_SCHEMA_NAME_grp3.par:TABLE19
expdp_SCHEMA_NAME_grp3.par:TABLE20
expdp_SCHEMA_NAME_grp3.par:TABLE21
expdp_SCHEMA_NAME_grp3.par:TABLE22
expdp_SCHEMA_NAME_grp4.par:TABLE23
expdp_SCHEMA_NAME_grp4.par:TABLE24
expdp_SCHEMA_NAME_grp4.par:TABLE25
expdp_SCHEMA_NAME_grp4.par:TABLE26
expdp_SCHEMA_NAME_grp4.par:TABLE27
expdp_SCHEMA_NAME_grp4.par:TABLE28
expdp_SCHEMA_NAME_grp4.par:TABLE29
expdp_SCHEMA_NAME_grp4.par:TABLE30
expdp_SCHEMA_NAME_grp4.par:TABLE31
expdp_SCHEMA_NAME_grp4.par:TABLE32
expdp_SCHEMA_NAME_grp4.par:TABLE33
expdp_SCHEMA_NAME_grp5.par:TABLE34
expdp_SCHEMA_NAME_grp5.par:TABLE35
expdp_SCHEMA_NAME_grp5.par:TABLE36
expdp_SCHEMA_NAME_grp5.par:TABLE37
```

Create a template impdp par file - `impdp_SCHEMA_NAME_TABLE_TEMPLATE.par`
```
userid='/ as sysdba'
CONTENT=DATA_ONLY
DIRECTORY=dpump_dir
parallel=3
TABLES=
SCHEMA_NAME.TABLE_NAME

dumpfile=DMPFILELOCATION
JOB_NAME=impdp_SCHEMA_NAME_TABLE_NAME
logfile=impdp_SCHEMA_NAME_TABLE_NAME.log

logtime=all
TABLE_EXISTS_ACTION=TRUNCATE
```


Generate import par file for each table via this script `run_datapump_import_generate.sh`

```
#!/bin/bash
SCHEMA_NAME=SCHEMA_NAME
DMP_FOLDER=dpump_dir
IMPORT_PAR_TEMPLATE=impdp_SCHEMA_NAME_TABLE_TEMPLATE.par

while read line
do
     TABLE_MAPPING=$line
     IFS=':' read -r -a arr <<< "$TABLE_MAPPING"
     DMP_FILENAME=${DMP_FOLDER}/${arr[0]}%u.dmp
     TABLE_NAME=${arr[1]}
     IMPORT_DMP_PAR_FILE=$DMP_FOLDER/impdp_${SCHEMA_NAME}_$TABLE_NAME\.par
     sed -e "s/TABLE_NAME/$TABLE_NAME/g" -e "s/DMPFILELOCATION/$DMP_FILENAME/g" $IMPORT_PAR_TEMPLATE > $IMPORT_DMP_PAR_FILE
done < table_dmp_file.mapping

```

import the individual table one by one via `run_datapump_import_individual_table.sh`

```
#!/bin/bash
SCHEMA_NAME=SCHEMA_NAME
DMP_FOLDER=dpump_dir

while read line
do
     TABLE_MAPPING=$line
     IFS=':' read -r -a arr <<< "$TABLE_MAPPING"
     TABLE_NAME=${arr[1]}
     IMPORT_DMP_PAR_FILE=$DMP_FOLDER/impdp_${SCHEMA_NAME}_${TABLE_NAME}\.par
     # do impdp
     impdp parfile=$IMPORT_DMP_PAR_FILE
done < table_dmp_file.mapping
```