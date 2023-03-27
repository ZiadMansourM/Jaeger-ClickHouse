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