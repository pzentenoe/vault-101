# Vault dynamic secrets

## Overview

In this lab, you will learn how to create a MySQL credential dynamically when your app need it.

## Duration

|Component       |Timing            |
|----------------|------------------|
|Introduction    |3 minutes         |
|Lab             |9 minutes         |
|Overview        |2 minutes         |

## What you need

To complete this lab, you need:

- Ideally have done the previous labs.
- A computer with any of these operating systems: macOS, Linux or Windows.
- Deploy Vault using the latest official Docker image installed on that machine.
- The Vault CLI (you can find the installation instructions [here](https://www.vaultproject.io/docs/install/))
- To save time, please download the Vault and MySQL image **before** the workshop.

  ```bash
  docker pull vault
  docker pull mysql/mysql-server
  ```

## Introduction

Unlike the kv secrets where you had to put data into the store yourself, dynamic secrets are generated when they are accessed. Dynamic secrets do not exist until they are read, so there is no risk of someone stealing them or another client using the same secrets. Because Vault has built-in revocation mechanisms, dynamic secrets can be revoked immediately after use, minimizing the amount of time the secret existed.

In this lab we will deploy a MySQL instance in our Docker network, then through the database secret engine we'll create a database credential dynamically to be used by a client.

## Deploy MySQL on Docker

In this section we'll deploy a Docker container with a MySQL database.

- Deploy MySQL container

  ```bash
  docker pull mysql/mysql-server
  docker run --name=mysql -d --network vault-net mysql/mysql-server
  ```

- After a few seconds execute this command to obtain the root password

  ```bash
  docker logs mysql 2>&1 | grep GENERATED

  [Entrypoint] GENERATED ROOT PASSWORD: d@cAd[ytuB6@qEd(EDAr3g%OcKI
  ```

- Connect to MySQL using the root password

  ```bash
  docker exec -it mysql mysql -uroot -p
  ```

- Change the root password

  ```sql
  alter user 'root'@'localhost' identified by 'passw0rd!';
  ```

- Create a database and a database user

  ```sql
  create database marvel character set utf8 collate utf8_general_ci;
  create user 'nick' identified by 'password';
  grant all privileges on *.* to 'nick'@'%' with grant option;
  flush privileges;
  ```

- Create a table and some data

  ```sql
  use marvel;
  create table characters(id int auto_increment, name varchar(255) not null, age int, primary key(id));
  insert into characters(name, age) values ('Spider-man', 21);
  insert into characters(name, age) values ('Captain America', 100);
  insert into characters(name, age) values ('Vision', 1);
  exit;
  ```

- Enable the database secret engine

  ```bash
  export VAULT_ADDR=http://0.0.0.0:8200
  export VAULT_TOKEN=my-root-token
  vault secrets enable database
  ```

- Configure the engine with the previously created database user

  ```bash
  vault write database/config/marvel \
    plugin_name=mysql-database-plugin \
    connection_url="{{username}}:{{password}}@tcp(mysql:3306)/marvel" \
    allowed_roles="nick-fury" \
    username="nick" \
    password="password"
  ```

- Create a Vault database role for creating dynamically a credential with select permissions on the database, also have an expiration time

  ```bash
  vault write database/roles/nick-fury \
    db_name=marvel \
    creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}';GRANT SELECT ON marvel.* TO '{{name}}'@'%';" \
    default_ttl="1h" \
    max_ttl="24h"
  ```

- Get the dynamically created credentials

  ```bash
  vault read database/creds/nick-fury

  Key                Value
  ---                -----
  lease_id           database/creds/nick-fury/t5ZSRWuYefrOqFOTAAT4AgQ9
  lease_duration     1h
  lease_renewable    true
  password           A1a-KrSuX8oylV8mdRcc
  username           v-token-nick-fury-syGxj2R1qWZLbn
  ```

- Connect to MySQL using that credentials

  ```bash
  docker exec -it mysql mysql -uv-token-nick-fury-syGxj2R1qWZLbn -p
  ```

- Get previously created records

  ```sql
  use marvel;
  select * from characters;
  ```

- Try to add a new record with this user

  ```sql
  insert into characters(name, age) values ('Hulk', 40);
  ```

This shoud fail because that user only have select permission on that database.

## Lab review

### In this lab, you've learned how to

In this lab you:

- Deploy a MySQL database with a privileged user
- Enable and configure the database secret engine
- Create dynamically a user and then use it to access a database

### Further learning

- [Vault database secret engine](https://www.vaultproject.io/docs/secrets/databases/index.html)
