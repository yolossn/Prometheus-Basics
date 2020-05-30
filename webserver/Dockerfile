FROM golang:1.14 AS builder
WORKDIR /src
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o server ./serverWithMetric.go

FROM scratch
COPY --from=builder /src/server /bin/server

EXPOSE 8090
CMD ["/bin/server"]
