# stage: builder
FROM concourse/golang-builder AS builder

WORKDIR /src
COPY . .
ENV CGO_ENABLED 0
COPY assets/ /assets
RUN go build -o /assets/check ./cmd/check
RUN go build -o /assets/print-metadata ./cmd/print-metadata
RUN go get -u github.com/awslabs/amazon-ecr-credential-helper/ecr-login/cli/docker-credential-ecr-login \
      && cp ${GOPATH}/bin/docker-credential-ecr-login /assets/ecr-login
RUN set -e; \
    for pkg in $(go list ./...); do \
      go test -o "/tests/$(basename $pkg).test" -c $pkg; \
    done

# stage: resource
FROM alpine:edge AS resource
RUN apk --no-cache add \
      bash \
      docker \
      jq \
      ca-certificates \
      xz \
      util-linux \
    ;
COPY --from=builder /assets /opt/resource
RUN ln -s /opt/resource/ecr-login /usr/local/bin/docker-credential-ecr-login

# stage: tests
FROM resource AS tests
COPY --from=builder /tests /tests
ADD . /docker-image-resource
RUN set -e; \
    for test in /tests/*.test; do \
      $test -ginkgo.v; \
    done

# final output stage
FROM resource
