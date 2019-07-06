## Introduction

The aim of the **HappsC** project is to provide a convenient and readily available setup for running Happs using Docker containers, that can facilitate quick deployment of code changes during testing and development, whilst still being suitable for use in production, thus ensuring consistency across all environments. Happs is based on Laravel which is a web application framework that attempts to take the pain out of development by easing common tasks used in most web projects. The application consists of two service components:

- `has` Happs application service running on container `hasc` which includes PHP and Composer for running Laravel
- `hdc` Happs database service running on container `hdsc` using MariaDB

## Overview

Configuration for each environment is maintained in this repository using files named: `<environment>.env` (e.g. `local.jb` for my own local environment, `production.env` for production, etc.). Therefore for any new environment one should create a similar file named accordingly and commit it to the repository. You may use the contents of an existing file as a template and then update the values accordingly. The name of the environment happens to also be used throughout the application to determine other settings accordingly. This is done via variable `$APP_ENVIRONMENT` stored in `<environment>.env` and its value should therefore be identical to `<environment>` i.e. consistent with the name of the configuration file for the given environment.

For security reasons, the configuration maintained in the repository is limited just to settings that do not involve any credentials or other sensitive details. Such secrets need to be populated in separate files that have to be maintained locally. These files should never be published to a repository! The repository however does includes a sample of such files at `secrets/sample/` which you can use as a template for the actual files stored locally on the given environment. Below is an overview of each of the required secrets files:

- `.asc.env` Application service related secrets
- `.dsc.env` Database service related secrets
- `.csc.env` Common secrets to both services

These files need to be always exactly named as described above. The application however allows you to configure the location of the directory storing these files, on a per environment basis - this is configured using the `$SEC_DIR` variable found in the `<environment>.env` configuration file. For security reasons, use restricted permissions for the files such as `600` and for the storing directory as well, such as `700`. Also recommended is to have the directory hidden using format: `/path/to/hidden/secrets/.directory/`

This repository also conveniently references the [Happs repository](https://bitbucket.org/kryptonmlt/happs/src) at `services/has/happs/` via a submodule, meaning that when you clone this repository, the Happs code can also be automatically downloaded from its own repository (requires using the `--recursive` option), thus avoiding the need to have to separately fetch the latter.

## Quick Start

On your machine, enter the directory where you intend to download this Git repository and execute the following:

```bash
git clone --recursive https://github.com/johnblaut/happsc.git
cp $APP_ENVIRONMENT.env .env
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
