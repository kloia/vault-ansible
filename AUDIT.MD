# Enable File and syslog audit 

Enable file audit
````
export VAULT_SKIP_VERIFY=true
export VAULT_TOKEN="<ROOT-TOKEN>"

vault audit enable file \
    file_path=/var/log/vault/vault_audit.log \
    hmac_accessor=false \
    elide_list_responses=true

````

Enable syslog audit 
````
vault audit enable syslog tag="vault" facility="LOCAL0"

touch /etc/rsyslog.d/30-vault.conf
# LOCAL0 facility kullanan Vault audit logları
local0.*    /var/log/vault/audit.log
local0.*    @@<SYSLOG_SERVER_IP>:514 
````
