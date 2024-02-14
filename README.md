# This code is for below security requirement

1. You have a number of Windows clients which already joined enterprise AD.
2. based on security base line, most of your users have no local administrator right.
3. management on local administrator password is a huge workload for the ICT people.
   
   how to do management password based on ISO27002:
   
        1. administrator password shall be regular changed.

        2. aline with password policy.


# Tools we used

    hashicorp/vault Community
    
    https://github.com/hashicorp/vault
    
    https://developer.hashicorp.com/vault/install?product_intent=vault


# Vault configuration

## 1. Common config.hcl

ui = true

disable_mlock = true

storage "raft" {

  path    = "./vault/data"
  
  node_id = "node1"

}


listener "tcp" {

  address     = "0.0.0.0:8200"
  
  tls_disable = "true"

}


api_addr = "http://127.0.0.1:8200"

cluster_addr = "https://127.0.0.1:8201"

## 2. Secrets engines

   2 KV Secrets engines, one is version2 , the other is version 1.

   like below
   
<img width="332" alt="image" src="https://github.com/parrotao/Vault_Windows_Admins_Password_Change/assets/37337484/6a79f0f6-4a72-4fd2-a23d-a803e5d0be7e">

## 3. Create one user for each client

## 4. Create new policy for this user

<<PolicyName = win_pass>>

path "kv/*" {

capabilities = ["create", "read", "update", "delete", "list"]

}

path "kv_v1/*" {

capabilities = [ "update"]

}

path "sys/capabilities-self" { 

    capabilities = ["create", "read", "update", "delete", "list"]

}

path "sys/mounts/*"

{

capabilities = [ "read"]

}

path "sys/mounts" {

capabilities = [ "read"]

}

## 5. Assign policy to user

vault write auth/userpass/users/<%user_name%>  policies=win_pass

and Do Not Attach 'default' Policy To Generated Tokens

<img width="699" alt="image" src="https://github.com/parrotao/Vault_Windows_Admins_Password_Change/assets/37337484/db35aec0-fb9e-4cad-addd-5156140fd947">


# Running 

## 1. Run Sourcecode with the account have local adminstrator right

## 2. Run Sourcecode again and check the result

kv_v1 is for change status {"Pending","Changing","Changed"} 

* if you want to change the admin password, you can change the status to "Pending", After sucessful changed, the status value will be change to "Changed"

<img width="328" alt="image" src="https://github.com/parrotao/Vault_Windows_Admins_Password_Change/assets/37337484/bd80f1cd-b2ae-4fc0-a8ce-3e91a3aa7b79">

kv is for password store (you can check version for all history change)
<img width="592" alt="image" src="https://github.com/parrotao/Vault_Windows_Admins_Password_Change/assets/37337484/47155f6e-d565-49ef-8485-a546c578b920">


# Security considering

1. The sourcecode has been written by vbs for easy deployment from GPO

2. The vault URL and token is hard coding in the source code, so you shall review the user policy agan to ensure the user has no right to read KV storage.

3. The function returnpass can be replace any logical by yourselves.

4. User can use URL and Token in sourcecode to login, he can change status in kv_v1 and add new version of password to KV, but he can't get the list of history password.

5. Audit and other ACL control should be established.
