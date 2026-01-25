FROM ubuntu:latest

RUN apt-get update && \
    apt-get install -y openvpn iproute2 iputils-ping && \
    rm -rf /var/lib/apt/lists/*

COPY server.conf /etc/openvpn/server.conf

EXPOSE 1194/udp

CMD ["openvpn", "--config", "/etc/openvpn/server.conf"]