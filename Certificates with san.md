To set the **Subject Alternative Name (SAN)** when creating a certificate for your Nginx server, you'll need to create an OpenSSL configuration file that includes the SAN extension, as OpenSSL doesn’t allow you to specify SAN directly from the command line.

### Here’s how you can do it:

### Step-by-Step Process to Add SAN to a Certificate

#### 1. **Create a Configuration File for OpenSSL**

You need to create an OpenSSL configuration file that includes the SAN extension. Start by creating a file called `domain1.com.conf`.

```bash
nano domain1.com.conf
```

Inside this file, add the following content:

```conf
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[req_distinguished_name]
C = US
ST = California
L = San Francisco
O = Example Company
CN = domain1.com  # This is the common name (CN) for the certificate

[v3_req]
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = domain1.com        # Primary domain (matches CN)
DNS.2 = www.domain1.com     # Additional domain/subdomain
DNS.3 = sub.domain1.com     # Another subdomain (you can add more)
```

- **[req_distinguished_name]**: Contains the information for the common name (CN) and other attributes of the certificate.
- **[alt_names]**: Lists the **Subject Alternative Names (SAN)**. Each `DNS.n` entry represents an additional domain name or subdomain you want to include in the SAN.
  
You can add more `DNS.*` entries for each domain/subdomain you want to support.

#### 2. **Generate a Private Key for Your Domain**

Generate a private key for your domain:

```bash
openssl genrsa -out domain1.com.key 2048
```

#### 3. **Create a Certificate Signing Request (CSR)**

Use the configuration file to generate a Certificate Signing Request (CSR) that includes the SAN information:

```bash
openssl req -new -key domain1.com.key -out domain1.com.csr -config domain1.com.conf
```

This will generate a CSR (`domain1.com.csr`) that includes both the Common Name (CN) and the Subject Alternative Name (SAN) fields.

#### 4. **Sign the CSR with Your Root CA**

Use your previously generated root CA to sign the CSR and create a certificate:

```bash
openssl x509 -req -in domain1.com.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out domain1.com.crt -days 500 -sha256 -extfile domain1.com.conf -extensions v3_req
```

- `-extfile domain1.com.conf -extensions v3_req`: This ensures that OpenSSL reads the SAN extension from your config file when signing the certificate.

Now, you’ll have a signed certificate (`domain1.com.crt`) that includes the SANs you specified.

#### 5. **Configure Nginx to Use the New Certificate**

Update your Nginx configuration to use the newly created certificate and private key:

```nginx
server {
    listen 443 ssl;
    server_name domain1.com www.domain1.com sub.domain1.com;

    ssl_certificate /etc/nginx/ssl/domain1.com.crt;
    ssl_certificate_key /etc/nginx/ssl/domain1.com.key;

    # Other Nginx settings
    ...
}
```

#### 6. **Restart Nginx**

Finally, restart Nginx to apply the changes:

```bash
sudo systemctl restart nginx
```

### Verifying the Certificate

You can verify that the SAN is correctly included in the certificate by running:

```bash
openssl x509 -in domain1.com.crt -text -noout
```

Look for the `X509v3 Subject Alternative Name` section, which should list all the DNS names (SANs) you specified.

### Summary

1. **Create a config file** to specify the SAN extension.
2. **Generate a private key** and a **CSR** using that configuration.
3. **Sign the CSR** with your root CA and include the SANs in the certificate.
4. **Configure Nginx** to use the new certificate and **restart** it.

This will ensure that your Nginx server certificate includes both the CN and SAN fields, and supports multiple domains or subdomains. Let me know if you need further clarification!