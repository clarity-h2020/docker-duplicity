FROM alpine:latest

# This work is based on 'https://github.com/wernight/docker-duplicity' and was modified
# by:
#
# *   Björn Ludwig
#
# The modifications include:
#
# *   Removal of unneeded installation of python dependencies,
# *   Updating the according Pipfile and Pipfile.lock,
# *   Creating a symlink from */usr/bin/python3* to */usr/bin/python*,
# *   Replacing the manual *.gnupg* folder creation by GPG inherent initialization,
#
# Copyright 2020 Werner Beroux, Björn Ludwig

ENV HOME=/home/duplicity

RUN set -x \
 && apk add --no-cache \
        ca-certificates \
        duplicity \
        lftp \
        openssh \
        openssl \
        rsync \
 && update-ca-certificates

COPY Pipfile Pipfile.lock /

RUN set -x \
 && apk add --no-cache py3-pip \
 && pip3 install pipenv \
 && pipenv install --system --deploy \
 && apk del --purge py3-pip \
 # Ensure Python is callable via */usr/bin/python*.
 && ln -s /usr/bin/python3 /usr/bin/python \
 && adduser -D -u 1896 duplicity \
 && mkdir -p /home/duplicity/.cache/duplicity \
 # Ensure the *.gnupg* folder exists.
 && gpg --list-keys \
 && chmod -R go+rwx /home/duplicity/ \
 && su - duplicity -c 'duplicity --version'

VOLUME ["/home/duplicity/.cache/duplicity", "/home/duplicity/.gnupg"]

USER duplicity
 
CMD ["duplicity"]
