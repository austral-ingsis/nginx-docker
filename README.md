# Reverse Proxy Demo with NGINX

NGINX provides one public entry point, serves static files, and forwards requests to different backend services based on the URL path. Docker Stack runs the demo; the lesson focuses on NGINX.

## Start

Use Docker Engine or Docker Desktop with Linux containers. Run these commands on a Swarm manager. If this engine is not already in a swarm, first run `docker swarm init`.

Stop the previous Compose deployment, if running, before switching to this version. Ports 8080 and 8083 must be available.

```sh
docker stack deploy -c docker-stack.yml reverse-proxy
docker stack services reverse-proxy
```

Wait for the services to reach their desired replica counts. Inspect `docker stack ps reverse-proxy --no-trunc` if they do not.

## Explore NGINX

1. Run `curl -i http://localhost:8080/` to see `static/index.html`.
2. Run `curl -i http://localhost:8080/somethingElse` to see the custom 404 page and its HTTP status.
3. Run `curl -i http://localhost:8080/service-1/` to reach service1 through the proxy.
4. Run `curl -i http://localhost:8080/service-2/` to reach service2 through the same public port.
5. Run `curl -i http://localhost:8080/service-12/` to see a static page instead.
6. Run `curl -i http://localhost:8083/` to access the separate server directly.

Use trailing slashes for backend routes. Compare `curl -i http://localhost:8080/service-1` with `curl -i http://localhost:8080/service-1/`: the first request receives a redirect; the second is proxied internally. `curl -L` follows redirects. This illustrates the difference between a redirect and a reverse proxy.

Look at `conf.d/server.conf`:

- `/service-1/` and `/service-2/` choose different backends. Those backends have no published host ports.
- The proxy strips the service prefix before forwarding: `/service-1/index.html` reaches `/index.html` on service1.
- The proxy location shadows `static/service-1/index.html`, while `/service-12/` uses the static-file location. Ask students to predict which location handles each URL.
- The custom 404 location is `internal`: requesting `/my_custom_404.html` directly cannot expose it as a normal page.

To demonstrate an unavailable backend:

```sh
docker service scale reverse-proxy_service1=0
curl -i --max-time 10 http://localhost:8080/service-1/
curl -i http://localhost:8080/service-2/
docker service scale reverse-proxy_service1=1
```

Expect a gateway failure (502 or 504) for service1 while service2 remains accessible. This differs from a 404: NGINX matched a proxy route, but could not reach its upstream.

## Edit a page or NGINX configuration

The stack distributes HTML and configuration using Swarm configs, so no host bind mounts or custom image build are needed. The separate server's existing Dockerfile remains available as an image-building example; the stack uses its HTML directly.

Edit `static/index.html`, then change its config name in `docker-stack.yml` from `reverse-proxy_home_page_v1` to `reverse-proxy_home_page_v2` and redeploy:

```sh
docker stack deploy -c docker-stack.yml reverse-proxy
# Wait for the replacement proxy task to start:
docker stack services reverse-proxy
curl -i http://localhost:8080/
```

Configs are immutable snapshots: local file edits alone do not affect running containers. For each changed file, increment its config name suffix in `docker-stack.yml` (`v1`, `v2`, `v3`, etc.). Leave unchanged configs at their existing versions. The stack name is `reverse-proxy`.

## Cleanup

```sh
docker stack rm reverse-proxy
```

Removal is asynchronous; wait for it to finish before deploying the same stack again.
