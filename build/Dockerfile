FROM golang:1.17.0 as builder

WORKDIR /go/src/ithub.com/kbudde/rabbitmq_exporter

# Create the user and group files that will be used in the running container to
# run the process as an unprivileged user.
RUN mkdir /user && \
    echo 'nobody:x:65534:65534:nobody:/:' > /user/passwd && \
    echo 'nobody:x:65534:' > /user/group

# Fetch dependencies first; they are less susceptible to change on every build
# and will therefore be cached for speeding up the next build
COPY ./go.mod ./go.sum ./
RUN go mod download
COPY ./ ./ 
RUN CGO_ENABLED=0 go build -o /bin/rabbitmq_exporter

FROM scratch as scratch

COPY --from=builder /bin/rabbitmq_exporter /bin/rabbitmq_exporter

# Import the user and group files from the first stage.
COPY --from=builder /user/group /user/passwd /etc/

# Import the Certificate-Authority certificates for enabling HTTPS.
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# Declare the port on which the webserver will be exposed.
# As we're going to run the executable as an unprivileged user, we can't bind
# to ports below 1024.
EXPOSE 9419

# Perform any further action as an unprivileged user.
USER nobody:nobody

# Check if exporter is alive; 10 retries gives prometheus some time to retrieve bad data (5 minutes)
HEALTHCHECK --retries=10 CMD ["/bin/rabbitmq_exporter", "-check-url", "http://localhost:9419/health"]
# Run the compiled binary.
ENTRYPOINT ["/bin/rabbitmq_exporter"]

FROM quay.io/sysdig/sysdig-mini-ubi:1.1.10 as ubi

COPY --from=builder /bin/rabbitmq_exporter /bin/rabbitmq_exporter

# Import the user and group files from the first stage.
COPY --from=builder /user/group /user/passwd /etc/

# Import the Certificate-Authority certificates for enabling HTTPS.
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

EXPOSE 9419

# Perform any further action as an unprivileged user.
USER nobody:nobody

# Check if exporter is alive; 10 retries gives prometheus some time to retrieve bad data (5 minutes)
HEALTHCHECK --retries=10 CMD ["/bin/rabbitmq_exporter", "-check-url", "http://localhost:9419/health"]
# Run the compiled binary.
ENTRYPOINT ["/bin/rabbitmq_exporter"]