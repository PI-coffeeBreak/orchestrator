# 🔹 Step 1: Build Keycloak with custom settings
FROM quay.io/keycloak/keycloak:26.1.4 AS builder

# Enable health and metrics support
ENV KC_HEALTH_ENABLED=true
ENV KC_METRICS_ENABLED=true
ENV KC_DB=postgres
ENV PROXY_ADDRESS_FORWARDING=true
#ENV KC_HTTP_RELATIVE_PATH=/auth/
ENV KC_HTTP_ENABLED=true
ENV KC_HTTPS=true
ENV KC_PROXY_HEADERS=forwarded

WORKDIR /opt/keycloak

# Copy custom themes into the Keycloak providers directory
COPY ./keycloak/providers/ /opt/keycloak/providers/

# Generate a self-signed certificate for HTTPS (Development use only!)
RUN keytool -genkeypair -storepass password -storetype PKCS12 \
    -keyalg RSA -keysize 2048 \
    -dname "CN=server" \
    -alias server \
    -ext "SAN:c=DNS:localhost,IP:127.0.0.1" \
    -keystore conf/server.keystore

# Build Keycloak with the applied configurations
RUN /opt/keycloak/bin/kc.sh build

# Ensure the /exports directory exists and contains the necessary files
RUN mkdir -p /opt/keycloak/data/import && echo "Ensure this directory contains the necessary files"

# 🔹 Step 2: Create the final image
FROM quay.io/keycloak/keycloak:26.1.4

# Copy the built Keycloak instance
COPY --from=builder /opt/keycloak/ /opt/keycloak/

COPY ./exports/ /opt/keycloak/data/import/

# Set database environment variables (Docker Compose will override them)
ENV KC_DB=postgres
ENV KC_DB_URL=jdbc:postgresql://keycloak-db:5432/keycloak
ENV KC_DB_USERNAME=keycloak
ENV KC_DB_PASSWORD=keycloakpassword
ENV KC_HEALTH_ENABLED=true
ENV KC_METRICS_ENABLED=true

# Set Keycloak hostname (ensure this matches your domain if using HTTPS)
ENV KC_HOSTNAME=localhost

ENV KEYCLOAK_IMPORT=/opt/keycloak/data/import

# Expose Keycloak's default port
EXPOSE 8443
EXPOSE 9000

# Start Keycloak in PRODUCTION mode
ENTRYPOINT ["/opt/keycloak/bin/kc.sh", "start", "--optimized", "--proxy-headers", "forwarded", "--import-realm"]
