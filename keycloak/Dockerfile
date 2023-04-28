FROM quay.io/keycloak/keycloak:21.0.2 as builder

ENV KC_HEALTH_ENABLED=true
ENV KC_METRICS_ENABLED=true
ENV KC_FEATURES=token-exchange
ENV KC_DB=postgres

RUN /opt/keycloak/bin/kc.sh build --features="preview"

# multistage build
FROM quay.io/keycloak/keycloak:21.0.2
COPY --from=builder /opt/keycloak/ /opt/keycloak/
WORKDIR /opt/keycloak

# For demonstration purposes only, please make sure to use proper certificates in production instead
# RUN keytool -genkeypair -storepass password -storetype PKCS12 -keyalg RSA -keysize 2048 -dname "CN=server" -alias server -ext "SAN:c=DNS:localhost,IP:127.0.0.1" -keystore conf/server.keystore

# Change these values to point to a running postgres instance
ENV KC_DB=postgres
ENV KC_DB_URL=jdbc:postgresql://db/keycloakdb
ENV KC_DB_USERNAME=admin
ENV KC_DB_PASSWORD=password
#ENV KC_HOSTNAME=localhost

# disable https
ENV KC_HOSTNAME_STRICT=false
ENV KC_HOSTNAME_STRICT_HTTPS=false
ENV KC_HTTP_ENABLED=true

ENTRYPOINT ["/opt/keycloak/bin/kc.sh", "start  --features=token-exchange,admin-fine-grained-authz"]  

