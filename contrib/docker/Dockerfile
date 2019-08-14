FROM artprod.dev.bloomberg.com/ubuntu18:latest

ENV http_proxy="http://proxy.bloomberg.com:81"
ENV https_proxy="http://proxy.bloomberg.com:81"
ENV ftp_proxy="ftp://proxy.bloomberg.com:81"
ENV no_proxy="localhost,127.0.0.1,localaddress,.localdomain.com,blp-dpkg.dev.bloomberg.com,blp-dpkg.dev.bloomberg.com,blp-dpkg.dev.bloomberg.com,.bloomberg.com,10.0.0.0/8,100.0.0.0/8"

RUN useradd --user-group --create-home --no-log-init --shell /bin/bash superset

ENV LANG=C.UTF-8 \
    LC_ALL=C.UTF-8

RUN apt-get update -y

RUN apt-get install -y apt-transport-https \
                       apt-utils \
                       build-essential \
                       libssl1.0-dev \
                       libffi-dev \
                       python3-dev \
                       python3-pip \
                       libsasl2-dev \
                       libldap2-dev \
                       libxi-dev \
                       vim \
                       less \
                       postgresql-client \
                       redis-tools \
                       nodejs-dev \
                       node-gyp \
                       npm

RUN curl -sL https://deb.nodesource.com/setup_10.x | bash - \
    && apt-get install -y nodejs

WORKDIR /home/superset

USER superset

COPY requirements.txt .
COPY requirements-dev.txt .
COPY contrib/docker/requirements-extra.txt .
COPY contrib/docker/pip.conf /home/superset/.pip/

RUN pip3 install --user --upgrade setuptools pip \
    && pip3 install --user pytest-runner \
    && pip3 install --user -r requirements.txt -r requirements-dev.txt -r requirements-extra.txt \
    && pip3 install --user gevent

COPY --chown=superset:superset superset superset

ENV PATH=/home/superset/superset/bin:$PATH \
    PYTHONPATH=/home/superset/superset/:$PYTHONPATH

RUN cd superset/assets \
    && npm install \
    && npm ci \
    && npm run build \
    && rm -rf node_modules

COPY contrib/docker/docker-init.sh .
COPY contrib/docker/docker-entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]

HEALTHCHECK CMD ["curl", "-f", "http://localhost:8088/health"]

EXPOSE 8088
