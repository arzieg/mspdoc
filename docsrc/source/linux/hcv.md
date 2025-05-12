# Hashicorp Vault (HCV)

## cli

vault login 
vault status [-format=json]

vault secrets list   - list the enabled secret engines

vault auth list   
vault path-help /auth/approle

vault kv list kv-clab-endor/
vault secrets enable -path=dev-secrets -version=2 kv   // enable new kv secret in path /dev-secrets


Process: 

0. login
```
vault login
```

```
export VAULT_TOKEN=....
```


1. Create ACL policy

```
vault policy write developer-vault-policy - << EOF
path "dev-secrets/+/creds" {
   capabilities = ["create", "list", "read", "update"]
}
EOF
```
```
curl \
    -i \
    -k \
    -H "X-Vault-Token: $VAULT_TOKEN" \
    -X PUT \
    -d '{"policy":"path \"dev-secrets/data/creds\" {\n  capabilities = [\"create\", \"update\"]\n}\n\npath \"dev-secrets/data/creds\" {\n  capabilities = [\"read\"]\n}\n"}' \
    $VAULT_ADDR/v1/sys/policies/acl/developer-vault-policy

curl -k -s -H "X-Vault-Token: $VAULT_TOKEN" $VAULT_ADDR/v1/sys/policy | jq ".data.policies"
```



2. create User

In diesem Beispiel muss dann in vault "Authentication Method" userpass aktiviert werden

```
vault write /auth/userpass/users/arne \
    password='ChangeMe' \
    policies=developer-vault-policy
```

```
curl \
  -k \
  -H "X-Vault-Token: $VAULT_TOKEN" \
  -X POST \
  -d '{"type": "userpass"}' \
  $VAULT_ADDR/v1/sys/auth/userpass
```


```
curl \
  -k \
  -H "X-Vault-Token: $VAULT_TOKEN" \
  -X POST \
  -d '{"password":"ChangeMe","token_policies":"developer-vault-policy"}' \
  $VAULT_ADDR/v1/auth/userpass/users/arne
```


3. Secret Engine aktivieren

```
vault secrets enable -path=dev-secrets -version=2 kv
```

```
curl -s -k \
  -H "X-Vault-Token: $VAULT_TOKEN" \
  $VAULT_ADDR/v1/sys/mounts | jq ".data"

curl \
    -k \
    -H "X-Vault-Token: $VAULT_TOKEN" \
    -X POST \
    -d '{ "type":"kv-v2" }' \
    $VAULT_ADDR/v1/sys/mounts/dev-secrets

``` 


4. Einloggen

```
vault login -method=userpass username=arne
```

```
curl -s \
  -k \
  -X POST \
  -d '{ "password": "ChangeMe" }' \
  $VAULT_ADDR/v1/auth/userpass/login/arne | jq ".auth.client_token"

export ARNE_TOKEN=...
``` 

5. Key speichern

```
vault kv put /dev-secrets/creds api-key=E6BED968-0FE3-411E-9B9B-C45812E4737A
```

```
curl -s \
  -k \
  -H "X-Vault-Token: $ARNE_TOKEN" \
  -X PUT \
  -d '{ "data": {"password": "Driven Siberian Pantyhose Equinox"} }' \
  $VAULT_ADDR/v1/dev-secrets/data/creds | jq ".data"
``` 

6. Key auslesen

```
vault kv get /dev-secrets/creds
```

```
curl -s -k \
  -H "X-Vault-Token: $ARNE_TOKEN" \
  $VAULT_ADDR/v1/dev-secrets/data/creds | jq ".data"
```



........................

# AppRole

0. List AppRole

```
vault list auth/approle/role
```

1. Create Policy

```
create file superadmin.hcl:

path "*" {
  capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}

vault policy write super_admin super_admin.hcl
```

2. AppRole 

```
vault write auth/approle/role/super_admin policies="super_admin"

#anderes Beispiel
vault write auth/approle/role/hoth \
    token_ttl=3600 \
    token_max_ttl=14400 \
    policies="hoth_read_policy"

vault read auth/approle/role/super_admin
```

3. Secret

```
vault write -force auth/approle/role/super_admin/secret-id
```

4. Get RoleID

```
vault read auth/approle/role/super_admin/role-id

vault read auth/approle/role/<role-name>   - incl. settings
```


