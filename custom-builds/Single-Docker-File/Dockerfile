# Base image and versions
ARG BASE_IMAGE=exadel/compreface-core:latest
ARG VERSION=latest

################# init images ####################
FROM exadel/compreface-admin:${VERSION} as admin
FROM exadel/compreface-api:${VERSION} as api
FROM exadel/compreface-fe:${VERSION} as fe

################# compreface-core ####################
FROM ${BASE_IMAGE}

ENV UWSGI_PROCESSES=2
ENV UWSGI_THREADS=1
ENV TZ=America/Los_Angeles

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

################# compreface-admin ####################
ENV POSTGRES_URL=jdbc:postgresql://${postgres_domain}:${postgres_port}/${POSTGRES_DB}
ENV SPRING_PROFILES_ACTIVE=dev
ENV ENABLE_EMAIL_SERVER=false
ENV ADMIN_JAVA_OPTS=-Xmx1g
ENV CRUD_PORT=8081
ENV MAX_FILE_SIZE=5MB
ENV MAX_REQUEST_SIZE=10MB

RUN apt-get update && apt-get install -y wget apt-transport-https gnupg \
    && wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | apt-key add - \
    && echo "deb https://packages.adoptium.net/artifactory/deb focal main" | tee /etc/apt/sources.list.d/adoptium.list \
    && apt-get update \
    && apt-get install -y temurin-17-jdk \
    && rm -rf /var/lib/apt/lists/*

COPY --from=admin /home/app.jar /app/admin/app.jar

################# compreface-api ####################
ENV API_JAVA_OPTS=-Xmx4g
ENV SAVE_IMAGES_TO_DB=true
ENV API_PORT=8080
ENV CONNECTION_TIMEOUT=10000
ENV READ_TIMEOUT=60000

COPY --from=api /home/app.jar /app/api/app.jar

################# compreface-fe ####################
COPY --from=fe /usr/share/nginx/html /usr/share/nginx/html

################# supervisord ####################
RUN apt-get update && apt-get install -y supervisor mc && rm -rf /var/lib/apt/lists/*
RUN mkdir -p /var/log/supervisor
COPY custom-builds/Single-Docker-File/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY custom-builds/Single-Docker-File/startup.sh /startup.sh
RUN chmod +x /startup.sh

CMD ["/usr/bin/supervisord"]
