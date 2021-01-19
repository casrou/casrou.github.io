---
layout: post
title: Minimal Actix Web Application on Docker
---

Setup a minimal web server using [actix-web](https://actix.rs/docs/getting-started/) and [docker](https://hub.docker.com/_/rust).

## Problems
### Actix Web
Following the [getting started](https://actix.rs/docs/getting-started/) section for Actix Web (with `hello-world` swapped for `actix-hello-world`), I got the following error when starting the application with `cargo run`:

```
 => ERROR [4/4] RUN cargo install --path .                                                                         0.5s
------
 > [4/4] RUN cargo install --path .:
#8 0.512   Installing actix-hello-world v0.1.0 (/usr/src/myapp)
#8 0.514 error: failed to compile `actix-hello-world v0.1.0 (/usr/src/myapp)`, intermediate artifacts can be found at `/usr/src/myapp/target`
#8 0.514
#8 0.514 Caused by:
#8 0.514   failed to parse lock file at: /usr/src/myapp/Cargo.lock
#8 0.514
#8 0.514 Caused by:
#8 0.514   invalid serialized PackageId for key `package.dependencies`
------
executor failed running [/bin/sh -c cargo install --path .]: exit code: 101
```
This was simply because my installed version of rust was outdated. I fixed this by running `rustup update`.

### Docker
To run my application as a docker image, I used the [Start a Rust instance running your app](https://hub.docker.com/_/rust) section from the official rust image on Docker Hub.

Attempting to run the docker image after building using the command `docker run -it --rm --name actix-hello-world-app actix-hello-world`, I was met with the following error:

```
docker: Error response from daemon: OCI runtime create failed: container_linux.go:370: starting container process caused: exec: "myapp": executable file not found in $PATH: unknown.
```
This was fixed by changing the version of rust and the name _myapp_ to _actix-hello-world_. My updated `Dockerfile` then looked like this:

```
FROM rust:1.49.0

WORKDIR /usr/src/actix-hello-world
COPY . .

RUN cargo install --path .

CMD ["actix-hello-world"]
```

After succesfully running my actix-web application on Docker, I wanted to access one of the endpoints from outside Docker.
Naturally, I published the specified port with the `-p` parameter, which gave me the following run command:

`docker run -it -p 8080:8080 --rm --name actix-hello-world-app actix-hello-world`

Unfortunately, when accessing _http://localhost:8080/_, Chrome gave me an *ERR_EMPTY_RESPONSE* response. From [this answer](https://stackoverflow.com/a/46203897/9549916) on StackOverflow, I changed the binding address inside `main.rs` from _127.0.0.1:8080_ to _0.0.0.0:8080_, re-built the application and ran it again, which fixed the problem and Chrome now succesfully showed `Hello World`.

## Relevant files

{% gist 0a2a55b489160a8f26178b7eb7e85aee %}

{% gist 20d444854192df9f7cdd04f52c9451d4 %}