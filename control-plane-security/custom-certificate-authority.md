# Managing Custom Certificate Authority (CA)

```bash
# Create a directory to store the CA files
mkdir /root/certificates
cd /root/certificates

# Create a private key for the CA
openssl genrsa -out ca.key 2048

# Create certificate signing request (CSR) for the CA
openssl req -x509 -new -key ca.key  -subj "/CN=Kubernetes-CA" -out ca.csr

# Sign the certificate with the CA
openssl x509 -req -in ca.csr -signkey ca.key -CAcreateserial -out ca.crt -days 365

# Verify the CA certificate
openssl x509 -in ca.crt -text -noout

# Clean up the CSR file
rm -f ca.csr
```
