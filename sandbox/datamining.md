# Data mining

## Online sources
As an online source of commands can be used [MariaDB » Knowledge Base » MariaDB Server Documentation » Built-in Functions](https://mariadb.com/kb/en/built-in-functions/)
with copy & paste filling the template.

## Build-in sources
As an build-in source of commands can be used `mysql.help_*` tables.


```sql
# localhost MariaDB script in progress

# List all databases
SHOW DATABASES;

# Use mysql database
USE mysql;

# List tables
SHOW tables;

# List content of help_* functions
SELECT * FROM mysql.help_topic ht WHERE 1 ;
SELECT * FROM mysql.help_category hc WHERE 1 ;
SELECT * FROM mysql.help_keyword hk WHERE 1 ;
SELECT * FROM mysql.help_relation hr WHERE 1 ;

# In a row
SELECT * FROM mysql.help_topic ht
JOIN mysql.help_category hc ON ht.help_category_id = hc.help_category_id 
WHERE 1 ;

# All in a row
SELECT * FROM mysql.help_category hc
JOIN mysql.help_topic ht ON ht.help_category_id = hc.help_category_id 
LEFT JOIN mysql.help_relation hr ON hr.help_topic_id = ht.help_topic_id 
LEFT JOIN mysql.help_keyword hk ON hr.help_keyword_id = hk.help_keyword_id 
WHERE 1 
AND hc.name  LIKE '% Function%' ;


# Basic parsing into template output - see last column
SELECT 
@name := name AS functionname,
@descr := description AS original,
@descr := REPLACE(@descr , "\n", "") AS singleline,
@descr := REGEXP_REPLACE(@descr, '^(.*)Examples.*$','\\1') AS withoutexampes,
@syntax := REGEXP_REPLACE(@descr, '^(Syntax\-\-\-\-\-\-\ )(.*)(Description\-\-\-\-\-\-\-\-\-\-\-\.*)$','\\2') AS syntaxonly,
@descr := REGEXP_REPLACE(@descr, '^Syntax\-\-\-\-\-\-\ (.*)$','\\1') AS withoutsyntaxheader,
@descr := REGEXP_REPLACE(@descr, '^(.*)Description\-\-\-\-\-\-\-\-\-\-\-\ (.*)$','\\1xxx\\2') AS withoutdescriptionheaders,
@descr := CONCAT(name, 'xxx', @descr) AS concatwithname,
@descr := REGEXP_REPLACE(@descr, '^(.*)xxx(.*)xxx(.*)$', '<template\n  autoinsert="true"\n  context="sql_mysql_mariaDB"\n  deleted="false"\n  description="\\3"\n  enabled="true"\n  name="\\1">\\2\n</template>') AS xmltemplateitem
FROM mysql.help_topic ht 
WHERE ht.name = 'DAYOFYEAR' 
OR help_category_id IN (SELECT help_category_id 	FROM mysql.help_category hc WHERE name like '%Functions')
;

# 1st Test on DAYOFYEAR function to get syntax of the function
SELECT 
name AS functionname,
description,
REGEXP_SUBSTR(description, '(?s)(?U)Syntax.*\-\-\-\-\-\-(.*)Description') AS syntaxonly
FROM mysql.help_topic ht 
WHERE ht.name = 'DAYOFYEAR' 
;

# 2nd Test on DAYOFYEAR function to get syntax of the function
SELECT 
REGEXP_REPLACE(description, '(?s)Syntax\n\-\-\-\-\-\-(.*)\n(.*)\nDescription(.*)', '\\1') AS syntaxonly,
description,
name AS functionname
FROM mysql.help_topic ht 
WHERE ht.name = 'DAYOFYEAR' 
;


```
