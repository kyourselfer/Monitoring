### Task
## Connect Oracle Tablespace to the Zabbix monitoring system. The result should be displayed as a percentage.
--- UserParameter=oracle.tablespace_space[*]
`/etc/zabbix/zabbix_agentd.d/oracle.conf`
```
UserParameter=oracle.tablespace_space[*],sudo -u oracle /etc/zabbix/scripts/oracle_tablespace_space.sh $1
```
--- scripts/oracle_tablespace_space.sh
`/etc/zabbix/scripts/oracle_tablespace_space.sh`
```
#!/bin/bash

. /home/oracle/.bash_profile

justconnect(){
sqlplus -s -L zabbix/ZabbixPassWork << SQL
set echo off;
set tab off;
set pagesize 0;
set feedback off;
set trimout on;
set heading off;
select round((b.BYTES-a.BYTES)*100/b.BYTES,2) percent_used
from  (select TABLESPACE_NAME, sum(BYTES) BYTES from dba_free_space where TABLESPACE_NAME = '$1' group by TABLESPACE_NAME) a,
      (select TABLESPACE_NAME, sum(BYTES) BYTES from dba_data_files where TABLESPACE_NAME = '$1' group by TABLESPACE_NAME) b
where a.TABLESPACE_NAME=b.TABLESPACE_NAME;
SQL
}

# Values
TB1=$1;

RESULT=$( justconnect $TB1 );

#убираем пробелы
RESULT=`(echo ${RESULT} | fmt -su)`

#проверяем на число и точки
if [[ "${RESULT}" =~ ^[0-9\.]+$ ]]; then
:
else
RESULT="-1"
fi

echo $RESULT;
exit 0;

```
--- Checks
zabbix_get -k 'oracle.tablespace_space[USERS]' -s 127.0.0.1
