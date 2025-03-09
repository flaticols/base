FROM alpine:3.21.3

# Create a non-root user
ARG USER_ID=1000
ARG GROUP_ID=1000
ARG USERNAME=appuser

# Set up essential environment variables
ENV HOME=/home/$USERNAME \
    LANG=en_US.UTF-8 \
    TZ=Europe/Amsterdam \
    PATH=$PATH:/app

# Install only absolute minimum dependencies
RUN apk update && apk upgrade && \
    apk add --no-cache \
    ca-certificates \
    tzdata \
    bash \
    curl \
    # Clean cache
    && rm -rf /var/cache/apk/*

# Update CA certificates and set timezone
RUN update-ca-certificates && \
    cp /usr/share/zoneinfo/Europe/Amsterdam /etc/localtime && \
    echo "Europe/Amsterdam" > /etc/timezone

# Create non-root user
RUN addgroup -g $GROUP_ID -S $USERNAME && \
    adduser -u $USER_ID -S $USERNAME -G $USERNAME -s /bin/bash -h /home/$USERNAME

# Create app directory with proper permissions
RUN mkdir -p /app && \
    chown -R $USERNAME:$USERNAME /app

# Add a healthcheck
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/ || exit 1

# Remove unnecessary setuid/setgid binaries
RUN find / -perm /6000 -type f -exec chmod a-s {} \; || true

# Switch to non-root user
USER $USERNAME
WORKDIR /app

# Default to empty CMD - should be overridden when using this as a base image
CMD ["sleep", "infinity"]