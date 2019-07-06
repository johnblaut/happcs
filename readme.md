## Introduction

The aim of the **HappsC** project is to provide a convenient and readily available setup for running Happs using Docker containers, that can facilitate quick deployment of code changes during testing and development, whilst still being suitable for use in production, thus ensuring consistency across all environments. Happs is based on Laravel which is a web application framework that attempts to take the pain out of development by easing common tasks used in most web projects. The application consists of two service components:

- **has**: Happs application service running on container: **hasc** which includes PHP and Composer for running Laravel
- **hdc**: Happs database service running on container: **hdsc** using MariaDB

This repository also conveniently references the [Happs repository](https://bitbucket.org/kryptonmlt/happs/src) at `services/has/happs/` via a submodule, meaning that when you clone this repository, the Happs code can also be automatically downloaded (requires using the --recursive option), thus avoiding the need to have to separately fetch the latter.

## Quick Start

On your machine, enter the directory where you intend to download this Git repository and execute the following:

```bash
git clone --recursive https://github.com/johnblaut/happsc.git
mv $APP_ENVIRONMENT.env .env
cp secrets/sample/.asc.env secrets/sample/.csc.env secrets/sample/.dsc.env $SEC_DIR/
vim $SEC_DIR/.asc.env
vim $SEC_DIR/.csc.env
vim $SEC_DIR/.dsc.env
docker-compose up
```
You should then be able to access the application at: http://localhost:8000 (or an alternative port if your configuration specifies otherwise)

## Configuration Reference

TBD

