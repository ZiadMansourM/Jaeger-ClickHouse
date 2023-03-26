```console
ziadh@Ziads-MacBook-Air jaeger-clickhouse % docker run --rm -p 9000:9000 --name some-clickhouse-server --ulimit nofile=262144:262144 -d clickhouse/clickhouse-server:22
GOOS=linux make build run
make run-hotrod
```

```yml
clickhouse:
    image: clickhouse/clickhouse-server:22
    container_name: some-clickhouse-server
    ports:
      - "9000:9000"
    ulimits:
      nofile:
        soft: 262144
        hard: 262144
```

## Building
    
```console
ziadh@Ziads-MacBook-Air jaeger-clickhouse % GOOS=linux make build-all-platforms
GOOS=linux GOARCH=amd64 /Library/Developer/CommandLineTools/usr/bin/make build
CGO_ENABLED=0 installsuffix=cgo go build -trimpath -o ./output-files/jaeger-clickhouse-linux-amd64 ./cmd/jaeger-clickhouse/main.go
GOOS=linux GOARCH=arm64 /Library/Developer/CommandLineTools/usr/bin/make build
CGO_ENABLED=0 installsuffix=cgo go build -trimpath -o ./output-files/jaeger-clickhouse-linux-arm64 ./cmd/jaeger-clickhouse/main.go
GOOS=darwin GOARCH=amd64 /Library/Developer/CommandLineTools/usr/bin/make build
CGO_ENABLED=0 installsuffix=cgo go build -trimpath -o ./output-files/jaeger-clickhouse-darwin-amd64 ./cmd/jaeger-clickhouse/main.go
GOOS=darwin GOARCH=arm64 /Library/Developer/CommandLineTools/usr/bin/make build
CGO_ENABLED=0 installsuffix=cgo go build -trimpath -o ./output-files/jaeger-clickhouse-darwin-arm64 ./cmd/jaeger-clickhouse/main.go
```

```console
ziadh@Ziads-MacBook-Air jaeger-clickhouse % ls output-files 
jaeger-clickhouse-darwin-amd64	jaeger-clickhouse-linux-amd64
jaeger-clickhouse-darwin-arm64	jaeger-clickhouse-linux-arm64
```

```console
docker run --rm --name jaeger 
-e JAEGER_DISABLED=false 
--link some-clickhouse-server 
-it -u 501 
-p16686:16686 -p14250:14250 -p14268:14268 -p6831:6831/udp 
-v "/Users/ziadh/Desktop/repos/sk_sre/jaeger/clickhouse-example/jaeger-clickhouse:/data" 
-e SPAN_STORAGE_TYPE=grpc-plugin 
jaegertracing/all-in-one:1.32.0 
--query.ui-config=/data/jaeger-ui.json 
--grpc-storage-plugin.binary=/data/jaeger-clickhouse-linux-arm64 
--grpc-storage-plugin.configuration-file=/data/config.yaml 
--grpc-storage-plugin.log-level=debug
```