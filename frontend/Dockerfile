FROM nginx:alpine
LABEL maintainer="274822031+VesTheRat@users.noreply.github.com"
RUN apk add --no-cache nodejs npm && npm install -g htmlhint 
COPY . /usr/share/nginx/html
RUN htmlhint /usr/share/nginx/html/index.html 
EXPOSE 80

