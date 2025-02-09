######## pocketbase #########
FROM alpine:3 AS pb_builder

ARG PB_VERSION=0.22.27
ARG ARCH=amd64

ADD https://github.com/pocketbase/pocketbase/releases/download/v${PB_VERSION}/pocketbase_${PB_VERSION}_linux_${ARCH}.zip /tmp/pb.zip
RUN unzip /tmp/pb.zip -d /pb/

FROM alpine:3 AS pb_prod
COPY --from=pb_builder /pb/pocketbase /pb/pocketbase
RUN apk add --no-cache ca-certificates
EXPOSE 8090
CMD ["/pb/pocketbase", "serve", "--http=0.0.0.0:8090"]

######## remix #########
FROM node:20.5.1-bookworm-slim AS remix_builder
WORKDIR /remix
COPY remix/package*.json ./
RUN --mount=type=cache,id=npm,target=/root/.npm \
    npm ci --include=dev
COPY remix/. ./
RUN npm run build

FROM node:20.5.1-bookworm-slim AS remix_prod
WORKDIR /remix
COPY --from=remix_builder /remix/build ./build
COPY --from=remix_builder /remix/package*.json ./
RUN --mount=type=cache,id=npm,target=/root/.npm \
    npm ci --omit=dev
EXPOSE 3000
CMD ["npm", "run", "start"]
