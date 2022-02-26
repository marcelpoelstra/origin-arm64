ARG ARCH=arm64v8

FROM ${ARCH}/ubuntu:focal

# ARGs declared before FROM are in a different scope, so need to be stated again
# https://docs.docker.com/engine/reference/builder/#understand-how-arg-and-from-interact
ARG DEBIAN_FRONTEND=noninteractive
# Unified Streaming version
ARG VERSION=1.11.10

RUN apt update && apt-get install -y wget gnupg2

# Get USP public key
RUN wget https://beta.apt.unified-streaming.com/unifiedstreaming.pub && \
    apt-key add unifiedstreaming.pub && \
    sh -c 'echo "deb [arch=arm64] https://beta.apt.unified-streaming.com focal-ports multiverse" > /etc/apt/sources.list.d/unified-streaming-arm64.list' 

# Install Origin
RUN apt-get update \
&& apt-get -y install \
    apache2 \
    python3 \
    python3-pip \    
&&  pip3 install \
        pyyaml==5.3.1 \
        schema==0.7.3 \
&& apt-get -y install \
    "mp4split=${VERSION}" \
    "libapache2-mod-smooth-streaming=${VERSION}" \
    "libapache2-mod-unified-s3-auth=${VERSION}" \
    "libapache2-mod-unified-remix=${VERSION}" \    
    "python3-manifest-edit=${VERSION}" \
    "python3-libfmp4=${VERSION}" \
    "manifest-edit=${VERSION}" \
&&  apt-get -y autoremove \
&&  apt-get clean

# Set up directories and log file redirection
RUN mkdir -p /run/apache2 \
    && rm var/log/apache2/error.log \
    && rm var/log/apache2/access.log \
    && ln -s /dev/stderr /var/log/apache2/error.log \
    && ln -s /dev/stdout /var/log/apache2/access.log \
    && mkdir -p /var/www/unified-origin \
    && rm -f /etc/apache2/sites-enabled/* \
    && a2dismod mpm_event \
    && a2enmod mpm_worker \
    && a2enmod mod_smooth_streaming \
    && a2enmod mod_unified_s3_auth \
    && a2enmod headers \
    && a2enmod proxy \
    && a2enmod proxy_http \
    && a2enmod ssl \
    && a2enmod ext_filter

# Copy apache config and entrypoint script
COPY conf.d/* /etc/apache2/conf-enabled
COPY sites-enabled/* /etc/apache2/sites-enabled
COPY entrypoint.sh /usr/local/bin/entrypoint.sh
RUN chmod +x /usr/local/bin/entrypoint.sh

# Copy webpage
COPY html /var/www/unified-origin/

# Copy Transcoder Config for ffmpeg-usp
COPY ffmpeg-transcoders.usp /etc/ffmpeg-transcoders.usp

# set Apache as owner of /var/www/unified-origin so it can write from API
RUN chown www-data:www-data /var/www/unified-origin

EXPOSE 80

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]

CMD ["-D", "FOREGROUND"]