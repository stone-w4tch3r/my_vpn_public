## containers for facade

- `docker run -d --rm -p 8080:80 dockercloud/hello-world` docker logo and some text
- `docker run -d --rm -p 8080:80 traefik/whoami` info about user request
- `docker run -d --rm -p 8080:8000 crccheck/hello-world` ascii docker logo and some text
- `docker run -d --rm -p 8080:5678 hashicorp/http-echo -text="hello world"` shows passed text as html. This is the most
  promising one
- `docker run -p 8080:8080 -p 8443:8443 --rm -t mendhak/http-https-echo` shows verbose info about request, 8080 for
  http, 8443 for https