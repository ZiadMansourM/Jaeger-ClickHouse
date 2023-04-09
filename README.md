# 🔧 Configure Jaeger to use ClickHouse as storage backend:
- [🔗](https://github.com/jaegertracing/jaeger-clickhouse) Jaeger ClickHouse grpc-plugin repo.
- [🔗](https://hub.docker.com/r/clickhouse/clickhouse-server/) ClickHouse DockerHub.

# 📝 Structure - [working-example](https://github.com/ZiadMansourM/jaeger/tree/main/clickhouse)
We have 3 directories:
- clickhouse-volumes: for ClickHouse DB volumes.
- clickhouse-plugin: contains go code for graph-plugin that needs to be built.
- data: contains the required binaries and files for jaeger to work with ClickHouse.

### > Example structure
```console
ziadh@Ziads-MacBook-Air clickhouse % tree -I clickhouse-volumes
.
├── README.md
├── clickhouse-plugin
│   ├── Dockerfile
│   ├── go.mod
│   ├── go.sum
│   ├── jaeger-clickhouse-darwin-arm64
│   ├── jaeger-clickhouse-linux-arm64
│   └── main.go
├── data
│   ├── config.yaml
│   ├── jaeger-clickhouse-linux-arm64
│   └── jaeger-ui.json
└── docker-compose.yml

2 directories, 11 files
```

### > Build binaries - [ref](https://www.digitalocean.com/community/tutorials/how-to-build-go-executables-for-multiple-platforms-on-ubuntu-16-04)
In order to build binaries you need to run one of those commands as per your convenience.
```console
*** Template: GOOS=${GOOS} GOARCH=${GOARCH} go build -o jaeger-clickhouse-file main.go
$ GOOS=linux GOARCH=arm64 go build -o jaeger-clickhouse-linux-arm64 main.go
or
$ GOOS=darwin GOARCH=arm64 go build -o jaeger-clickhouse-darwin-arm64 main.go
```

### > docker-compose file:
- [ ] I tried to automate the building binaries processes according to platform but it fails on my machine to output binary on volume, might be a problem with image permission to write on a volume that is already populated on host.

```yml
version: '3'

services:
  # build-binaries:
  #   build:
  #     context: .
  #     dockerfile: ./clickhouse/Dockerfile
  #     args:
  #       GOOS: linux
  #       GOARCH: arm64
  #   volumes:
  #     - ./output:/app/output
  clickhouse:
    image: clickhouse/clickhouse-server:22
    container_name: some-clickhouse-server
    ports:
      - "9000:9000" # client-server communication
      - "8123:8123" # HTTP interface
    volumes:
      - ./clickhouse-volumes/data:/var/lib/clickhouse
      - ./clickhouse-volumes/logs:/var/log/clickhouse-server
  jaeger:
    image: jaegertracing/all-in-one:1.32.0
    container_name: jaeger
    restart: always
    ports:
      - "16686:16686"
      - "14250:14250"
      - "14268:14268"
      - "6831:6831/udp"
    environment:
      - JAEGER_DISABLED=false
      - SPAN_STORAGE_TYPE=grpc-plugin
    volumes:
      - "./data:/data"
    command: >
      --query.ui-config=/data/jaeger-ui.json
      --grpc-storage-plugin.binary=/data/jaeger-clickhouse-linux-arm64
      --grpc-storage-plugin.configuration-file=/data/config.yaml
      --grpc-storage-plugin.log-level=debug
    links:
      - clickhouse
    user: "501"
    # depends_on:
    #   - build-binaries
  hotrod:
    image: jaegertracing/example-hotrod:1.32.0
    environment:
      JAEGER_AGENT_HOST: jaeger
      JAEGER_AGENT_PORT: 6831
    ports:
      - "8080:8080"
    depends_on:
      - jaeger
    command: all
```

```Dockerfile
FROM golang:1.20-alpine

VOLUME /app/output
WORKDIR /app/output

COPY ./clickhouse/go.mod .
COPY ./clickhouse/go.sum .
RUN go mod download
COPY ./clickhouse/main.go .

ARG GOOS
ARG GOARCH
RUN GOOS=${GOOS} GOARCH=${GOARCH} go build -o jaeger-clickhouse-file main.go

RUN touch myfile.txt
```

### > grace-plugin for ClickHouse code - [ref](https://github.com/jaegertracing/jaeger-clickhouse#docker-database-example)
```go
package main

import (
	"flag"
	"net/http"
	"os"
	"path/filepath"

	// Package contains time zone info for connecting to ClickHouse servers with non-UTC time zone
	_ "time/tzdata"

	hclog "github.com/hashicorp/go-hclog"
	"github.com/jaegertracing/jaeger/plugin/storage/grpc"
	"github.com/jaegertracing/jaeger/plugin/storage/grpc/shared"
	"github.com/prometheus/client_golang/prometheus/promhttp"
	yaml "gopkg.in/yaml.v3"

	"github.com/jaegertracing/jaeger-clickhouse/storage"
)

func main() {
	var configPath string
	flag.StringVar(&configPath, "config", "", "The absolute path to the ClickHouse plugin's configuration file")
	flag.Parse()

	logger := hclog.New(&hclog.LoggerOptions{
		Name: "jaeger-clickhouse",
		// If this is set to e.g. Warn, the debug logs are never sent to Jaeger even despite
		// --grpc-storage-plugin.log-level=debug
		Level:      hclog.Trace,
		JSONFormat: true,
	})

	cfgFile, err := os.ReadFile(filepath.Clean(configPath))
	if err != nil {
		logger.Error("Could not read config file", "config", configPath, "error", err)
		os.Exit(1)
	}
	var cfg storage.Configuration
	err = yaml.Unmarshal(cfgFile, &cfg)
	if err != nil {
		logger.Error("Could not parse config file", "error", err)
	}

	go func() {
		http.Handle("/metrics", promhttp.Handler())
		err = http.ListenAndServe(cfg.MetricsEndpoint, nil)
		if err != nil {
			logger.Error("Failed to listen for metrics endpoint", "error", err)
		}
	}()

	var pluginServices shared.PluginServices
	store, err := storage.NewStore(logger, cfg)
	if err != nil {
		logger.Error("Failed to create a storage", err)
		os.Exit(1)
	}
	pluginServices.Store = store
	pluginServices.ArchiveStore = store
	pluginServices.StreamingSpanWriter = store

	grpc.Serve(&pluginServices)
	if err = store.Close(); err != nil {
		logger.Error("Failed to close store", "error", err)
		os.Exit(1)
	}
}
```

### > Configure ClickHouse - [ref](https://github.com/jaegertracing/jaeger-clickhouse#how-to-start-using-jaeger-over-clickhouse)
```yml
address: some-clickhouse-server:9000
# Directory with .sql files to run at plugin startup, mainly for integration tests.
# Depending on the value of "init_tables", this can be run as a
# replacement or supplement to creating default tables for span storage.
# If init_tables is also enabled, the scripts in this directory will be run first.
init_sql_scripts_dir:
# Whether to automatically attempt to create tables in ClickHouse.
# By default, this is enabled if init_sql_scripts_dir is empty,
# or disabled if init_sql_scripts_dir is provided.
init_tables:
# Maximal amount of spans that can be pending writes at a time.
# New spans exceeding this limit will be discarded,
# keeping memory in check if there are issues writing to ClickHouse.
# Check the "jaeger_clickhouse_discarded_spans" metric to keep track of discards.
# If 0, no limit is set. Default 10_000_000.
max_span_count:
# Batch write size. Default 10_000.
batch_write_size:
# Batch flush interval. Default 5s.
batch_flush_interval:
# Encoding of stored data. Either json or protobuf. Default json.
encoding:
# Path to CA TLS certificate.
ca_file:
# Username for connection to ClickHouse. Default is "default".
username:
# Password for connection to ClickHouse.
password:
# ClickHouse database name. The database must be created manually before Jaeger starts. Default is "default".
database:
# If non-empty, enables a tenant column in tables, and uses the provided tenant name for this instance.
# Default is empty. See guide-multitenancy.md for more information.
tenant:
# Endpoint for serving prometheus metrics. Default localhost:9090.
metrics_endpoint: localhost:9090
# Whether to use sql scripts supporting replication and sharding.
# Replication can be used only on database with Atomic engine.
# Default false.
replication:
# Table with spans. Default "jaeger_spans_local" or "jaeger_spans" when replication is enabled.
spans_table:
# Span index table. Default "jaeger_index_local" or "jaeger_index" when replication is enabled.
spans_index_table:
# Operations table. Default "jaeger_operations_local" or "jaeger_operations" when replication is enabled.
operations_table:
# TTL for data in tables in days. If 0, no TTL is set. Default 0.
ttl: 1
# The maximum number of spans to fetch per trace. If 0, no limit is set. Default 0.
max_num_spans:
```

#### > Have a look at the [Makefile](https://github.com/jaegertracing/jaeger-clickhouse/blob/main/Makefile) to know that this file is required also.
```json
{
  "dependencies": {
    "dagMaxNumServices": 200,
    "menuEnabled": true
  },
  "archiveEnabled": true
}
```

## Retention Policy applied
Retention policy applied. spans were deleted after 24 hours as specified in config.yml TTL line [here](https://github.com/ZiadMansourM/jaeger/blob/406521f840e5d65f9159f4440cc582db8ba94f12/clickhouse/data/config.yaml#L47).

Old Spans |  New ones old deleted
:--:|:--:
![Before](https://user-images.githubusercontent.com/116031573/228610052-22fc9ae5-d765-481b-b143-76e48aa1b68a.png)  |  ![After](https://user-images.githubusercontent.com/116031573/228610038-6fccd01b-03be-4e34-9b50-627d741c966a.png)


# ➕ Extra details
Will be helpful reference while debugging or extending.

### > [Makefile](https://github.com/jaegertracing/jaeger-clickhouse/blob/main/Makefile) runs the following command - [ref](https://github.com/jaegertracing/jaeger-clickhouse#docker-database-example)
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

### > Command suggested to run [here](https://github.com/jaegertracing/jaeger-clickhouse#docker-database-example) won't work this one will
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
