FROM --platform=$BUILDPLATFORM golang:1.27-alpine@sha256:738d1cf061836894ff6bb8c33881080ac66de8cf0586615012a0c8f592649cfa AS builder
ARG GOPROXY

ARG TARGETOS
ARG TARGETARCH

WORKDIR /workspace

COPY go.mod go.sum ./
RUN for attempt in 1 2 3; do GOOS=$TARGETOS GOARCH=$TARGETARCH go mod download && break; [ "$attempt" = 3 ] && exit 1; sleep $((attempt * 10)); done

COPY api api/
COPY pkg pkg/
COPY cmd cmd/
COPY internal internal/

RUN GOOS=$TARGETOS GOARCH=$TARGETARCH CGO_ENABLED=0 go build -ldflags="-extldflags=-static" -o /lineage-controller-webhook cmd/lineage-controller-webhook/main.go

FROM scratch

COPY --from=builder /lineage-controller-webhook /lineage-controller-webhook
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt

ENTRYPOINT ["/lineage-controller-webhook"]
