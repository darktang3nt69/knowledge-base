Here are some additional HashiCorp Vault CLI commands:

### **Secrets Engines**

- **List Enabled Secrets Engines:**
    
    sh
    
    Copy code
    
    `vault secrets list`
    
- **Disable Secrets Engine:**
    
    sh
    
    Copy code
    
    `vault secrets disable [path]`
    

### **Dynamic Secrets**

- **Generate Dynamic Database Credentials:**
    
    sh
    
    Copy code
    
    `vault read database/creds/[role-name]`
    

### **Leases**

- **List Leases:**
    
    sh
    
    Copy code
    
    `vault list sys/leases/lookup/[path]`
    
- **Revoke a Lease:**
    
    sh
    
    Copy code
    
    `vault lease revoke [lease_id]`
    

### **Audit**

- **Enable Audit Device:**
    
    sh
    
    Copy code
    
    `vault audit enable file file_path=/path/to/audit.log`
    
- **List Audit Devices:**
    
    sh
    
    Copy code
    
    `vault audit list`
    

### **PKI**

- **Generate a Certificate:**
    
    sh
    
    Copy code
    
    `vault write pki/issue/[role-name] common_name=[domain]`
    

### **KV (Key-Value) Version 2**

- **List Secrets:**
    
    sh
    
    Copy code
    
    `vault kv list [path]`
    
- **Delete a Secret Version:**
    
    sh
    
    Copy code
    
    `vault kv delete [path]`
    

These commands cover more advanced Vault operations, such as dynamic secrets, auditing, and key-value management.

4o