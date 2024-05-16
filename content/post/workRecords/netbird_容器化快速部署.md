---
title: "Netbird_容器化快速部署"
date: 2023-10-24T11:55:11+08:00
categories:
- 工作
- 记录
tags:
- 工作
- 记录
keywords:
- 案例
#thumbnailImage: //example.com/image.jpg
---
netbird搭建文档
<!--more-->
# 背景：
随着对接医院不断增多，出于联合调试需求，往往都需要使用vpn进行联通。
# 环境：
|环境|	版本|		
|---|---|
|centos	|7.9	|
|management	|0.11.5|
|keyclock	|19.0.3	|
|coturn|	4.6|
|dashboard|	1.5.1|		
|signal|	0.11.5|		
开放端口：分别是Dashboard HTTP和HTTPS，Management gRCP和HTTP API，Signal gRPC API）。80, 443, 33073, 10000。Coturn用于使用STUN / TURN协议的中继器：UDP 3478 UDP 49152-65535。
# 详细参数：
密码认证：keyclock
```
version: "3.2"
services:
  keycloak:
    image: bitnami/keycloak:latest
    container_name: keycloak
    environment:
      - KEYCLOAK_CREATE_ADMIN_USER=true
      - KEYCLOAK_ADMIN_USER=user
      - KEYCLOAK_ADMIN_PASSWORD=Aa123456
      - KEYCLOAK_DATABASE_HOST=keyclock-postgres
      - KEYCLOAK_DATABASE_PORT=5432
      - KEYCLOAK_DATABASE_NAME=keycloak
      - KEYCLOAK_DATABASE_USER=postgres
      - KEYCLOAK_DATABASE_PASSWORD=Aa123456
      - KEYCLOAK_DATABASE_SCHEMA=public
    ports:
      - 8081:8080
    networks:
      - "netbird-net"
  keyclock-postgres:
    image: postgres
    container_name: keyclock-postgres
    restart: always
    environment:
      - POSTGRES_PASSWORD=Aa123456
      - POSTGRES_USER=postgres
      - POSTGRES_DB=keycloak
    networks:
      - "netbird-net"
    ports:
      - 60001:5432
    volumes:
      - /data/data/keyclock-postgres:/bitnami/postgresql
networks:
  netbird-net:
``` 
netbird   docker-compose.yaml:
```
version: "3"
services:
  #UI dashboard
  dashboard:
    image: wiretrustee/dashboard:main
    restart: unless-stopped
    ports:
      - 8084:80
    environment:
      - AUTH_AUDIENCE=netbird-client
      - AUTH_CLIENT_ID=netbird-client
      - AUTH_AUTHORITY=https://keycloak.aaa.com/realms/netbird
      - USE_AUTH0=false
      - AUTH_SUPPORTED_SCOPES=openid profile email offline_access api
      - NETBIRD_MGMT_API_ENDPOINT=https://netbird.aaa.com:33072
      - NETBIRD_MGMT_GRPC_API_ENDPOINT=https://netbird.aaa.com:33072
      #- NGINX_SSL_PORT=443
      #- LETSENCRYPT_DOMAIN=netbird.aaa.com
      #- LETSENCRYPT_EMAIL=
      - AUTH_REDIRECT_URI=
      - AUTH_SILENT_REDIRECT_URI=
    volumes:
      - /data/data/netbird/netbird-letsencrypt:/etc/letsencrypt/
  # Signal
  signal:
    image: netbirdio/signal:latest
    restart: unless-stopped
    volumes:
      - /data/data/netbird/netbird-signal:/var/lib/netbird
    ports:
      - 10001:80
  #     # port and command for Let's Encrypt validation
  #      - 443:443
  #    command: ["--letsencrypt-domain", "netbird.aaa.com", "--log-file", "console"]
  # Management
  management:
    image: netbirdio/management:latest
    restart: unless-stopped
    depends_on:
      - dashboard
    volumes:
      - /data/data/netbird/netbird-mgmt:/var/lib/netbird
      - /data/data/netbird/netbird-letsencrypt:/etc/letsencrypt:ro
      - ./management.json:/etc/netbird/management.json
    ports:
      - 33072:443 #API port
  #     # port and command for Let's Encrypt validation without dashboard container
  #   - 443:443
  #    command: ["--letsencrypt-domain", "netbird.aaa.com", "--log-file", "console"]
    command: ["--port", "443", "--log-file", "console"]
  # Coturn
  coturn:
    image: coturn/coturn
    restart: unless-stopped
    domainname: netbird.aaa.com
    volumes:
      - ./turnserver.conf:/etc/turnserver.conf:ro
    #      - ./privkey.pem:/etc/coturn/private/privkey.pem:ro
    #      - ./cert.pem:/etc/coturn/certs/cert.pem:ro
    network_mode: host
``` 
management.json
```
{
    "Stuns": [
        {
            "Proto": "udp",
            "URI": "stun:netbird.aaa.com:3478",
            "Username": "",
            "Password": null
        }
    ],
    "TURNConfig": {
        "Turns": [
            {
                "Proto": "udp",
                "URI": "turn:netbird.aaa.com:3478",
                "Username": "self",
                "Password": "password."
            }
        ],
        "CredentialsTTL": "12h",
        "Secret": "secret",
        "TimeBasedCredentials": false
    },
    "Signal": {
        "Proto": "http",
        "URI": "netbird.aaa.com:10001",
        "Username": "",
        "Password": null
    },
    "Datadir": "",
    "HttpConfig": {
        "Address": "0.0.0.0:33072",
        "AuthIssuer": "https://keycloak.aaa.com/realms/netbird",
        "AuthAudience": "netbird-client",
        "AuthKeysLocation": "https://keycloak.aaa.com/realms/netbird/protocol/openid-connect/certs",
        "CertFile":"/etc/letsencrypt/live/netbird.aaa.com/fullchain.pem",
        "CertKey":"/etc/letsencrypt/live/netbird.aaa.com/privkey.pem",
        "OIDCConfigEndpoint":"https://keycloak.aaa.com/realms/netbird/.well-known/openid-configuration"
    },
    "IdpManagerConfig": {
        "Manager": "none"
     },
    "DeviceAuthorizationFlow": {
        "Provider": "none",
        "ProviderConfig": {
          "Audience": "netbird-client",
          "Domain": "",
          "ClientID": "",
          "TokenEndpoint": "https://keycloak.aaa.com/realms/netbird/protocol/openid-connect/token",
          "DeviceAuthEndpoint": "https://keycloak.aaa.com/realms/netbird/protocol/openid-connect/auth/device"
         }
    }
}
```
turnserver.conf
```
listening-port=3478
tls-listening-port=5349
min-port=49152
max-port=65535
fingerprint
lt-cred-mech
user=self:3H5XrWMYUSjMY4BaDmq+lefL0wOO0TbrkpprzVBTvbc
realm=wiretrustee.com
cert=/etc/coturn/certs/cert.pem
pkey=/etc/coturn/private/privkey.pem
log-file=stdout
no-software-attribute
pidfile="/var/tmp/turnserver.pid"
no-cli
``` 
 
# 参考：
https://netbird.io/docs/getting-started/self-hosting
https://netbird.io/docs/integrations/identity-providers/self-hosted/using-netbird-with-keycloak
https://netbird.io/docs/getting-started/installation
https://github.com/netbirdio/netbird/tree/main/signal
https://github.com/netbirdio/netbird/tree/main/management
 
 
 
 