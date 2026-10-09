FROM --platform=$BUILDPLATFORM golang:1.27-alpine@sha256:738d1cf061836894ff6bb8c33881080ac66de8cf0586615012a0c8f592649cfa AS builder
ARG GOPROXY

ARG TARGETOS
ARG TARGETARCH
ARG VERSION=dev

RUN apk add --no-cache make git

COPY . /src/
WORKDIR /src

RUN for attempt in 1 2 3; do go mod download && break; [ "$attempt" = 3 ] && exit 1; sleep $((attempt * 10)); done

# Build cozystack-operator
RUN CGO_ENABLED=0 GOOS=${TARGETOS} GOARCH=${TARGETARCH} go build \
  -ldflags="-w -s -X github.com/cozystack/cozystack/pkg/version.Version=${VERSION}" \
  -o /cozystack-operator \
  ./cmd/cozystack-operator

FROM alpine:3.24@sha256:294b683cb724975bec92580e1e685676bd4b50bda910ddb8c51d4cabeaec77e6

COPY --from=builder /cozystack-operator /usr/bin/cozystack-operator

ENTRYPOINT ["/usr/bin/cozystack-operator"]
