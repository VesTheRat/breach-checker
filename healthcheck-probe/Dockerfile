FROM alpine:latest
RUN apk add --no-cache curl
CMD ["sh","-c", "if curl -f http://backend:5000/health; then echo 'backend: OK'; else echo 'backend: UNREACHABLE'; fi"]
