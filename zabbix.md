# Zabbix Notes

[Docker Hub Reference](https://hub.docker.com/r/zabbix/zabbix-appliance)

### Building a Zabbix appliance:
User: `Admin`
Pass: `zabbix`
```
docker pull zabbix/zabbix-appliance

docker run --name local-zabbix-appliance -p 80:80 -p 10051:10051 -d zabbix/zabbix-appliance:alpine-trunk
```

### Connecting to Zabbix DB
User: `zabbix`
Pass: `zabbix`
```
docker exec -ti local-zabbix-appliance /bin/bash
mysql -u zabbix -p
<password>
```

### Query Examples

[MariaDB Reference](https://mariadb.com/kb/en/getting-data-from-mariadb/)

* List Databases `SHOW DATABASES;`
* List Tables `SHOW TABLES FROM <dbName>`
* Query Table `SELECT key_ as 'key', name as 'display_name', history as 'retention' FROM zabbix.items \G`

[Zabbix DB Schema](https://zabbix.org/wiki/Database_Schemas)
