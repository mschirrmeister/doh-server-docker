# doh-server Docker

This project provides a **Dockerfile** for [doh-server](https://github.com/jedisct1/doh-server) project.

The **doh-server** can be used as an **DoH server** or **ODoH target**.

**ODoH** stands for **Oblivious DNS over HTTPS**.

Below are a few examples on how to run it. The recommended way is to run it behind a proxy who handles the TLS termination.

## Examples

Run the container with a local certificate, where doh-server handles TLS itself.

    docker run -it -d \
      --name odoh-server-target \
      -p 3000:3000/tcp \
      -v /Users/marco/MyData/Secure/Certificates/<CertKey>:/certs/cert.key \
      -v /Users/marco/MyData/Secure/Certificates/<CertFullChain>:/certs/cert.pem \
      doh-server:latest --tls-cert-key-path /certs/cert.key --tls-cert-path /certs/cert.pem -l 0.0.0.0:3000 --server-address <upstream-recursive-resolver>:53 -H <FQDN> --allow-odoh-post

Run the container with **Traefik** as a frontend and HTTP (plain) to the backend.

    docker run -it -d \
      --name odoh-server-target \
      -p 127.0.0.1:3000:3000/tcp \
      -p 127.0.0.1:3000:3000/udp \
      -l 'traefik.http.routers.odohtarget.rule=Host(`<FQDN>`)' \
      -l 'traefik.http.routers.odohtarget.tls=true' \
      doh-server:latest -l 0.0.0.0:3000 --server-address <upstream-recursive-resolver>:53 -H <FQDN> --allow-odoh-post

Run the container with **Traefik** as a frontend and **TLS** to the backend.

    docker run -it -d \
      --name odoh-server-target \
      -p 3000:3000/tcp \
      -v /Users/marco/MyData/Secure/Certificates/<CertKey>:/certs/cert.key \
      -v /Users/marco/MyData/Secure/Certificates/<CertFullChain>:/certs/cert.pem \
      -l "traefik.http.routers.odohtarget.rule=Host(`<FQDN>`)" \
      -l "traefik.http.routers.odohtarget.tls=true" \
      -l "traefik.http.services.odohtarget.loadbalancer.server.scheme=https" \
      -l "traefik.http.services.odohtarget.loadbalancer.serversTransport=odohtarget@file" \
      doh-server:latest --tls-cert-key-path /certs/cert.key --tls-cert-path /certs/cert.pem -l 0.0.0.0:3000 --server-address <upstream-recursive-resolver>:53 -H <FQDN> --allow-odoh-post
