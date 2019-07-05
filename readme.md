## Introduction

The aim of the HappsC project is to provide a convenient and readily available setup for running Happs using containers.
Happs is based on Laravel which is a web application framework that attempts to take the pain out of development by easing common tasks used in most web projects.
The application consists of two service components:

- has: Happs application service running on container: hasc which includes PHP and Composer for running Laravel
- hdc: Happs database service running on container: hdsc using MariaDB

## Quick Start

`cd $HAPPSC_REPO_DIR`

`git clone --recursive https://github.com/johnblaut/happsc.git`

`mv $APP_ENVIRONMENT.env .env`

`cp secrets/sample/.asc.env secrets/sample/.csc.env secrets/sample/.dsc.env $SEC_DIR`

`vim $SEC_DIR/.asc.env`

`vim $SEC_DIR/.csc.env`

`vim $SEC_DIR/.dsc.env`

`docker-compose up`

## Setup Guide

TBD

