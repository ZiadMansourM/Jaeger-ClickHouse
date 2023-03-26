# Jaeger with ClickHouse Storage Plugin - [ref](https://github.com/jaegertracing/jaeger-clickhouse).

This docker-compose.yml file defines two services, clickhouse and jaeger.

The clickhouse service uses the official ClickHouse image and maps the container ports `8123` and `9000` to the host. 
- Port 8123 is the default ClickHouse `client port`. 
- Port 9000 is the default ClickHouse `server port`.

The jaeger service uses the official `Jaeger all-in-one` image and maps the container ports `16686` and `6831/udp` to the host. 
- Port 16686 is the default `Jaeger UI port`. 
- Port 6831/udp is the default `Jaeger agent port`.

The jaeger service also sets several environment variables to configure Jaeger to use ClickHouse as the storage backend:

- `STORAGE_TYPE` is set to grpc-plugin to enable the ClickHouse storage plugin.
- `STORAGE_PLUGIN_BINARY` is set to /go/bin/clickhouse to specify the path to the ClickHouse storage plugin binary.
- `STORAGE_PLUGIN_CONFIGURATION` is set to specify the connection details to the ClickHouse server and the table to use for storing spans.
- `SPAN_STORAGE_TYPE` is set to grpc-plugin to enable the ClickHouse storage plugin for spans.
- `SPAN_STORAGE_PLUGIN_BINARY` is set to /go/bin/clickhouse to specify the path to the ClickHouse storage plugin binary for spans.
- `SPAN_STORAGE_PLUGIN_CONFIGURATION` is set to specify the connection details to the ClickHouse server and the table to use for storing spans.

The `depends_on` option is used to ensure that the clickhouse service is started before the jaeger service.

Once you have saved this docker-compose.yml file, you can start the services by running the command
> docker-compose up.