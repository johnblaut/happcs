## Introduction

The aim of the **HappsC** project is to provide a convenient and readily available setup for running Happs using Docker containers, that can facilitate quick deployment of code changes during testing and development, whilst still being suitable for use in production, thus ensuring consistency across all environments. Happs is based on Laravel which is a web application framework that attempts to take the pain out of development by easing common tasks used in most web projects.

## Overview

On Docker, the application is set up to consist of two service components:

- `has` Happs application service running on container `hasc` which includes PHP and Composer for running Laravel
- `hdc` Happs database service running on container `hdsc` using MariaDB

Configuration for each environment is maintained in this repository using a file named: `.env` found in directory `env/<environment>/` (e.g. `env/local.jb/` for my own local environment, `env/production/` for production, etc.). Therefore for any new environment one should create a similar `env/<environment>` directory named accordingly, containing an `.env` file and commit it to the repository.  For a specific environment's `.env` file you may use the contents of an existing `.env` file from  another environment as a template and then update the values accordingly. The name of the environment happens to also be used throughout the application to determine other settings and conditions accordingly. This is done via variable `$APP_ENVIRONMENT` stored in tne `.env` file of the given environment and its value should therefore be identical to `<environment>` i.e. consistent with the name of the directory under `env/` storing the configuration for that given environment.

Additionally, one should know that by default, Docker Compose actually reads two files, a `docker-compose.yml` and an optional `docker-compose.override.yml file`. By convention, the `docker-compose.yml` contains the base configuration. The override file, as its name implies, can contain configuration overrides for existing services or entirely new services. Hence for each environment, such a `docker-compose.override.yml` file is also maintained under `env/<environment>/` to cater for any particular Docker Compose configuration overrides that are required for a given environment. Thus for any new environment one should also add a `docker-compose.override.yml` under `env/<environment>/`, containing any required configuration overrides and have it committed to the repository.

For security reasons, the configuration maintained in the repository is limited just to settings that do not involve any credentials or other sensitive details. Such secrets need to be populated in separate files that have to be maintained locally. These files should never be published to a repository! The repository however does includes a sample of such files at `secrets/sample/` which you can use as a template for the actual files stored locally on the given environment. Below is an overview of each of the required secrets files:

- `.asc.env` Application service related secrets
- `.dsc.env` Database service related secrets
- `.csc.env` Common secrets to both services

These files need to be always exactly named as described above. The application however allows you to configure the location of the directory storing these files, on a per environment basis - this is configured using the `$SEC_DIR` variable found in the `<environment>.env` configuration file. For security reasons, use restricted permissions for the files such as `600` and for the storing directory as well, such as `700`. Also recommended is to have the directory hidden using format: `/path/to/hidden/secrets/.directory/`

This repository also conveniently references the [Happs repository](https://bitbucket.org/kryptonmlt/happs/src) at `services/has/happs/` via a submodule, meaning that when you clone this repository, the Happs code can also be automatically downloaded from its own repository (requires using the `--recursive` option), thus avoiding the need to have to separately fetch the latter.

## Quick Start

On your machine, enter the parent directory where you intend to download this Git repository and execute the following:

```bash
git clone --recursive https://github.com/johnblaut/happsc.git
cd happsc
ln -s env/$APP_ENVIRONMENT/.env
ln -s env/$APP_ENVIRONMENT/.env/docker-compose.override.yml
cp secrets/sample/.asc.env secrets/sample/.csc.env secrets/sample/.dsc.env $SEC_DIR/
vim $SEC_DIR/.asc.env
vim $SEC_DIR/.csc.env
vim $SEC_DIR/.dsc.env
docker-compose up
```
You should then be able to access the application at: http://localhost:8000 (or an alternative port if your configuration specifies otherwise)

## Configuration Reference

#### `<environment>.env`

TBD

#### `.asc.env`

TBD

#### `.csc.env`

TBD

#### `.dsc.env`

TBD
