# etcd Security Guidelines

## Installing etcd

```bash
# Download and install wget to download etcd
apt-get update && apt-get install -y wget

# Download etcd
wget https://github.com/etcd-io/etcd/releases/download/v3.5.4/etcd-v3.5.4-linux-amd64.tar.gz
tar xzvf etcd-v3.5.4-linux-amd64.tar.gz
mv etcd-v3.5.4-linux-amd64/etcd* /usr/local/bin/

# Verify the installation
etcd --version
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
