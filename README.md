
[![Docker Pulls](https://img.shields.io/docker/pulls/fekide/traefik-custom-error-pages.svg)](https://hub.docker.com/r/fekide/traefik-custom-error-pages/)
[![Docker Stars](https://img.shields.io/docker/stars/fekide/traefik-custom-error-pages.svg)](https://hub.docker.com/r/fekide/traefik-custom-error-pages/)

# Fork from [guillaumebriday/traefik-custom-error-pages](https://github.com/guillaumebriday/traefik-custom-error-pages)

This is a fork of the project by @guillaumebriday in order to provide some additional features of fix issues with the original version.

# Custom error pages for Traefik

A bunch of custom error pages for Traefik built with [Jekyll](https://jekyllrb.com/).

## Development

Install dependencies

```bash
$ bundle install
```

If you want to build the project on your host:

```bash
$ jekyll build
```

If you want to preview the pages before building the Docker image :

```bash
$ jekyll serve
```

Open [http://127.0.0.1:4000/](http://127.0.0.1:4000/).

## How to use with Traefik and Docker in Production

Run the container with labels, **change with your needs**:

```yml
# docker-compose.yml

  errorpage:
    image: fekide/traefik-custom-error-pages
    restart: unless-stopped
    networks:
      - web
    labels:
      # ...
      # default traefik configuration
      # ...
      - traefik.http.routers.errorpage.entrypoints: "websecure"
      - traefik.http.routers.errorpage.rule: "HostRegexp(`{host:.+}`)"
      - traefik.http.services.globalerrorpage.loadbalancer.server.port: "80"
      # The following is a middleware that gets activated when a service returns an error
      # It needs to be added to the middlewares of the respective router (see example below)
      - traefik.http.middlewares.errorpage.errors.status: 400-599
      - traefik.http.middlewares.errorpage.errors.service: globalerrorpage
      - traefik.http.middlewares.errorpage.errors.query: /{status}

  app: 
  # ...
    labels:
      # ...
      # other traefik configuration
      # ...
      - traefik.http.routers.app.middlewares: errorpage # ... other middlewares

```

> Due to the current implementation of the error middleware it is not possible to catch for example all traefik related errors (Service unavailable / Bad Gateway) without specifying them directly. It is possible however to add it to all routers by default:
> ```toml
> # Static traefik configuration
> [entryPoints.<name of entrypoint>.http]
> # in brackets the name of the middleware above
> middlewares = ["errorpage@docker"]
> ```
> However this will overwrite **ALL** requests with 400-599 codes, also ones to non HTML endpoints (like APIs) so it is **NOT RECOMMENDED**


## Build the image

This is a [multi-stage build](https://docs.docker.com/develop/develop-images/multistage-build/), to build the final image:

```bash
$ docker build -f .cloud/docker/Dockerfile -t traefik-custom-error-pages .
```

## How it works?

As you can see in the Dockerfile, I use [Nginx](https://www.nginx.com/) as Web server to serve static files. To generate this pages, I use [Jekyll](https://jekyllrb.com/) in the first step of the build.

You will find in this article [https://www.techjunktrunk.com/docker/2017/11/03/traefik-default-server-catch-all](https://www.techjunktrunk.com/docker/2017/11/03/traefik-default-server-catch-all/) why I set up `rule` this way.

It's very useful because this container will respond to all requests only if there is no container with a real rule.

## Credits

I used the [Laravel](https://laravel.com/) default HTTP error pages.

## Contributing

Do not hesitate to contribute to the project by adapting or adding features ! Bug reports or pull requests are welcome.

## License

This project is released under the [MIT](http://opensource.org/licenses/MIT) license.
