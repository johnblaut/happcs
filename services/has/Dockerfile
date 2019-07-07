ARG phptag

FROM php:${phptag}

ARG apphome
ARG appport
ARG appotfdeploy
ARG appuname
ARG appgname
ARG appuid
ARG appgid

ENV APP_HOME_DIR=$apphome
ENV APPPORT=$appport
ENV APP_OTF_DEPLOY_OPT=$appotfdeploy

# Determines the working directory to use depending on whether
# we want on the fly changes to be enabled or not - if enabled 
# use the /otf directory otherwise use the regular /release
# directory which is where the application code gets placed
ENV APP_WORK_DIR=${appotfdeploy:+$apphome/otf}
ENV APP_WORK_DIR=${APP_WORK_DIR:-$apphome/release}

# Update pacakges, installed needed packages, clear packages cache files, install needed php extensions and composer
RUN apt-get -y update --fix-missing && \
    apt-get upgrade -y && \
    apt-get -y install curl libcurl3 libcurl3-dev rsync zip unzip netcat libmcrypt-dev mysql-client zlib1g-dev && \
    apt-get clean autoclean && \
    apt-get autoremove --yes && \
    rm -rf /var/lib/apt/lists/* && \
    docker-php-ext-install mcrypt pdo_mysql zip && \
    curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# create application running user with a locked pw
RUN groupadd -g $appgid $appgname && \
    useradd -u $appuid -g $appgname -d $apphome -m $appuname && \
    passwd -l $appuname

# populate container with application code and configuration in /release
COPY happs     $apphome/release
COPY conf/.env $apphome/release/.env

RUN chmod 700 $apphome/release && \
    chmod 600 $apphome/release/.env && \
    chown -R ${appuname}. $apphome/release

WORKDIR $APP_HOME_DIR/release

# switch to application user as with composer it is
# preferred to avoid root when running it
USER $appuname

RUN composer update

# copy the entrypoint script at the end to avoid rebuilding
# most of the image layers again whenever this script needs 
# updating - user temporarily changed to root again to ensure
# file permissions and ownership can get applied correctly
# before launching the application
USER root

COPY bin $apphome/bin

RUN chmod 700 $apphome/bin $apphome/bin/*.sh && \
    mkdir -m 700 /apphome/otf
    ln -s $APP_WORK_DIR $apphome/deploy
    chown -R ${appuname}. $apphome

# switch back permanently to the intended running user for launching the application
USER $appuname

WORKDIR $APP_HOME_DIR/deploy

ENTRYPOINT ["happs_init.sh"]

EXPOSE $APPPORT

CMD ["php artisan serve"]

