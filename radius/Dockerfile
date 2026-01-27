FROM ubuntu:latest

RUN apt-get update && \
    apt-get install -y freeradius freeradius-postgresql postgresql-client && \
    rm -rf /var/lib/apt/lists/*

COPY raddb /etc/freeradius/3.0
RUN sed -i '1i$INCLUDE /etc/freeradius/3.0/users\n' /etc/freeradius/3.0/mods-config/files/authorize

COPY init_db.sql /init_db.sql

EXPOSE 1812/udp 1813/udp

CMD ["freeradius", "-X"]
