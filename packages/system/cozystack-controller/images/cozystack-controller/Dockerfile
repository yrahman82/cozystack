FROM --platform=$BUILDPLATFORM golang:1.27-alpine@sha256:738d1cf061836894ff6bb8c33881080ac66de8cf0586615012a0c8f592649cfa AS builder
ARG GOPROXY

ARG TARGETOS
ARG TARGETARCH
ARG VERSION=dev

WORKDIR /workspace

COPY go.mod go.sum ./
RUN for attempt in 1 2 3; do GOOS=$TARGETOS GOARCH=$TARGETARCH go mod download && break; [ "$attempt" = 3 ] && exit 1; sleep $((attempt * 10)); done

COPY api api/
COPY pkg pkg/
COPY cmd cmd/
COPY internal internal/

RUN GOOS=$TARGETOS GOARCH=$TARGETARCH CGO_ENABLED=0 go build \
    -ldflags="-extldflags=-static -X github.com/cozystack/cozystack/pkg/version.Version=${VERSION}" \
    -o /cozystack-controller cmd/cozystack-controller/main.go

FROM scratch

COPY --from=builder /cozystack-controller /cozystack-controller
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt

ENTRYPOINT ["/cozystack-controller"]
