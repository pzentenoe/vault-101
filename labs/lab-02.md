# Vault basics

## Overview

In this lab, you will learn how to manage ACL Policies in Vault and use the encryption as a service feature of Vault.

## Duration

|Component       |Timing            |
|----------------|------------------|
|Introduction    |2 minutes         |
|Lab             |7 minutes         |
|Overview        |2 minutes         |

## What you need

To complete this lab, you need:

- Complete the previous lab.
- A computer with any of this operating system: MacOS, Linux or Windows.
- The latest version of Docker installed on that machine.
- The Vault CLI (you can find the installation instructions [here](https://www.vaultproject.io/docs/install/))

## Introduction

Everything in Vault is path based, and policies are no exception. Policies provide a declarative way to grant or forbid access to certain paths and operations in Vault. Policies are deny by default, so an empty policy grants no permission in the system.

In this lab we will play with policy workflows and syntaxes.

## Create and modify a policy

In this section we'll create a policy only with read permission, test it, and then modify the policy with more privileges testing the behaviour.

- Create a read-only policy and name it "dev-policy.hcl"

  ```bash
  export VAULT_TOKEN=my-root-token

  cat > dev-policy.hcl <<EOF
  path "secret/awesome-app" {
      capabilities = ["read"]
  }
  EOF

  vault policy write awesome-app-ro ./sample-policy.hcl
  ```

- Create a token with the associated policy and export that token

  ```bash
  vault token create -policy=awesome-app-ro
  export VAULT_TOKEN=[GENERATED_TOKEN]
  ```

- Try to create a secret with that token (shoud fail because that policy is read-only)

  ```bash
  vault kv put secret/awesome-app api_key=abc1234 api_secret=1a2b3c4d
  ```

- Update that policy with all permisions on the given path and sub-paths.

  ```bash
  cat > dev-policy.hcl <<EOF
  path "secret/awesome-app/*" {
    capabilities = ["list", "create", "update", "read", "delete"]
  }
  EOF

  vault policy write awesome-app-ro ./sample-policy.hcl
  ```

- Try to create a secret again (this time should success)

  ```bash
  vault kv put secret/awesome-app api_key=abc1234 api_secret=1a2b3c4d
  ```

## Encrypt a text string and decrypt it

In this section we'll encrypt a text using the transit engine (encrypt as a service) and then decript it.

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

### What you've learned

In this lab you:

- Create a simple policy and a token associated to this
- Update that policy with more permissions
- Encrypt and decrypt a plain text

### Search and further learning

- [Vault architecture](https://www.vaultproject.io/docs/internals/architecture.html)
