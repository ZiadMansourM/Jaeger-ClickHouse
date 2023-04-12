# First stage: Build the Go binary https://hub.docker.com/layers/library/golang/1.20.2/images/sha256-2101aa981e68ab1e06e3d4ac35ae75ed122f0380e5331e3ae4ba7e811bf9d256?context=explore
FROM golang:1.20.2@sha256:2101aa981e68ab1e06e3d4ac35ae75ed122f0380e5331e3ae4ba7e811bf9d256 AS builder

RUN mkdir -p /jaeger-binaries \
    && cd /jaeger-binaries \
    && git clone https://github.com/jaegertracing/jaeger-clickhouse.git

WORKDIR /jaeger-binaries/jaeger-clickhouse/cmd/jaeger-clickhouse

RUN mkdir /output \
    && go mod download github.com/ClickHouse/clickhouse-go \
    && GOOS=linux GOARCH=amd64 go build -o /output/jaeger-clickhouse-linux-amd64 main.go

# Second stage: Create the final image and copy the binary https://hub.docker.com/layers/jaegertracing/all-in-one/1.32.0/images/sha256-a996be69abb23980ea090f381e0f9b1d429d32d043d6ac18c5759dead6680158?context=explore
FROM jaegertracing/all-in-one:1.32.0@sha256:a996be69abb23980ea090f381e0f9b1d429d32d043d6ac18c5759dead6680158

# Copy config.yml to /data/config.yaml and jaeger-ui.json to /data/jaeger-ui.json
COPY config.yaml /data/config.yaml
COPY jaeger-ui.json /data/jaeger-ui.json
COPY --from=builder /output/jaeger-clickhouse-linux-amd64 /data/jaeger-clickhouse-linux-amd64

RUN files="$(ls -la /data)" && echo $files