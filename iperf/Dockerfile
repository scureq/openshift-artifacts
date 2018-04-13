FROM alpine:3.5
LABEL maintainer scureq
RUN apk add --no-cache iperf3
EXPOSE 5201
ENTRYPOINT ["iperf3", "-s"]