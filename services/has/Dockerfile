ARG phptag

FROM php:${phptag}

ARG apphome
ARG appotfdeploy
ARG appport
ARG appuname
ARG appgname
ARG appuid
ARG appgid

ENV APP_OTF_DEPLOY_OPT=$appotfdeploy

ENV APP_HOME_DIR=$apphome

# Determines the working directory to use depending on whether
# we want on the fly changes to be enabled or not
ENV APP_WORK_DIR=${appotfdeploy:+$apphome/otf}
ENV APP_WORK_DIR=${APP_WORK_DIR:-$apphome/release}

ENV APPPORT=$appport

RUN apt-get -y update --fix-missing && \
    apt-get upgrade -y && \
    apt-get -y install curl libcurl3 libcurl3-dev rsync zip unzip netcat libmcrypt-dev mysql-client zlib1g-dev && \
    docker-php-ext-install mcrypt pdo_mysql zip && \
    curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

RUN mkdir -m 700 $apphome && \
    mkdir -m 700 $apphome/otf && \
    groupadd -g $appgid $appgname && \
    useradd -u $appuid -g $appgname -d $apphome -M $appuname && \
    passwd -l $appuname

COPY bin          $apphome/bin 
COPY happs        $apphome/release
COPY conf/app.env $apphome/release/.env

RUN chmod 700 $apphome/release $apphome/bin $apphome/bin/*.sh && \
    chmod 600 $apphome/release/.env && \
    chown -R ${appuname}. $apphome

WORKDIR $APP_HOME_DIR/release

USER $appuname

RUN composer update

WORKDIR $APP_WORK_DIR

EXPOSE $APPPORT

ENTRYPOINT ${APP_HOME_DIR}/bin/happs_init.sh

