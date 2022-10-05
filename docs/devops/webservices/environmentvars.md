---
title: "Apache Environment Variables"
date: 2021-05-24T11:29:58-04:00
draft: false
description: "Displaying Environment Variables"
---

## Enabling CGI on Apache2
1. ###### Install Apache2
```
sudo apt update -y
sudo apt install apache2 -y
```
2. ###### Enable CGI
```
sudo ln -s /etc/apache2/mods-available/cgi.load /etc/apache2/mods-enabled/
sudo systemctl restart apache2
```
3. ###### HTTP Envelop Scripts 
```
- cd /usr/lib/cgi-bin/ 
- nano http-env
```

<table>
<tr>
<th>Perl</th>
<th>Python</th>
</tr>
<tr>
<td>

```python
#!/usr/bin/perl
 
print "Content-type: text/html\n\n";
print "<pre>\n";
 
foreach $key (sort
keys(%ENV)) {
  print "$key = $ENV{$key}<p>";
}
 
print "</pre>\n";
```

</td>
<td>

```python
#!/usr/bin/python3
 
import os
 
def main():
    print("Content-type:text/html\r\n\r\n")
 
    for var in sorted(os.environ.keys()):
        print('%s = %s</br/>' % (var, os.environ[var]))
 
    return 0
 
 
if __name__ == '__main__':
    main()
```

</td>
</tr>
</table>

4. Make http-env exectuable
```
chmod +x /usr/lib/cgi-bin/http-env
```

#### Bash Script
- set executable permissions
  * sudo chmod +x http-env.sh
  * ./http-env.sh

##### http-env.sh
```python
echo " ### Apache2 - Install Apache ### "
sudo apt update -y
sudo apt install apache2 -y
 
echo " ### Apache2 - Enable CGI ### "
sudo ln -s /etc/apache2/mods-available/cgi.load /etc/apache2/mods-enabled/
sudo systemctl restart apache2
 
echo " ### Apache2 - Create HTTP-ENV Script ### "
cat <<EOT > /usr/lib/cgi-bin/http-env
#!/usr/bin/python3
 
import os
 
 
def main():
    print("Content-type:text/html\r\n\r\n")
 
    for var in sorted(os.environ.keys()):
        print('%s = %s</br/>' % (var, os.environ[var]))
 
    return 0
 
 
if __name__ == '__main__':
    main()
EOT
 
echo " ### Apache2 - Set Permissions on HTTP-ENV Script ### "
sudo chmod 755 /usr/lib/cgi-bin/http-env
 
echo " ### Apache2 - Verification ### "
verification=$(curl http://127.0.0.1/cgi-bin/http-env)
echo "$verification"
 
echo 

