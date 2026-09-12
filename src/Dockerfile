FROM alpine:3.24

RUN apk update && apk add openssh-server && \
    # This file is recreated by the entrypoint script
    rm /etc/ssh/sshd_config

COPY /entrypoint /entrypoint

ENTRYPOINT ["/entrypoint"]
