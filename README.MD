### 1

mkdir vault_config vault_file vault_logs

### 2
- Your AltName must match container name and container variables!
- Generate Root Crt and Vault Crt

mkdir -p /ca/vault && cd /ca

cp /etc/ssl/openssl.cnf ./

- Edit openssl.conf default parameters
vi ./openssl.cnf

```
[ CA_default ]

#dir            = ./demoCA              # Where everything is kept
dir             = /ca                   # Where everything is kept
```
- Comment line if "random number generator" error occurs
```
# RANDFILE = $ENV::HOME/.rnd
```
openssl genrsa -config ./openssl.cnf -out ca.key 2048

openssl req -new -x509 -days 3650 -key ca.key -subj "/C=RU/ST=UF/L=Moscow/O=VAULT/CN=CA" -config ./openssl.cnf -out ca.crt

openssl req -newkey rsa:2048 -nodes -keyout vault/vault.key -subj "/C=RU/ST=UF/L=Moscow/O=VAULT/CN=vault1" -config ./openssl.cnf -out vault/vault.csr
openssl x509 -req -extfile <(printf "subjectAltName=DNS:vault1,DNS:127.0.0.1") -days 3650 -in vault/vault.csr -CA ca.crt -CAkey ca.key -CAcreateserial -config ./openssl.cnf -out vault/vault.crt

- Read your Crt for test:

openssl x509 -in vault/vault.crt -text -noout

### 3
- Put ca.crt (for Vault verification), vault.crt, vault.key to vault_config
cp /ca/vault/{ca.crt,vault.crt,vault.key} ./vault_config/
- Grant read to Crt

chmod o+r vault_config/ca.crt vault_config/vault.*

### 4 
- Before use cli you must unseal Vault (Enter Token Keys), and

vault login
- (Enter your token)
