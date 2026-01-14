FROM golang:1.22-alpine AS builder

WORKDIR /usr/src/app/
    
COPY go.mod go.sum .

RUN go mod download

COPY . .

RUN go build -o /go/bin/checkout .

FROM alpine AS release

WORKDIR /usr/src/app/

COPY --from=builder /go/bin/checkout .

ENV CHECKOUT_PORT 8088

ENTRYPOINT ["./checkout"]

