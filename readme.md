## Introduction

The aim of the **HappsC** project is to provide a convenient and readily available setup for running Happs using Docker containers, that can facilitate quick deployment of code changes during testing and development, whilst still being suitable for use in production, thus ensuring consistency across all environments. Happs is based on Laravel which is a web application framework that attempts to take the pain out of development by easing common tasks used in most web projects.

## Overview

On Docker, the application is set up to consist of two service components:

- `has` Happs application service running on container `hasc` which includes PHP and Composer for running Laravel
- `hdc` Happs database service running on container `hdsc` using MariaDB

Configuration for each environment is maintained in this Git repository using a file named: `.env` found in directory `env/<environment>/` (e.g. `env/local.jb/` for my own local environment, `env/production/` for production, etc.). Therefore for any new environment one should create a similar `env/<environment>` directory named after the environment, containing an `.env` file updated accordingly and commit it to the repository.  For a specific environment's `.env` file you may use the contents of an existing `.env` file from  another environment as a template and then update the values accordingly. The name of the environment happens to also be used throughout the application to determine other settings and conditions accordingly. This is done via variable `$APP_ENVIRONMENT` stored in the `.env` file of the given environment and its value should therefore be identical to `<environment>` i.e. consistent with the name of the directory under `env/` storing the configuration for that given environment.

Additionally, one should know that by default, Docker Compose actually reads two files, a `docker-compose.yml` and an optional `docker-compose.override.yml` file. By convention, the `docker-compose.yml` contains the base configuration. The override file, as its name implies, can contain configuration overrides for existing services or entirely new services. Hence for each environment, such a `docker-compose.override.yml` file is also maintained under `env/<environment>/` to cater for any particular Docker Compose configuration overrides that may be required for that given environment. Thus for any new environment one should also add a `docker-compose.override.yml` under `env/<environment>/`, containing any required configuration overrides and have it committed to the repository.

For security reasons, the configuration maintained in the repository is limited just to settings that do not involve any credentials, secrets or other sensitive details. Such secrets instead need to be populated in separate files that have to be maintained locally. These files should never be published to a repository! The repository however does includes a sample of such files at `secrets/sample/` which you can use as a template for the actual files stored locally on the given environment. Below is an overview of each of the required secrets files:

- `.asc.env` Application service related secrets
- `.dsc.env` Database service related secrets
- `.csc.env` Common secrets to both services

These files need to be always exactly named as described above. The application however allows you to configure the location of the directory storing these files, on a per environment basis - this is configured using the `$SEC_DIR` variable found in the `<environment>/.env` configuration file. For security reasons, use restricted permissions for the files such as `600` and for the storing directory as well such as `700`. Also recommended is to have this as a hidden directory.

This repository also conveniently references the [Happs repository](https://bitbucket.org/kryptonmlt/happs/src) at `services/has/happs/` via a submodule, meaning that when you clone this repository, the Happs code can also be automatically downloaded from its own repository (requires using the `--recursive` option), thus avoiding the need to have to separately fetch the latter.

As for the DB service container, a named volume `hdsv` is being used for mounting the data directory on the container at `/var/lib/mysql`. This way the data can persist, regardless of whether the container is running or not.


## Requirements

- Git
- Docker


## Run Instructions

On your machine, enter the parent directory where you intend to download this Git repository and execute the following commands:

#### Notes:

- _Ensure first to read the above Overiew section so that the purpose of all the commands below is clear_
- _Also make sure to go through the Configuration Reference further below_
- _The below commands can also be used as is on Windows if using Git Bash, otherwise adapt them accordingly for the standard Windows GUI environment_

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
# of the enclosed variables, as defined in the corresponding 'env/<environment>/.env' file for your environment
#
# the '.env' and 'docker-compose.override.yml' files for the selected environment need to be in the same working
# directory alongside the main 'docker-compose.yml' file in order to be able to manage the containers with
# the 'docker-compose' command - this can be acheived with the two symlinks below, which point to these 
# two files for the selected environment, by replacing the #$APP_ENVIRONMENT# placeholder accordingly 
# e.g. for 'production', these two files would be found under 'env/production/', so the symlinks should point to the
# files in this location, such that these files can also be referenced as if they were in the current working directory
# as required and expected by the 'docker-compose' command

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
This should be set to the desired version of PHP to run. The application container makes use of an official PHP image (CLI only in this case) as its base. When building the application container, the value of this variable will be used to fetch the PHP image that has a tag equal to this value. Since the PHP images are tagged by version number, the container will therefore run the PHP version as defined by this variable. Currently the value being used is `7.1` meaning that the container is using the latest PHP 7.1 image as its base. Update this value accordingly in order to change the PHP version (making sure the new value is an actual available tag).

##### MDB_TAG
This should be set to the desired version of MariaDB to run. The DB container makes direct use of an official MariaDB image as is. When building the DB container, the value of this variable will be used to fetch the MariaDB image that has a tag equal to this value. Since also the MariaDB images are tagged by version number, the container will therefore run the MariaDB version as defined by this variable. Currently the value being used is `10.2.8`. Update this value accordingly in order to change the MariaB version (making sure the new value is an actual available tag).

##### APP_ENVIRONMENT
Set this to the name of the environment e.g. `production`, `test`, `staging`, `local.dev1`, `local.dev2`, etc. If you are operating on a Windows environment, make sure the value of this variable includes the string `win` e.g. `local.jb.win`. The application makes use of this string to implement additional measures required for when operating within a Windows environment such as, when copying files to a mapped directory mounted from a Windows host, there could be issues when encountering symlinks, so by checking for this string the application in such cases will convert the symlinks to actual files during the copying process in order to avoid issues of this sort. Refer to the below documentation for `APP_OTF_DEPLOY` for more information regarding the purpose of this copy operation to a mapped directory.

##### APP_VERSION
Set this to the application's version number. Currently using value `1.0` for this variable. Together with `APP_ENVIRONMENT`, this variable is also used to form the tag for the application service container image. The format of the tag is: `$APP_VERSION-$APP_ENVIRONMENT` e.g. `1.0-production`

##### APP_HOME
Set this to the root directory within the container under which the application's whole directory structure will reside including the application code. Currently set to `/opt/happs`

##### APP_OTF_DEPLOY
When this variable is defined, immediate on the fly code changes can be made to the code, without having to rebuild the image each time, thus making testing much easier for developers. Normally the code gets stored in the image under `/opt/happs/release`. Doing any changes to the contents of this directory, would each time result in having to rebuild the image in order to pick up such changes, since these files actually form part of the image and thus the image has to be updated accordingly whenever these change. However when this variable is defined, the application is invoked from a different directory that is `/opt/happs/otf`, which is actually mapped to a directory on the Docker host while the container is running. This means that if this mapped directory on the host contains the application files, any changes done to these files directly on the host, will be then be picked up immediatly in the container as well and thus such changes can be tested right away without having to rebuild the image. Of course, when using this feature, one must ensure that in the mapped directory all the neccessary files required for running the application are actually present ( for instance the required `/vendor` files are not readily provided in the `happs` repository but are dynamically generated when invoking `composer update` and need to be in place for the application to work as intended ). That being said, in case this directory is found to be empty and provided the environment is a local one, while all along the `APP_OTF_DEPLOY` variable remains defined, then the application will automatically make an identical copy of the packaged code in `/opt/happs/release` and place it on `/opt/happs/otf` which corresponds to the mapped host directory ( this would be the copy operation that is made reference to in the above documentation for `APP_ENVIRONMENT` ). Hence by mounting an empty directory, one can still conveniently test the code, without needing to actually make it available and also the starting contents of this directory will be an exact clone of the files included in the image ( i.e. in `/opt/happs/release` ), thus ensuring that the code being tested is the same as what is intended be used in production since in production it is only meant to use the application code stored in the image. In order to make use of this feature, whereby code changes done locally on the host can be immediately visible on the container without needing to rebuild, the only requirement for the `APP_OTF_DEPLOY` variable is that it is set to some value, for example `enabled` would be a suitable value. When this feature is not be used, for example in production environments, then the variable should remain undefined, that is set to an empty string or else one can have it entirely omitted from the configuration.

##### APP_RESTART
This determines the restart policy of the application service container - `no` is the default restart policy, and it does not restart a container under any circumstance. When `always` is specified, the container always restarts e.g. can be useful in environments where downtime needs to be avoided.

##### APP_PORT
The HTTP port that the Laravel PHP application will listen on. Usuaally this is port 8000, but one can have it changed accordingly. The port number cannot be less than 1024, since the application runs as a regular unprivileged user.

##### APP_UNAME
The name of the regular unpriveleged user that will run the application e.g. `happps` - root should be avoided and never used for running the application.

##### APP_GNAME
The group of the regular unpriveleged user that will run the application e.g. `happps`

##### APP_UID
The user id of the regular unpriveleged user that will run the application e.g. `1200`

##### APP_GID
The group id of the regular unpriveleged user that will run the application e.g. `1200`

##### APP_LOCAL_DIR
This is the path to  the local directory on the host that will map to the `/opt/happs/otf` directory on the container, that will allow changes done to the code on the host to be picked up immediately i.e. on the fly in the container. In Linux environments, this would be achieved via a bind mount, so it's important that the permissions of this directory are set correctly, for this to work. The application runs as an unprivileged user with a specific UID:GID as defined by the `APP_UID` and `APP_GID` variables mentioned just above. Hence the permissions of the local directory on the Linux host, in terms of UID and GID, need to be set up in such a way, that the unprivileged application user in the container can read the contents of the mapped host directory. Ideally one would avoid to make the directory world readable, so one could restrict the local host directory with `700` permissions and ensure it is owned by a local user having a matching UID to the one used inside the container, so that it can be read by the application, but not by other users (except root of course). For instance if one is using user `happs` with UID `1200` inside the container, then one can create an identical user on the Linux Docker host and have the mapped directory owned by this user. Strictly speaking, only the UID needs to match - the user name could be different but one might want to have them the same on both ends for consistency. In the case of Windows Docker hosts, the mounting of a mapped local directory is essentially done using a CIFS network share, so Docker on Windows will prompt for your local Windows credentials when the mapped directory is being mounted, in order to authenticate access towards it - UID/GIDs are not a direct concern in the case of Windows hosts.

##### MYSQL_USER
The name of DB user used by the application to access the database e.g. `happs`

##### MYSQL_DATABASE
The name of the database that the application will use.g. `happs`

##### MYSQL_PORT
The listening port of the DB instance - usually `3306`

##### MYSQL_HOST
The database host. When using the included MariaDB container for the database service, this should always remain set to the name of the DB container i.e. `hdsc` which the containers are able to resolve internally. Only change this value in case for some reason you need to connect the application to some other external database.

##### MYSQL_RESTART
This determines the restart policy of the database service container - works in the same manner as described for `APP_RESTART` but applies to the DB container instead.

##### SEC_DIR
The path to the local directory on the Docker host where the configuration files storing any required credentials and secrets ( i.e. `.asc.env`, `.csc.env` and `.dsc.env` ) will reside. It is recommended to restrict this directory as much as possible e.g. `700` permissions and have it hidden i.e. name of directory would begin with a `.` - as for the three configuration files which are already hidden these are recommended to have restricted permissions as well e.g. `600`


### `docker-compose.override.yml`

In this setup, this configuration file is available on a per environment basis and for each environment it can be found under `env/<environment>/`. By convention, Docker Compose reads the  `docker-compose.yml` file for the base configuration of the application, while if available, the `docker-compose.override.yml` is read for any specific configuration overrides that may need to be applied - so in the case of this setup, these overrides would be specific for the given environment being deployed. In most cases this `docker-compose.override.yml` file is practically empty ( e.g. see `env/production/docker-compose.override.yml` ) as the base `docker-compose.yml` configuration alone is sufficient. A specific case where overrides are required is when one wants to be able to apply on the fly changes to the code on a local environment, which as mentioned before is enabled via the `APP_OTF_DEPLOY` variable and involves mounting a local host directory as a mapped directory on the container. For this mount to take place, a `volumes` entry needs to be present in the Docker Compose configuration. Therefore for environments where this feature may want to be used, such as on local environments, the `docker-compose.override.yml` will contain the extra configuration needed for mounting the mapped host directory ( see `env/local.jb/docker-compose.override.yml` and `env/local.jb.win/docker-compose.override.yml` ), in addition to the base configuration provided in `docker-compose.yml`. For other environments, such as production, this feature is not planned to be used, as in such environments the intention is to use the code already packaged in the image - thus the `docker-compose.override.yml` file for these environments is left virtually empty, so that no additional configuration other than the base configuration gets applied.


## Secrets Management

As mentioned before, secrets and other sensitive credentials are mantainined only locally, in three files ( i.e. `.asc.env`, `.csc.env` and `.dsc.env` ) residing in the directory as defined by the `SEC_DIR` configuration variable. These secrets need to be referenced by the Laravel application and normally it would expect to find them in a `.env file` in its own application directory ( not to be confused with the other `.env` configuration file found in `env/<environment>/` which is referenced by Docker Compose after applying the symlinks mentioned in the _Run Instructions_ section ). For security purposes, the original application .env file that is copied to the container image ( found in Git at `services/has/conf/.env` ), is a generic one populated with a placeholder `_SECRET_` string for any secrets and credentials related variables. This way one ensures that no sensitive details get included permanently in the image. The actual secrets values only get written to this `.env` file during run time thanks to the entrypoint script ( found in Git at `services/has/bin/happs_init.sh` ) invoked on container startup, which will replace the `_SECRET_` placeholder strings with the actual values defined in the local secrets files ( i.e. `.asc.env`, `.csc.env` and `.dsc.env` ). 

Worth noting is that the [Laravel documentation](https://laravel.com/docs/5.8/configuration) mentions that shell environment variables of the same name will override similar variables found in the application's `.env` file. At the same time though the same documentation mentions that all of the variables listed in the `.env` file will be loaded when the application receives a request, which therefore means that one can update these variables by either updating them in the `.env` file or else by overriding them with a new value in the shell environment. Both methods would require the user to perform operations from within the container, however updating the `.env` file can be done in a much easier manner, when 'on the fly' updates are enabled ( done via variable `APP_OTF_DEPLOY` ) as the user can then edit the file directly from the Docker host without needing to operate from within the container, since the application files, including this `.env` file are actually in a mapped directory mounted from the Docker host. For this reason it was preferred to have the variables read from the application `.env` file rather than from the shell, as the `.env` file can be accessed more easily when 'on the fly' updates are enabled. This meant that the initial variables used to define these secret values had to be given a different name then the actual variable names found in the application `.env` file, since if they shared the same name, the initial variables read from the shell would override the ones in the file, meaning that any future updates for these values done during runtime through the file would never take any effect, as the initial shall variables would retain priority ( e.g. with reference to the `services/has/bin/happs_init.sh` entry point script, one can see that the variable APP_KEY found in the file takes its value from shell variable APPKEY not APP_KEY - the same can be said for DB_PASSWORD which takes its value from MYSQL_PASSWORD rather than DB_PASSWORD - this way when say APP_KEY is updated in the file, that update can be picked up the application, since there is no overriding APP_KEY variable on the shell, as the shell variable being used to set the file variable is a different one i.e. it is APPKEY rather than APP_KEY ).

While at it, this mechanism is also being used for other configurable variables in the application `.env` file besides secrets. For such settings the initial placeholder value is `_INHERIT_`, and while in the case of the secrets the actual values were being taken from the `.asc.env`, `.csc.env` and `.dsc.env` files, in the case of these other settings, the actual values are being taken from the enviroment's `.env` file found in Git under `env/<environment>`. In this way, for a given environment, the intended values for all configurable variables selected thus far are therefore managed from a single `.env` file, i.e. just the 'outer' `.env` file found under `env/<environment>` which is referenced by Docker Compose ( via the symlinks mentioned in the  _Run Instructions_ section ) and thus there is no need to do any changes to the application `.env` file, which therefore is packaged inside the container in a generic form in terms of `_SECRET_` and `_INHERIT_` placeholders. The application `.env` file would only be edited, when one wants to test configuration changes immediately, which is only possible when on the fly updates are enabled ( i.e. when variable `APP_OTF_DEPLOY` is defined ). For normal operation however, this 'inner' application `.env` file is not expected to be maintained by the user - the only `.env` file managed by the user is the 'outer' one and is maintained in Git under `env/<environment>/` in order to permanently and conveniently keep track of all required configuration for all managed environments, excluding secrets and credentials of course which as explained for security reasons are maintained only locally in files: `.asc.env`, `.csc.env` and `.dsc.env`

### `.asc.env`
_Application service related secrets_

##### APPKEY
This is used by the application as an encryption key and needs to be a random 32 character string in base64 format i.e. `base64:<random_string>` - such a string can be generated using the following command: `echo "base64:$(openssl rand -base64 32)"` e.g. `base64:tHQ5PhiAHZKaKMjXYnAbHkQIFtYHVqv8eYyWngwrPJE=`

##### REDISPW
Redis credentials. Currently not used and set to null.

##### MAILPW
This is the SMTP password in case the SMTP service being used requires authentication. Currently not used and set to null.

##### PUSHKEY
Key credentials for the Pusher broadcast service. Currently not used and set to an empty string.

##### PUSHSEC
Secret credentials for the Pusher broadcast service. Currently not used and set to an empty string.


### `.dsc.env`
_Database service related secrets_

#### MYSQL_ROOT_PASSWORD
The password for the DB root user.


### `.csc.env`
_Common secrets to both services_

#### MYSQL_PASSWORD
The password for the DB user used by the application.

