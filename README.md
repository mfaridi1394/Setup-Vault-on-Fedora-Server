# Setup-Vault-on-Fedora-Server in production mode
Setup Vault hashicorp on Fedora Server
Vault is use for keep our secret safe and secure.
For setup Vault in production mode in Fedora Server, we must do:
wget vault from hashicorp website

```
unzip vault*.zip
mv vault /usr/local/bin

vault --version

vault -autocomplete-install

complete -C /usr/bin/vault vault

mkdir /etc/vault

mkdir -p /var/lib/vault/data

sudo useradd --system --home /etc/vault --shell /bin/false vault

sudo chown -R vault:vault /etc/vault /var/lib/vault/
```

For start at boot We can use systemd
```
vim /etc/systemd/system/vault.service

[Unit]
Description=vault server
Requires=network-online.target
After=network-online.target
ConditionFileNotEmpty=/etc/vault/config.hcl

[Service]
User=vault
Group=vault
Restart=on-failure
ExecStart=/usr/local/bin/vault server -config=/etc/vault
ExecStop=/usr/local/bin/vault step-down
#ExecReload=/bin/kill --signal HUP $MAINPID

[Install]
WantedBy=multi-user.target
```
This is my config.hcl in production mode
```
vim /etc/vault/config.hcl

#mlock = true
disable_mlock = true
ui = true
storage "file" {
path = "/var/lib/vault/data"
}

#storage "consul" {
# address = "127.0.0.1:8500"
# path = "vault"
#}

# HTTP listener
#listener "tcp" {
# address = "127.0.0.1:8202"
# tls_disable = "false"
#}

# HTTPS listener
listener "tcp" {
address = "0.0.0.0:8200"
tls_disable = "false"
tls_cert_file = "/var/lib/vault/data/tls/tls.crt"
tls_key_file = "/var/lib/vault/data/tls/tls.key"
}


cluster_addr="http://127.0.0.1:8201"
api_addr="http://127.0.0.1:8200"

```

I use file for store data, you can use other File storage for your project.

 cd /var/lib/vault/data/
 
 mkdir tls
 
 cd tls
 ```
 sudo openssl req -out tls.crt -new -keyout tls.key -newkey rsa:4096 -nodes -sha256 -x509 -subj "/O=HashiCorp/CN=Vault" -addext "subjectAltName =IP:127.0.0.1,IP:192.168.90.125,DNS:vault.mfaridi.local"  -days 3650 
 ```
 We must copy our self sign certificate 

cp tls.crt /etc/pki/ca-trust/source/anchors

and update our certificate cache
```
sudo update-ca-trust

sudo systemctl daemon-reload

systemctl enable --now vault

sudo systemctl enable --now vault

sudo systemctl status vault

sudo journalctl -u vault

sudo chown -R vault:vault tls/

sudo systemctl restart vault

export VAULT_ADDR=https://192.168.90.125:8200

echo "export VAULT_ADDR=https://192.168.90.125:8200" >> ~/.bashrc
```
We must init the Keys and tokens for use Vault
```
sudo vault operator init > /etc/vault/init.file
```
after init we must export our root token 

export VAULT_TOKEN="****************************"

vault status can show us our vault server is sealed and must be unseal
we use this commands  for unsealed vault
```
vault operator unseal key1

vault operator unseal key2

vault operator unseal key3
```

We can use this command for create first secret store

vault secrets enable -version=1 -path=secret/application kv

We cacn use this command for put our username and password from JSON file to our secret store

vault kv put secret/application/prod @/home/mostafa/applicationazmoonoracle.json



