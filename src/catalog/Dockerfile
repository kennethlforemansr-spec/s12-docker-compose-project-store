FROM golang:1.22-alpine AS build

WORKDIR /app

COPY go.mod ./

RUN go mod download

COPY . .

RUN go build -o catalog .

FROM alpine:3.20

WORKDIR /app

COPY --from=build /app/catalog /app/catalog
COPY --from=build /app/db /app/db
COPY --from=build /app/config /app/config

EXPOSE 8080

ENTRYPOINT ["/app/catalog"]