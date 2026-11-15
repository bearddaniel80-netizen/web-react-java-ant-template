# syntax=docker/dockerfile:1.7
# ---------- Common Git Stage ----------
FROM alpine:latest AS git-base

RUN --mount=type=cache,target=/var/cache/apk \
    apk update && \
    apk add git

# ------- assembly ---------------
FROM git-base AS assembly

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

ENV GIT_URL=https://github.com/bearddaniel80-netizen/${GIT_REPO}.git

RUN git clone -b build --single-branch ${GIT_URL} ${BUILD_DIR}

RUN git clone -b common --single-branch ${GIT_URL} ${COMMON_DIR}

RUN git clone -b ${DOMAIN} --single-branch ${GIT_URL} ${DOMAIN_DIR}

# ........ build file ..........
WORKDIR /app/build

RUN cp ${BUILD_DIR}/ant/* .

# ........ actual code ..........
WORKDIR /app/${APP_LANG}/src/main/${APP_LANG}/example

RUN cp ${COMMON_DIR}/Application.${EXT} .
RUN cp -R ${COMMON_DIR}/config .
RUN cp -R ${COMMON_DIR}/web .
RUN cp -R ${DOMAIN_DIR}/service .

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

# To run spring test
# RUN ant test

RUN ant dist

# ---------- Java runtime ----------
FROM eclipse-temurin:21-jre AS runtime
WORKDIR /app
COPY --from=build /app/build/dist/ ./
ENTRYPOINT ["java", "-cp", "aql-0.0.1.jar:lib/*", "example.Application"]