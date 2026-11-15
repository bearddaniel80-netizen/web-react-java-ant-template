# syntax=docker/dockerfile:1.7
# ---------- deps ----------
FROM node:22-alpine AS deps

ENV GIT_BASE=github.com/bearddaniel80-netizen

RUN apk add --no-cache git

RUN git clone --depth=1 https://${GIT_BASE}/web-template.git

WORKDIR /build

RUN cp /web-template/frontend/package.json .

RUN --mount=type=cache,target=/root/.npm \
    npm install

# ---------- React build ----------
FROM deps AS react-runtime

WORKDIR /app

COPY --from=deps /build/node_modules ./node_modules

COPY --from=deps /web-template/frontend .
# To generate docs
# RUN npm run docs

RUN npm run build

# ------- assembly ---------------
FROM deps AS assembly

ARG DOMAIN
ARG APP_LANG
ARG EXT

ENV APP_LANG=${APP_LANG}

ENV EXT=${EXT}

ENV DOMAIN=${DOMAIN}

ENV BUILD_DIR=/build-downloads
ENV COMMON_DIR=/common-downloads
ENV DOMAIN_DIR=/${DOMAIN}-downloads

ENV GIT_REPO=web-${APP_LANG}-lib

ENV GIT_URL=https://${GIT_BASE}/${GIT_REPO}.git

RUN git clone -b build --single-branch ${GIT_URL} ${BUILD_DIR}

RUN git clone -b common --single-branch ${GIT_URL} ${COMMON_DIR}

RUN git clone -b ${DOMAIN} --single-branch ${GIT_URL} ${DOMAIN_DIR}

# ........ build file ..........
WORKDIR /app/build

RUN cp ${BUILD_DIR}/ant/* .

# ........ actual code ..........
WORKDIR /app/${APP_LANG}/src/main/${APP_LANG}/example

RUN cp ${COMMON_DIR}/Application.${EXT} .

# ........ resources ..............
WORKDIR /app/${APP_LANG}/src/main

RUN cp -R ${DOMAIN_DIR}/resources .

# ---- build stage ----
FROM eclipse-temurin:21-jdk AS build

ARG APP_LANG
ENV APP_LANG=${APP_LANG}

RUN apt-get update \
 && apt-get install -y --no-install-recommends ant \
 && rm -rf /var/lib/apt/lists/*

 WORKDIR /app

# Resolve dependencies first so this layer is cached until ivy.xml changes
COPY --from=assembly /app/build .
RUN ant resolve

COPY ${APP_LANG}/ ./

COPY --from=assembly /app/${APP_LANG} .

# Copy React production build into Spring Boot static resources
COPY --from=react-runtime /app/dist/ src/main/resources/static/

# To run spring test
# RUN ant test

RUN ant dist

# ---------- Java runtime ----------
FROM eclipse-temurin:21-jre AS runtime
WORKDIR /app
COPY --from=build /app/build/dist/ ./
ENTRYPOINT ["java", "-cp", "aql-0.0.1.jar:lib/*", "example.Application"]