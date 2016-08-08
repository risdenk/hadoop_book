# Kerberos
## kinit
```bash
kinit
kinit PRINCIPAL
kinit -kt ${KEYTAB}.keytab
kinit -kt ${KEYTAB}.keytab PRINCIPAL
```

## klist
```bash
klist
klist -kt ${KEYTAB}.keytab

# List encryption types
klist -e
klist -e -kt ${KEYTAB}.keytab
```

## Keytabs
A keytab must be treated like a password. Although it is not in clear text, someone having access to a keytab is just like having access to your password since they can then impersonate you.

### Manually Creating Keytab (w/ password)
```bash
ktutil
```
```
add_entry -password -p principal@EXAMPLE.COM -k 3 -e aes256-cts-hmac-sha1-96
add_entry -password -p principal@EXAMPLE.COM -k 3 -e aes128-cts-hmac-sha1-96
add_entry -password -p principal@EXAMPLE.COM -k 3 -e des3-cbc-sha1
write_kt
quit
```
Available Encryption Types
```
aes256-cts-hmac-sha1-96
aes128-cts-hmac-sha1-96
arcfour-hmac
des-cbc-md5
des-cbc-crc
```
#### ktutil and Active Directory
* http://kerberos.996246.n3.nabble.com/ktutil-problems-generating-AES-keys-salt-tp41104p41129.html

You cannot specify the salt when using ktutil. What you need to do 
instead is to specify exactly the same principal that AD uses as salt. 

For machine accounts this salting principal is 
`host/sAMAccountname.domain_name@DOMAIN_NAME`
```
   sAMAccountname := the machines samaccountname without "$" 
   domain_name := name of AD domain name in lowercase 
   DOMAIN_NAME := name of AD domain name in uppercase 
```

For user accounts this salting principal is username@DOMAIN_NAME 
```
username := first component of userPrincipalname or sAMAccountName if userPrincipalname attribute is not set
```