FROM mariadb:10.5

ENV MYSQL_ROOT_PASSWORD=admin123
ENV MYSQL_DATABASE=accounts

RUN apt-get update -y && \
    apt-get install -y git unzip zip && \
    rm -rf /var/lib/apt/lists/*
    
WORKDIR /tmp/
RUN git clone -b main https://github.com/hkhcoder/vprofile-project.git && \
    mv vprofile-project/src/main/resources/db_backup.sql /docker-entrypoint-initdb.d/db_backup.sql

EXPOSE 3306

