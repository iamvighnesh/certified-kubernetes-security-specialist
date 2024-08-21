# etcd Security Guidelines

## Installing etcd

> For certificate authority (CA) management, refer to the [Managing Custom Certificate Authority (CA)](../control-plane-security/custom-certificate-authority.md) section.

### Generate Server Certificates and Key

```bash
# Switch to certificates directory
cd /root/certificates

# Generate a private key for the server
openssl genrsa -out etcd-server.key 2048

# Generate a certificate signing request (CSR) for the server
openssl req -new -key etcd-server.key -out etcd-server.csr -subj "/CN=etcd-server"

# Sign the server certificate with the CA
openssl x509 -req -in etcd-server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out etcd-server.crt -days 365

# Verify the server certificate
openssl x509 -in etcd-server.crt -text -noout
```

### Generate Client Certificates and Key

```bash
# Generate a private key for the client
openssl genrsa -out etcd-client.key 2048

# Generate a certificate signing request (CSR) for the client
openssl req -new -key etcd-client.key -out etcd-client.csr -subj "/CN=etcd-client"

# Sign the client certificate with the CA
openssl x509 -req -in etcd-client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out etcd-client.crt -days 365

# Verify the client certificate
openssl x509 -in etcd-client.crt -text -noout
```

### Install ectd as a Systemd Service

```bash
# Download and install wget to download etcd
apt-get update && apt-get install -y wget

# Download etcd
wget https://github.com/etcd-io/etcd/releases/download/v3.5.4/etcd-v3.5.4-linux-amd64.tar.gz
tar xzvf etcd-v3.5.4-linux-amd64.tar.gz
mv etcd-v3.5.4-linux-amd64/etcd* /usr/local/bin/

# Start etcd as a systemd service with client cert and key
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=
After=network.target

[Service]
User=root
Type=notify
Environment=ETCD_NAME=etcd-node-1
Environment=ETCD_DATA_DIR=/var/lib/etcd
Environment=ETCD_LISTEN_PEER_URLS=https://
Environment=ETCD_LISTEN_CLIENT_URLS=https://
Environment=ETCD_INITIAL_ADVERTISE_PEER_URLS=https://
Environment=ETCD_INITIAL_CLUSTER=etcd-node-1=https://
Environment=ETCD_INITIAL_CLUSTER_STATE=new
Environment=ETCD_INITIAL_CLUSTER_TOKEN
Environment=ETCD_ADVERTISE_CLIENT_URLS=https://
Environment=ETCD_CERT_FILE=/etc/etcd/etcd-server.crt
Environment=ETCD_KEY_FILE=/etc/etcd/etcd-server.key
Environment=ETCD_TRUSTED_CA_FILE=/etc/etcd/ca.crt
Environment=ETCD_PEER_CERT_FILE=/etc/etcd/etcd-peer.crt
Environment=ETCD_PEER_KEY_FILE=/etc/etcd/etcd-peer.key
Environment=ETCD_PEER_TRUSTED_CA_FILE=/etc/etcd/ca.crt
ExecStart=/usr/local/bin/etcd
Restart=always
RestartSec=10s
LimitNOFILE=40000

[Install]
WantedBy=multi-user.target
EOF

# Reload systemd and start etcd
systemctl daemon-reload
systemctl start etcd
systemctl enable etcd

# Verify etcd is running
etcdctl version

# Verify etcd is running
etcdctl endpoint health

# Verify etcd is running
etcdctl member list

# Verify etcd is running
etcdctl put foo bar
etcdctl get foo
```

## Key Security Areas

1. Plain Text Data Storage
    - All keys and values are stored in plain text in etcd.
    - Keys and Values can be read by anyone with access to etcd.
    - Use encryption at rest to protect data stored in etcd.
2. Transport Layer Security (using HTTPS)
    - Use TLS to encrypt data in transit.
    - TLS can be used to encrypt data between etcd nodes and clients i.e. Kubernetes API Server.
3. Identity Perimeter (Authentication using HTTPS client certificates)
    - Use client certificates to authenticate clients like Kubernetes API Server.
    - Client certificates can be used to authenticate etcd nodes to each other.
    - Prefer client certificates provided by a trusted Certificate Authority (CA).
4. Network Security
    - Use firewalls to restrict access to etcd.
    - Port 2379 is used for client communication.
    - Port 2380 is used for peer communication.
5. Role-Based Access Control (RBAC)
    - Use RBAC to restrict access to etcd.
    - Use etcd's built-in RBAC system.
    - Use Kubernetes RBAC to restrict access to etcd.

[Return to Main Page](../README.md)
