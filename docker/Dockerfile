### Webapp build
FROM node:16.3.0@sha256:ca6daf1543242acb0ca59ff425509eab7defb9452f6ae07c156893db06c7a9a4 as nodebuild

WORKDIR /webapp
ADD webapp/ /webapp

### 'CPPFLAGS="-DPNG_ARM_NEON_OPT=0"' Needed To Avoid Bug Described in: https://github.com/imagemin/optipng-bin/issues/118#issuecomment-1019838562
### Can be Removed when Ticket will be Closed
RUN CPPFLAGS="-DPNG_ARM_NEON_OPT=0" npm install --no-optional && \
    npm run pack

# Build Focalboard on Debian Bullseye (glibc 2.31)
FROM golang:1.22-bullseye AS builder

WORKDIR /go/src/focalboard
COPY . .

# Build the server with glibc 2.31 compatibility
RUN CGO_ENABLED=1 GOOS=linux GOARCH=amd64 make server-docker os=linux arch=amd64


## Final image
FROM debian:buster-slim@sha256:5b0b1a9a54651bbe9d4d3ee96bbda2b2a1da3d2fa198ddebbced46dfdca7f216

RUN mkdir -p /opt/focalboard/data/files
RUN chown -R nobody:nogroup /opt/focalboard

WORKDIR /opt/focalboard

COPY --from=nodebuild --chown=nobody:nogroup /webapp/pack pack/
COPY --from=gobuild --chown=nobody:nogroup /go/src/focalboard/bin/docker/focalboard-server bin/
COPY --from=gobuild --chown=nobody:nogroup /go/src/focalboard/LICENSE.txt LICENSE.txt
COPY --from=gobuild --chown=nobody:nogroup /go/src/focalboard/docker/server_config.json config.json

USER nobody

EXPOSE 8000/tcp

EXPOSE 8000/tcp 9092/tcp

VOLUME /opt/focalboard/data

CMD ["/opt/focalboard/bin/focalboard-server"]
