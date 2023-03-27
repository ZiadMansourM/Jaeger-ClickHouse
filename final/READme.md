# jaeger with clickhouse - [link](https://github.com/jaegertracing/jaeger-clickhouse)

- [X] Make sure you copy these three files:
- config.yaml
- jaeger-clickhouse-linux-arm64
- jaeger-ui.json
- [X] docker-compose.yml
```.yml
version: '3'
services:
  jaeger:
    image: jaegertracing/all-in-one:1.32.0
    container_name: jaeger
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


<img width="1440" alt="Screenshot 2023-03-27 at 2 49 12 AM" src="https://user-images.githubusercontent.com/64917739/227816443-d5cc5862-2f29-45d2-928a-fa6e79871fce.png">
<img width="1440" alt="Screenshot 2023-03-27 at 2 49 24 AM" src="https://user-images.githubusercontent.com/64917739/227816459-6da065a2-5e16-43fa-b894-567554b27f04.png">

