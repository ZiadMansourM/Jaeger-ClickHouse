FROM golang:1.17-alpine

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download
COPY main.go .

ARG GOOS
ARG GOARCH
RUN GOOS=${GOOS} GOARCH=${GOARCH} go build -o /app/output/jaeger-clickhouse-${GOOS}-${GOARCH} main.go
