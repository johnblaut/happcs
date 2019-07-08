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

On your machine, enter the parent directory where you intend to download this Git repository and execute the following commands:

#### Notes:

- _Ensure first to read the above Overiew section so that all the commands below are clearer_
- _Any strings in the steps below starting and ending with a #, are simply just indicative placeholders that are intended for you to replace with the actual values of the enclosed variable names between the # characters, when you are submitting these commands on the shell. The value of these variables should therefore match with what has been set in the `env/<environment>/.env` configuration file. These are not predefined environment variables on the host that will be interpreted automatically by the shell, unless you wish to actually declare and export these yourself in advance. That should work as well if you wish to do so - of course in that case omit the # characters.
- _As explained in the overvivew above, configuration is maintained in Git on a per environment basis in `env/<environment>/` directories. Hence the below steps assume that such configuration is already present in Git and will be fetched when cloning the Git repository. If however at this time you are not able to submit and commit changes to the Git repository, in that exceptional case, after you enter the cloned `happsc` directory and before you run the first `ln` symlink command, create your own `env/<environment>/` directory directly on the host and within it add the expected `.env` and `docker-compose.override.yml` configuration files, updated accordingly for your environment.
- _The below commands can also be used as is on Windows if using Git Bash_

```bash
git clone --recursive https://github.com/johnblaut/happsc.git
cd happsc
ln -s env/#$APP_ENVIRONMENT#/.env
ln -s env/#$APP_ENVIRONMENT#/docker-compose.override.yml
cp secrets/sample/.asc.env secrets/sample/.csc.env secrets/sample/.dsc.env #$SEC_DIR#/
vim #$SEC_DIR#/.asc.env
vim #$SEC_DIR#/.csc.env
vim #$SEC_DIR#/.dsc.env
docker-compose up
```
You should then be able to access the application at: http://localhost:8000 (or an alternative port if your configuration specifies otherwise)

## Configuration Reference

### `.env`

##### PHP_TAG
This should be set to the desired version of PHP to run. The application container makes use of an official PHP image (CLI only in this case) as its base. When building the application container, the value of this variable will be used to fetch the PHP image that has a tag equal to this value. Since the PHP images are tagged by version number, the container will therefore run the PHP version as defined by this variable. Currently the value being used is `7.1` meaning that the container be using the latest PHP 7.1 image as its base. Update this value accordingly in order to change the PHP version (make sure the new value is an actual available tag).

##### MDB_TAG
This should be set to the desired version of MariaDB to run. The DB container makes direct use of an official MariaDB image as is. When building the DB container, the value of this variable will be used to fetch the MariaDB image that has a tag equal to this value. Since also the MariaDB images are tagged by version number, the container will therefore run the MariaDB version as defined by this variable. Currently the value being used is `10.2.8`. Update this value accordingly in order to change the MariaB version (making sure the new value is an actual available tag).

##### APP_ENVIRONMENT
Set this to the name of the environment e.g. production, test, staging, local.dev1, local.dev2, etc.

##### APP_VERSION
Set this to the application's version number. Currently using value `1.0` for this variable.

##### APP_HOME
Set this to the root directory within the container under which the application's whole directory structure will reside including the application code. Current set to `/opt/happs`

##### APP_OTF_DEPLOY
When this variable is defined, immediate on the fly code changes can be made to the code, without having to rebuild the image each time, thus making testing much easier for developers. Normally the code gets stored in the image under `/opt/happs/release`. Doing any changes to the contents of this directory, would each time result in having to rebuild the image in order to pick up such changes, since these files actually form part of the image and thus the image has to be updated accordingly whenever these change. However when this variable is defined, the application is invoked from a different directory that is `/opt/happs/otf`, which is actually mapped to a directory on the Docker host while the container is running. This means that if this mapped directory on the host contains the application files, any changes done to these files directly on the host, will be then be picked up immediatly in the container as well and thus such changes can be tested right away without having to rebuild the image. Of course, when using this feature, one must ensure that in the mapped directory all the neccessary files required for running the application are actually present. That being said, in case this directory is found to be empty and provided the environment is a local one, while all along the `APP_OTF_DEPLOY` variable remains defined, then the application will automatically make a copy of the packaged code in `/opt/happs/release` and place it on `/opt/happs/otf` which corresponds to the mapped host directory. Hence by mounting an empty directory, one can still conveniently test the code not without needing to actually make it available and also the starting contents of this directory will be an exact clone of the files included in the image (in `/opt/happs/release`), thus ensuring that the code being tested is the same as what is intended be used in production since in production it is meant to use the application code stored in the image. In order to make use of this feature, whereby code changes done locally on the host can be immediately visible on the container without needing to rebuild, the only requirement for the `APP_OTF_DEPLOY` variable is that it is set to some value, for example `enabled` would be a suitable value. When this feature is not be used, for example in production environments, then the variable should remain undefined, that is set to an empty string or else one can have it entirely omitted from the configuration.

##### APP_RESTART
This determines the restart policy of the containers.


##### APP_PORT


##### APP_UNAME


##### APP_GNAME


##### APP_UID


##### APP_GID


##### APP_LOCAL_DIR


##### MYSQL_USER


##### MYSQL_DATABASE


##### MYSQL_PORT


##### MYSQL_HOST


##### MYSQL_RESTART


##### SEC_DIR


### `docker-compose.override.yml`



### `.asc.env`

TBD

### `.csc.env`

TBD

### `.dsc.env`

TBD
