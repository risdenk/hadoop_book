# Miscellaneous
## Curl
### SSL
Unsigned Certificate
```bash
curl -k ... 'https://...'
```
### SPNEGO (Kerberos)
```bash
kinit
curl --negotiate -u : ...
```