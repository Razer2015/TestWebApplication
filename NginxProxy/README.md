# Localhost Docker testing

OpenSSL commands tested on Windows Subsystem Linux (WSL). Docker running on Windows.

---
## Generating certificates

### Setting up the SSL configurations

1. Add your IP/DNS entry at the end of the ./Config/subjects.conf
2. Change the password from the first command below
3. Run the commands
4. Figure out how to add certificate to your device/browser
    - For example FireFox can use `./CA/ssl.crt`
    - Android requires generating a specific file from step 3

### Generate CA root

1. Generating the crt and key files (go with defaults)
```
openssl req -x509 -nodes -days 5475 -newkey rsa:2048 -keyout ./CA/ssl.key -out ./CA/ssl.crt -config ./Config/openssl.conf -passin pass:somepassword
```

2. (Optional) | Generating a PFX that can be imported/trusted (Windows, maybe others too)
```
openssl pkcs12 -export -out ./CA/ssl.pfx -inkey ./CA/ssl.key -in ./CA/ssl.crt
```

3. (Optional) | Generating certificate that can be imported/trusted (Android, maybe others too)
```
openssl x509 -inform PEM -outform DER -in ./CA/ssl.crt -out ./CA/ssl.der.crt
```

4. (Optional) | _Checking the certificate information_
```
openssl x509 -in ./CA/ssl.crt -text -noout
```

### Generate Intermediate cert

TODO

### Generate end entity cert

1. Generate private key for Certificate Signing Request (CSR)
```
openssl genrsa -out ./Certificate/cert.key 4096
```

2. Generate Certificate Signing Request (CSR)
```
openssl req -new -sha256 -key ./Certificate/cert.key -out ./Certificate/cert.csr -config ./Config/openssl.conf
```

3. Generate the end entity certificate
```
openssl x509 -req -sha256 -days 1096 -in ./Certificate/cert.csr -CA ./CA/ssl.crt -CAkey ./CA/ssl.key -out ./Certificate/cert.crt -CAcreateserial -extfile ./Config/subjects.conf
```

4. Generate certificate chain for use in Nginx (sometimes cert.crt is enough)
```
cat ./Certificate/cert.crt ./CA/ssl.crt > ./Certificate/chain.crt
```
---
## Building the application image

```
docker build -t xfilefin/test-backend:latest ..\ --file ..\TestWebApplication\Dockerfile
```

---
## Edit the Windows hosts file

You need to add the following entries to Windows hosts file (`C:\Windows\System32\drivers\etc\hosts`) when testing this on Windows.

```
127.0.0.1 grimoire.local
127.0.0.1 grimoire.gt5p.local
127.0.0.1 grimoire.gt5.local
127.0.0.1 grimoire.gt6.local
```

---
## Running the application and proxy

### Starting the containers
```
docker-compose up
```

or

```
docker stack deploy -c docker-compose.yml grimoire
```

### Shutting down the containers
```
docker-compose down
```

or

```
docker stack rm grimoire
```

### Browsing the endpoints  
  
**Region list**  
https://grimoire.local//init/regionlist.xml  

**Server lists**  
https://grimoire.local/init/gt5p/serverlist.xml  
https://grimoire.local/init/gt5/serverlist.xml  
https://grimoire.local/init/gt6/serverlist.xml  

**Grimoire backends**  
https://grimoire.gt5p.local/WeatherForecast  
https://grimoire.gt5.local/WeatherForecast  
https://grimoire.gt6.local/WeatherForecast  
