FROM ubuntu:latest

RUN apt-get update && \
    apt-get install -y bind9 dnsutils && \
    rm -rf /var/lib/apt/lists/*

COPY named.conf /etc/bind/named.conf

EXPOSE 53/tcp 53/udp

CMD ["named", "-g"]
