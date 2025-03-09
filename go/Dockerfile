# First stage: Build environment for Go
FROM golang:1.24-alpine3.21 AS builder

# Install build dependencies
RUN apk add --no-cache \
    ca-certificates \
    tzdata \
    git \
    make \
    bash

# Set working directory
WORKDIR /build

# Copy go module files first for better layer caching
COPY go.mod go.sum* ./
RUN go mod download && go mod verify

# Copy source code
COPY . .

# Build for multiple architectures
# Note: The actual build command will be executed by the Docker buildx
ARG TARGETOS=linux
ARG TARGETARCH=amd64

# Set CGO according to your needs (disabled by default for better portability)
ENV CGO_ENABLED=0

# Build the application with proper flags
# -trimpath: removes file system paths from binaries
# -ldflags: strip debug information and set version info
RUN GOOS=${TARGETOS} GOARCH=${TARGETARCH} go build \
    -trimpath \
    -ldflags="-w -s -X main.version=$(git describe --tags --always || echo 'dev') -X main.buildTime=$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
    -o /build/app \
    ./cmd/main.go

# Run tests (optional, can be commented out)
RUN go test -v ./...

# Second stage: Use our minimal runtime image
FROM secure-runtime-base AS runtime

# Copy timezone data from builder
COPY --from=builder /usr/share/zoneinfo /usr/share/zoneinfo

# Copy the compiled binary
COPY --from=builder --chown=appuser:appuser /build/app /app/

# Copy configuration files if needed
COPY --from=builder --chown=appuser:appuser /build/config /app/config

# Create data directory if your app needs one
RUN mkdir -p /app/data && chown -R appuser:appuser /app/data

# Set executable permissions
RUN chmod +x /app/app

# Expose ports if needed
EXPOSE 8080

# Run the application
USER appuser
WORKDIR /app
CMD ["/app/app"]