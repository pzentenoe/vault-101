# Vault basics

## Overview

In this lab, you will prepare your Vault development environment and create your first secret using the Vault CLI (command line interface).

## Duration

|Component       |Timing            |
|----------------|------------------|
|Introduction    |5 minutes         |
|Lab             |10 minutes        |
|Overview        |5 minutes         |

## What you need

To complete this lab, you need:

- A computer with any of this operating system: MacOS, Linux or Windows.
- The latest version of Docker installed on that machine.
- The Vault CLI (you can find the installation instructions [here](https://www.vaultproject.io/docs/install/))

## Introduction

One of the core features of Vault is the ability to read and write arbitrary secrets securely. On this lab, we'll do this using the CLI and the HTTP API that can be used to programmatically do operations with Vault.

Secrets written to Vault are encrypted and then written to backend storage. For our dev server, backend storage is in-memory, but in production this would more likely be on disk or in Consul. Vault encrypts the value before it is ever handed to the storage driver. The backend storage mechanism never sees the unencrypted value and doesn't have the means necessary to decrypt it without Vault.

## Project setup

Deploy Vault using the Docker container in development mode in order to make this lab easier. This mode is not recomended for production, but is enough for the objectives of this lab.

1. Open a terminal window and then create a new Docker network for this lab:

    ```bash
    docker network create vault-net
    ```

2. Deploy Vault using the lates official Docker image:

    ```bash
    docker run --cap-add=IPC_LOCK \
      -e 'VAULT_DEV_ROOT_TOKEN_ID=my-root-token' \
      --name vault \
      --network vault-net \
      -p 8200:8200 \
      vault
    ```

3. Verify the CLI is working. On a second terminal windows do the following:

    ```bash
    export VAULT_ADDR=http://0.0.0.0:8200

    export VAULT_TOKEN=my-root-token

    vault status
    ```

If everything is OK, you'll see an output like this.

  ```bash
  Key             Value
  ---             -----
  Seal Type       shamir
  Initialized     true
  Sealed          false
  Total Shares    1
  Threshold       1
  Version         1.0.3
  Cluster Name    vault-cluster-8a5fec39
  Cluster ID      246c20df-2d29-7d18-a366-9785ee2588d3
  HA Enabled      false
  ```

In developer mode, we pass our root token when deploying the container so we can use it later. In a production environment, the root token is obtained when we unseal Vault (this is not covered in this Lab but you can find more info [here](https://www.vaultproject.io/docs/concepts/seal.html))

## Your first secret

In this section we'll create a secret using the key/value engine, this engine is usually the first and the most common common way to create secrets in Vault.

- Create a key/value secret

  ```bash
  vault kv put secret/awesome-app api_key=abc1234 api_secret=1a2b3c4d
  ```

- Getting that secret

  ```bash
  vault kv get secret/awesome-app
  ```

- Print only the value of a given field

  ```bash
  vault kv get -field=api_key secret/awesome-app
  ```

- Delete that secret

  ```bash
  vault kv delete secret/awesome-app
  ```

## Using the REST API

Here we will use the REST API to do Vault secret operations.

- First create a JSON file with the secret (name it payload.json)

  ```json
  {
    "data": {
      "api_key": "abc1234",
      "api_secret": "1a2b3c4d"
    }
  }
  ```

- Create the secret using curl

  ```bash
  curl \
    --header "X-Vault-Token: my-root-token" \
    --request POST \
    --data @payload.json \
    https://127.0.0.1:8200/v1/secret/data/awesome-app
  ```

- Get that secret

  ```bash
  curl \
    --header "X-Vault-Token: my-root-token" \
    https://127.0.0.1:8200/v1/secret/data/awesome-app
  ```