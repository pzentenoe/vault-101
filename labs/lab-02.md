# Vault ACL policies and encryption as a service

## Overview

In this lab, you will learn how to manage ACL Policies in Vault and use the encryption as a service feature of Vault.

## Duration

|Component       |Timing            |
|----------------|------------------|
|Introduction    |3 minutes         |
|Lab             |7 minutes         |
|Overview        |2 minutes         |

## What you need

To complete this lab, you need:

- To have completed the previous lab.
- A computer with any of these operating systems: macOS, Linux or Windows.
- Deploy Vault using the latest official Docker image installed on that machine.
- The Vault CLI (you can find the installation instructions [here](https://www.vaultproject.io/docs/install/))
- To save time, please download the Vault and MySQL image **before** the workshop.

  ```bash
  docker pull vault
  docker pull mysql/mysql-server
  ```

## Introduction

Everything in Vault is path based, and policies are no exception. Policies provide a declarative way to grant or forbid access to certain paths and operations in Vault. Policies are *deny by default*, so an empty policy grants no permission in the system.

In this lab we will play with policy workflows and syntaxes and then we'll use the transit engine to encrypt a plain text.

## Create and modify a policy

In this section, we'll create a policy only with read permission only, test it, and then modify the policy with more privileges.

- Create a create and update policy with the name "dev-policy"

  ```bash
  export VAULT_ADDR=http://0.0.0.0:8200
  export VAULT_TOKEN=my-root-token

  echo 'path "secret/*" {
      capabilities = ["create", "update"]
  }' | vault policy write dev-policy -
  ```

- To see the list of policies, run:

  ```bash
  vault policy list
  ```

- To view the content of a policy, run:

  ```bash
  vault policy read dev-policy
  ```

- Create a token with the associated policy and export that token

  ```bash
  vault token create -policy=dev-policy
  export VAULT_TOKEN=[GENERATED_TOKEN]
  ```

- Try to create a secret with that token (shoud fail because that policy can't read secrets)

  ```bash
  vault kv put secret/awesome-app api_key=abc1234 api_secret=1a2b3c4d
  ```

- Try to get the content of the secret (should fail)

  ```bash
  vault kv get secret/awesome-app
  ```

- Update that policy with read permision on the given path

  ```bash
  echo 'path "secret/*" {
    capabilities = ["create", "update", "read"]
  }' | vault policy write dev-policy -
  ```

- Create a new token with the associated policy

  ```bash
  vault token create -policy=dev-policy
  export VAULT_TOKEN=[GENERATED_TOKEN]
  ```

- Try to get the content of the secret again

  ```bash
  vault kv get secret/awesome-app
  ```

## Encrypt a text string and decrypt it

In this section, we'll encrypt a text using the transit engine (encrypt as a service), that text will not be written into Vault but we can get the text back decrypting it.

- Enable transit engine

  ```bash
  vault secrets enable transit
  ```

- Create a named encryption key (usually each application have its own encryption key)

  ```bash
  vault write -f transit/keys/my-key
  ```

- Encryt some plain text using the encrypt endpoint (all plain text must be base64 encoded)

  ```bash
  vault write transit/encrypt/my-key plaintext=$(echo "super secret message" | base64)

  Key           Value
  ---           -----
  ciphertext    vault:v1:8SDd3WHDOjf7mq69CyCqYjBXAQQAVZRkFM13ok481zoCmHnSeDX9vyf7w==
  ```

- Decrypt that data using the decrypt endpoint

  ```bash
  vault write transit/decrypt/my-key ciphertext=vault:v1:8SDd3WHDOjf7mq69CyCqYBXAiQQAVZRkFM13ok481zoCmHnSeDX9vyf7w==

  Key          Value
  ---          -----
  plaintext    c3VwZXIgc2VjcmV0IG1lc3NhZ2UK
  ```

- Then decode the plaintext to obtain the source text before encryption

  ```bash
  echo "c3VwZXIgc2VjcmV0IG1lc3NhZ2UK" | base64 --decode
  ```

## Lab review

### In this lab, you've learned how to

In this lab you:

- Create a simple policy and a token associated to this
- Update that policy with more permissions
- Encrypt and decrypt a plain text

Now you can go to [Vault dynamic secrets](https://github.com/walmartdigital/vault-101/blob/master/labs/lab-03.md)

### Further learning

- [Policies in Vault](https://www.hashicorp.com/resources/policies-vault)
- [Transit Secrets Engine](https://www.vaultproject.io/docs/secrets/transit/index.html)
