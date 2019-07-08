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
- _Also make sure to go through the Configuration Reference below_
- _The below commands can also be used as is on Windows if using Git Bash otherwise adapt them accordingly for the standard Windows GUI environment_

```bash
git clone --recursive https://github.com/johnblaut/happsc.git
cd happsc

# before proceeding with the next command, ensure your 'env/<environment>/' files are already present 
# and fetched from Git - if not and you are currently unable to submit changes to the Git repository,
# then for now manually create the 'env/<environment>/' directory directly on the host and within this
# directory add the expected '.env' and 'docker-compose.override.yml' configuration files, and have
# these updated accordingly for your environment
#
# in the next commands replace the #$APP_ENVIRONMENT# and #$SEC_DIR# placeholders with the actual values
# of the enclosed variables as defined in the corresponding 'env/<environment>/.env' file for your environment

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
Set this to the name of the environment e.g. production, test, staging, local.dev1, local.dev2, etc. If you are operating on a Windows environment, make sure the value of this variable includes the string `win`. The application makes use of this string to implement some additional settings required for when operating within a Windows environment e.g. local.jb.win

##### APP_VERSION
Set this to the application's version number. Currently using value `1.0` for this variable.

##### APP_HOME
Set this to the root directory within the container under which the application's whole directory structure will reside including the application code. Current set to `/opt/happs`

##### APP_OTF_DEPLOY
When this variable is defined, immediate on the fly code changes can be made to the code, without having to rebuild the image each time, thus making testing much easier for developers. Normally the code gets stored in the image under `/opt/happs/release`. Doing any changes to the contents of this directory, would each time result in having to rebuild the image in order to pick up such changes, since these files actually form part of the image and thus the image has to be updated accordingly whenever these change. However when this variable is defined, the application is invoked from a different directory that is `/opt/happs/otf`, which is actually mapped to a directory on the Docker host while the container is running. This means that if this mapped directory on the host contains the application files, any changes done to these files directly on the host, will be then be picked up immediatly in the container as well and thus such changes can be tested right away without having to rebuild the image. Of course, when using this feature, one must ensure that in the mapped directory all the neccessary files required for running the application are actually present. That being said, in case this directory is found to be empty and provided the environment is a local one, while all along the `APP_OTF_DEPLOY` variable remains defined, then the application will automatically make a copy of the packaged code in `/opt/happs/release` and place it on `/opt/happs/otf` which corresponds to the mapped host directory. Hence by mounting an empty directory, one can still conveniently test the code not without needing to actually make it available and also the starting contents of this directory will be an exact clone of the files included in the image (in `/opt/happs/release`), thus ensuring that the code being tested is the same as what is intended be used in production since in production it is meant to use the application code stored in the image. In order to make use of this feature, whereby code changes done locally on the host can be immediately visible on the container without needing to rebuild, the only requirement for the `APP_OTF_DEPLOY` variable is that it is set to some value, for example `enabled` would be a suitable value. When this feature is not be used, for example in production environments, then the variable should remain undefined, that is set to an empty string or else one can have it entirely omitted from the configuration.

##### APP_RESTART
This determines the restart policy of the application service container - `no` is the default restart policy, and it does not restart a container under any circumstance. When `always` is specified, the container always restarts e.g. can be used in environments where downtime needs to be avoided.

##### APP_PORT
The HTTP port that the Laravel PHP application will listen on. Usuaally this is port 8000, but one can have it changed accordingly. The port number cannot be less than 1024, since the application runs as a regular unprivileged user.

##### APP_UNAME
The name of the regular unpriveleged user that will run the application e.g. `happps` - root should be avoided and never used.

##### APP_GNAME
The group of the regular unpriveleged user that will run the application e.g. `happps`

##### APP_UID
The user id of the regular unpriveleged user that will run the application e.g. `1200`

##### APP_GID
The group id of the regular unpriveleged user that will run the application e.g. `1200`

##### APP_LOCAL_DIR
The path to  the local directory on the host that will map to the `/opt/happs/otf` directory on the container, that will allow changes done to the code on the host to be picked up immediately i.e. on the fly in the container. In Linux environments, this would be achieved via a bind mount, so it's important that the permissions of this directory are set correctly, for this to work. The application runs as an unprivileged user with a specific UID:GID as defined by the `APP_UID` and `APP_GID` variables mentioned just above. Hence the permissions of the local directory on the Linux host, in terms of UID and GID, need to be set up in such a way, that the unprivileged application user in the container can read the contents of the mapped host directory. Ideally one would avoid to make the directory world readable, so one could restrict the local host directory with `700` permissions and ensure it is owned by a local user having a matching UID to the one used inside the container, so that it can be read by the application, but not by other users (except root of course). For instance if one is using user `happs` with UID `1200` inside the container, then one can create an identical user on the Linux Docker host and have the mapped directory owned by this user. Strictly speaking, only the UID needs to match - the user name could be different but one might want to have them the same on both ends for consistency. In the case of Windows Docker hosts, the mounting of a mapped local directory is essentially done using a CIFS network share, so Docker on Windows will prompt for your local Windows credentials when the mapped directory is being mounted, in order to authenticate access for it - UID/GIDs are not a direct concern on the Windows host end.

##### MYSQL_USER
The name of DB user used by the application to access the database e.g. `happs`

##### MYSQL_DATABASE
The name of the database that the application will connect to e.g. `happs`

##### MYSQL_PORT
The listening port of the DB instance - usually `3306`

##### MYSQL_HOST
The database host. When using the included MariaDB container for the database service, this should always remain set to the name of the DB container i.e. `hdsc` which the containers are able to resolve internally. Only change this value in case for some reason you need to connect the application to some other external database.

##### MYSQL_RESTART
This determines the restart policy of the database service container - works in the same manner as described for `APP_RESTART` but applies to the DB container instead.

##### SEC_DIR
The path to the local directory on the Docker host where the configuration files storing any required credentials and secrets ( i.e. `.asc.env`, `.csc.env` and `.dsc.env` ) will reside. It is recommended to restrict this directory as much as possible e.g. `700` permissions and have it hidden i.e. name of directory would begin with a `.` - as for the three configuration files which are already hidden these should have restricted permissions as well e.g. `600`

### `docker-compose.override.yml`



### `.asc.env`

TBD

### `.csc.env`

TBD

### `.dsc.env`

TBD
