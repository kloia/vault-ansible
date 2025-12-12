# Create self-signed certificates for a Vault Raft cluster

This document describes how to create a self-signed CA and server certificate for a Vault Raft cluster. Update the SAN (Subject Alternative Names) list to match your hostnames/IPs.

## Prerequisites
- openssl installed
- Optional: working directory where you keep generated certs (example: `certs/`)

## Steps

1. Create a working directory:
```bash
mkdir -p certs
cd certs
```

2. Generate a CA private key and self-signed CA certificate:
```bash
openssl genrsa -out myvault-ca.key 4096
openssl req -x509 -new -nodes -key myvault-ca.key -sha256 -days 1825 \
  -out myvault-ca.crt -subj "/CN=MyVaultRootCA"
```

3. Generate a server private key:
```bash
openssl genrsa -out vault.key 2048
```

4. Create a CSR configuration file (`vault-csr.conf`) to include SAN entries. Adjust DNS/IP entries for your environment:
```ini
[req]
prompt = no
distinguished_name = dn
req_extensions = req_ext

[dn]
CN = vault.local

[req_ext]
subjectAltName = @alt_names

[alt_names]
DNS.1 = vault
DNS.2 = vault.local
DNS.3 = ip-172-31-47-31.eu-central-1.compute.internal
DNS.4 = ip-172-31-37-99.eu-central-1.compute.internal
DNS.5 = ip-172-31-45-142.eu-central-1.compute.internal
DNS.6 = ip-172-31-45-83.eu-central-1.compute.internal
DNS.7 = ec2-3-121-202-22.eu-central-1.compute.amazonaws.com
DNS.8 = ec2-3-120-31-116.eu-central-1.compute.amazonaws.com
DNS.9 = ec2-63-178-227-160.eu-central-1.compute.amazonaws.com
DNS.10 = ec2-3-124-206-77.eu-central-1.compute.amazonaws.com
IP.1 = 127.0.0.1
```

5. Generate the CSR using the config:
```bash
openssl req -new -key vault.key -out vault.csr -config vault-csr.conf
```

6. Sign the CSR with your CA to produce the server certificate (includes SANs):
```bash
openssl x509 -req -in vault.csr -CA myvault-ca.crt -CAkey myvault-ca.key \
  -CAcreateserial -out vault.crt -days 825 -sha256 -extensions req_ext \
  -extfile vault-csr.conf
```

7. Copy the generated files into your Ansible role or files directory (example):
```bash
cp vault.crt vault.key myvault-ca.crt ../roles/vault/files/
```

## Notes and recommendations
- SANs are required for TLS hostname verification. Ensure every hostname/IP used to reach Vault is listed.
- Adjust key sizes and certificate validity (`-days`) according to security policies.
- For production, prefer certificates issued by a trusted internal or public CA rather than self-signed.
- Protect private keys (`*.key`) and avoid committing them to version control. Add them to `.gitignore`.
