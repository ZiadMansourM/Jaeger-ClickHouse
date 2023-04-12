```console
ziadh@Ziads-MacBook-Air clickhouse-incorso % docker-compose build --no-cache --progress plain
clickhouse uses an image, skipping
hotrod uses an image, skipping
Building jaeger
#1 [internal] load build definition from Dockerfile
#1 sha256:650b09d6d583408781b4ffff49ef71bceaba07beb91e40d59dcf1898164d4ff5
#1 transferring dockerfile: 945B done
#1 DONE 0.0s

#2 [internal] load .dockerignore
#2 sha256:c13c873b53cc89fcb89c3e3dc610b8710dddd30ac6e2ee6f52f390a35ef02e8e
#2 transferring context: 2B done
#2 DONE 0.0s

#4 [internal] load metadata for docker.io/library/golang:1.20.2
#4 sha256:ea08afc6c9ec6e6356f56edab8c5789b6ea239651c37323e1beb4cebe4777dbf
#4 DONE 1.7s

#3 [internal] load metadata for docker.io/jaegertracing/all-in-one:1.32.0@sha256:a996be69abb23980ea090f381e0f9b1d429d32d043d6ac18c5759dead6680158
#3 sha256:966baa1db6112926fd2fc59158dbb9e0dbf5b9808fea50bf789eeb3819bf9f8c
#3 DONE 1.7s

#9 [builder 1/4] FROM docker.io/library/golang:1.20.2@sha256:f7099345b8e4a93c62dc5102e7eb19a9cdbad12e7e322644eeaba355d70e616d
#9 sha256:2f7fce9f676d5c7ae187801f29b1ee9b42c2c9158e3f9e0e1db94cae3e7eb84f
#9 CACHED

#5 [stage-1 1/5] FROM docker.io/jaegertracing/all-in-one:1.32.0@sha256:a996be69abb23980ea090f381e0f9b1d429d32d043d6ac18c5759dead6680158
#5 sha256:b82a5eb21f573633f20e32ab4147c8a6f3fc3ec253415fed5046ac56d48bd6db
#5 CACHED

#6 [internal] load build context
#6 sha256:38848abfd9bd23a5e5c4daf5317de77bb8391c51761df63bdc2e8127010d4e8a
#6 transferring context: 66B done
#6 DONE 0.0s

#7 [stage-1 2/5] COPY config.yaml /data/config.yaml
#7 sha256:1d2ca66c6a94adec8a13b49ab424f1146b3c4f61e4e87dd5964ba2dd9163424d
#7 DONE 0.0s

#10 [builder 2/4] RUN mkdir -p /jaeger-binaries     && cd /jaeger-binaries     && git clone https://github.com/jaegertracing/jaeger-clickhouse.git
#10 sha256:2052d4ddef2f5acc76aeaf3809272979c09f1adf6d5257edd4784432bb84e755
#10 ...

#8 [stage-1 3/5] COPY jaeger-ui.json /data/jaeger-ui.json
#8 sha256:15e1ae6a207f57fdb1fd8ff7e2273af1de3db8d65cd9b8c9e860a3d1f80d7a32
#8 DONE 0.0s

#10 [builder 2/4] RUN mkdir -p /jaeger-binaries     && cd /jaeger-binaries     && git clone https://github.com/jaegertracing/jaeger-clickhouse.git
#10 sha256:2052d4ddef2f5acc76aeaf3809272979c09f1adf6d5257edd4784432bb84e755
#10 0.164 Cloning into 'jaeger-clickhouse'...
#10 DONE 3.2s

#11 [builder 3/4] WORKDIR /jaeger-binaries/jaeger-clickhouse/cmd/jaeger-clickhouse
#11 sha256:0ba6595237e90c187b8bab9e1926f2451a7c087004ac9f7f54252562ecd646da
#11 DONE 0.0s

#12 [builder 4/4] RUN mkdir /output     && go mod download github.com/ClickHouse/clickhouse-go     && GOOS=linux GOARCH=amd64 go build -o /output/jaeger-clickhouse-linux-amd64 main.go
#12 sha256:d36f0d1361edcc284526618314bc5ad21bde3df6739e7e14f51d6ff83fce6c40
#12 18.04 go: downloading github.com/jaegertracing/jaeger v1.38.2-0.20221007043206-b4c88ddf6cdd
#12 18.04 go: downloading github.com/hashicorp/go-hclog v1.3.1
#12 18.05 go: downloading github.com/prometheus/client_golang v1.13.0
#12 18.35 go: downloading gopkg.in/yaml.v3 v3.0.1
#12 19.38 go: downloading github.com/ClickHouse/clickhouse-go/v2 v2.3.0
#12 19.76 go: downloading github.com/fatih/color v1.13.0
#12 19.76 go: downloading github.com/mattn/go-isatty v0.0.14
#12 19.78 go: downloading github.com/prometheus/client_model v0.2.0
#12 20.11 go: downloading github.com/prometheus/common v0.37.0
#12 20.18 go: downloading github.com/gogo/protobuf v1.3.2
#12 20.18 go: downloading github.com/opentracing/opentracing-go v1.2.0
#12 20.20 go: downloading github.com/hashicorp/go-plugin v1.4.5
#12 20.29 go: downloading github.com/spf13/viper v1.13.0
#12 20.30 go: downloading go.uber.org/zap v1.23.0
#12 20.38 go: downloading google.golang.org/grpc v1.50.0
#12 22.02 go: downloading go.uber.org/atomic v1.10.0
#12 23.33 go: downloading github.com/beorn7/perks v1.0.1
#12 23.33 go: downloading github.com/cespare/xxhash/v2 v2.1.2
#12 23.33 go: downloading github.com/golang/protobuf v1.5.2
#12 23.33 go: downloading github.com/prometheus/procfs v0.8.0
#12 23.40 go: downloading google.golang.org/protobuf v1.28.1
#12 23.52 go: downloading github.com/mattn/go-colorable v0.1.12
#12 23.53 go: downloading golang.org/x/sys v0.0.0-20220928140112-f11e5e49a4ec
#12 23.75 go: downloading github.com/uber/jaeger-client-go v2.30.0+incompatible
#12 24.61 go: downloading github.com/ClickHouse/ch-go v0.47.3
#12 26.08 go: downloading github.com/andybalholm/brotli v1.0.4
#12 26.28 go: downloading github.com/pkg/errors v0.9.1
#12 26.31 go: downloading go.opentelemetry.io/otel/trace v1.10.0
#12 26.31 go: downloading github.com/matttproud/golang_protobuf_extensions v1.0.2-0.20181231171920-c182affec369
#12 27.32 go: downloading github.com/fsnotify/fsnotify v1.5.4
#12 27.33 go: downloading go.opentelemetry.io/otel v1.10.0
#12 27.36 go: downloading github.com/grpc-ecosystem/grpc-opentracing v0.0.0-20180507213350-8e809c8a8645
#12 27.37 go: downloading github.com/hashicorp/yamux v0.0.0-20211028200310-0bc27b27de87
#12 27.41 go: downloading github.com/mitchellh/go-testing-interface v1.14.1
#12 27.71 go: downloading github.com/oklog/run v1.1.0
#12 27.72 go: downloading golang.org/x/net v0.0.0-20221002022538-bcab6841153b
#12 27.73 go: downloading github.com/mitchellh/mapstructure v1.5.0
#12 27.73 go: downloading github.com/spf13/afero v1.8.2
#12 27.77 go: downloading github.com/spf13/cast v1.5.0
#12 27.81 go: downloading github.com/spf13/jwalterweatherman v1.1.0
#12 27.89 go: downloading github.com/spf13/pflag v1.0.5
#12 28.77 go: downloading go.uber.org/multierr v1.8.0
#12 28.83 go: downloading google.golang.org/genproto v0.0.0-20220822174746-9e6da59bd2fc
#12 28.85 go: downloading github.com/google/uuid v1.3.0
#12 28.89 go: downloading github.com/paulmach/orb v0.7.1
#12 28.89 go: downloading github.com/shopspring/decimal v1.3.1
#12 28.93 go: downloading github.com/uber/jaeger-lib v2.4.1+incompatible
#12 28.98 go: downloading github.com/go-faster/city v1.0.1
#12 30.07 go: downloading github.com/go-faster/errors v0.6.1
#12 30.12 go: downloading github.com/klauspost/compress v1.15.10
#12 30.16 go: downloading github.com/pierrec/lz4/v4 v4.1.15
#12 30.16 go: downloading github.com/segmentio/asm v1.2.0
#12 30.32 go: downloading golang.org/x/text v0.3.7
#12 38.47 go: downloading github.com/subosito/gotenv v1.4.1
#12 38.55 go: downloading github.com/hashicorp/hcl v1.0.0
#12 38.75 go: downloading gopkg.in/ini.v1 v1.67.0
#12 38.90 go: downloading github.com/magiconair/properties v1.8.6
#12 39.01 go: downloading github.com/pelletier/go-toml/v2 v2.0.5
#12 40.16 go: downloading github.com/pelletier/go-toml v1.9.5
#12 DONE 104.2s

#13 [stage-1 4/5] COPY --from=builder /output/jaeger-clickhouse-linux-amd64 /data/jaeger-clickhouse-linux-amd64
#13 sha256:76ad62d482728bc142b91858471838f9de9fafba8ba725e7ee993665e8386430
#13 DONE 0.1s

#14 [stage-1 5/5] RUN files="$(ls -la /data)" && echo $files
#14 sha256:87faf40a3ec2a807fe860e04c7fa6180368701c369b3ceabcc677faa87800e17
#14 0.162 total 28196 drwxr-xr-x 1 root root 4096 Apr 10 03:43 . drwxr-xr-x 1 root root 4096 Apr 10 03:43 .. -rw-r--r-- 1 root root 2319 Apr 9 18:36 config.yaml -rwxr-xr-x 1 root root 28853664 Apr 10 03:43 jaeger-clickhouse-linux-amd64 -rw-r--r-- 1 root root 109 Apr 9 15:48 jaeger-ui.json
#14 DONE 0.2s

#15 exporting to image
#15 sha256:e8c613e07b0b7ff33893b694f7759a10d42e180f2b4dc349fb57dc6b71dcab00
#15 exporting layers 0.1s done
#15 writing image sha256:ceffd16e7a34375c6bdd1d121140494dfbe116fe28a26b9dfeed0d302ccea21a done
#15 naming to docker.io/library/clickhouse-incorso_jaeger done
#15 DONE 0.1s

Use 'docker scan' to run Snyk tests against images to find vulnerabilities and learn how to fix them
ziadh@Ziads-MacBook-Air clickhouse-incorso % docker-compose up -d
Creating network "clickhouse-incorso_default" with the default driver
Creating default-clickhouse-server ... done
Creating jaeger                    ... done
Creating clickhouse-incorso_hotrod_1 ... done
ziadh@Ziads-MacBook-Air clickhouse-incorso % docker-compose ps   
           Name                          Command               State                Ports             
------------------------------------------------------------------------------------------------------
clickhouse-incorso_hotrod_1   /go/bin/hotrod-linux all         Up      0.0.0.0:8080->8080/tcp,        
                                                                       8081/tcp, 8082/tcp, 8083/tcp   
default-clickhouse-server     /entrypoint.sh                   Up      8123/tcp, 9000/tcp, 9009/tcp   
jaeger                        /go/bin/all-in-one-linux - ...   Up      0.0.0.0:14250->14250/tcp,      
                                                                       0.0.0.0:14268->14268/tcp,      
                                                                       0.0.0.0:16686->16686/tcp,      
                                                                       5775/udp, 5778/tcp,            
                                                                       0.0.0.0:6831->6831/udp,        
                                                                       6832/udp
```