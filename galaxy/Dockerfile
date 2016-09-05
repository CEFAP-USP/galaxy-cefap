# Galaxy - CEFAP
#
# VERSION       1.0.1

FROM quay.io/bgruening/galaxy:16.04 

MAINTAINER Geinfo, geninfo@icb.usp.br

RUN DEBIAN_FRONTEND=noninteractive apt-get -qq update && \
    apt-get install --no-install-recommends -y wget libcairo2-dev libcurl4-openssl-dev aufs-tools libgd2-xpm-dev build-essential && \
    apt-get autoremove -y && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV GALAXY_CONFIG_BRAND=CEFAP \
GALAXY_CONFIG_SERVE_XSS_VULNERABLE_MIMETYPES=True \
GALAXY_CONFIG_CONDA_AUTO_INSTALL=True \
GALAXY_CONFIG_CONDA_AUTO_INIT=True \
GALAXY_CONFIG_CONDA_ENSURE_CHANNELS=r,bioconda \
GALAXY_CONFIG_NGINX_UPLOAD_PATH=/galaxy/_upload

WORKDIR /galaxy-central

RUN echo "America/Sao_Paulo" | tee /etc/timezone
RUN dpkg-reconfigure --frontend nointeractive tzdata

# Copy config files to container
ADD galaxy.ini $GALAXY_ROOT/config/
ADD job_conf.xml $GALAXY_ROOT/config/
ADD pulsar_actions.yaml $GALAXY_ROOT/config/

# Install Tools
ADD toolshed_tools.yaml $GALAXY_ROOT/tools.yaml
RUN install-tools $GALAXY_ROOT/tools.yaml

# Install needed package
RUN . $GALAXY_VIRTUAL_ENV/bin/activate; pip install --ignore-installed pycurl

# Install needed modules
RUN cpan App::cpanminus
RUN cpanm URI::Escape
RUN cpanm GD::Graph::bars
RUN cpanm PerlIO::gzip

# Copy web files to new directory
ADD web /export/web
ADD web $GALAXY_CONFIG_DIR/web/
RUN chown -R $GALAXY_UID:$GALAXY_GID $GALAXY_CONFIG_DIR/web/

# Mark folders as imported from the host.
VOLUME ["/export/", "/data/", "/var/lib/docker"]

# Expose port 80 (webserver), 21 (FTP server), 8800 (Proxy), 9002 (supervisor webui)
EXPOSE :80
EXPOSE :21
EXPOSE :8800
EXPOSE :9002

# Autostart script that is invoked during container start
CMD ["/usr/bin/startup"]