FROM alpine:latest

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
 # Ensure python is callable via '/usr/bin/python'.
 && ln -s /usr/bin/python3 /usr/bin/python \
 && adduser -D -u 1896 duplicity \
 && mkdir -p /home/duplicity/.cache/duplicity \
 && mkdir -p /home/duplicity/.gnupg \
 && chmod -R go+rwx /home/duplicity/ \
 && su - duplicity -c 'duplicity --version'

VOLUME ["/home/duplicity/.cache/duplicity", "/home/duplicity/.gnupg"]

USER duplicity

ENTRYPOINT ["duplicity"]

CMD ["--help"]
