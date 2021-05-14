FROM debian:buster

ENV CANTALOUPE_VERSION=5.0.2 \
    JAVA_OPTS=-Xmx2g

EXPOSE 8182

VOLUME /imageroot

# Update packages and install tools
RUN apt-get update -qy && apt-get dist-upgrade -qy && \
    apt-get install -qy --no-install-recommends curl imagemagick \
    libopenjp2-tools ffmpeg unzip default-jre-headless && \
    apt-get -qqy autoremove && apt-get -qqy autoclean

# Run non privileged
RUN adduser --system cantaloupe

# Get and unpack Cantaloupe release archive
RUN curl --silent --fail -OL https://github.com/cantaloupe-project/cantaloupe/releases/download/v$CANTALOUPE_VERSION/cantaloupe-$CANTALOUPE_VERSION.zip \
    && unzip cantaloupe-$CANTALOUPE_VERSION.zip \
    && ln -s cantaloupe-$CANTALOUPE_VERSION cantaloupe \
    && rm cantaloupe-$CANTALOUPE_VERSION.zip \
    && mkdir -p /var/log/cantaloupe /var/cache/cantaloupe \
    && chown -R cantaloupe /cantaloupe /var/log/cantaloupe /var/cache/cantaloupe \
    && cp -rs /cantaloupe/deps/Linux-x86-64/* /usr/

USER cantaloupe
COPY cantaloupe.properties /cantaloupe/cantaloupe.properties

CMD ["sh", "-c", "java -Dcantaloupe.config=/cantaloupe/cantaloupe.properties -jar $JAVA_OPTS /cantaloupe/cantaloupe-$CANTALOUPE_VERSION.jar"]