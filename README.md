# Reverse Proxy Demo with NGINX

This demo showcases how to use an Nginx Reverse Proxy component to hide implementation details about which servers respond to which requests. It acts like a "programming interface".

### Steps

1. Run `docker compose up -d` (make sure that port 8080 is open)
2. Run `curl http://localhost:8080`. This will show the `/static/index.html` file
3. Change the `/static/index.html` to another thing
4. Run `curl http://localhost:8080` and see how the file changed
5. Run `curl http://localhost:8080/somethingElse` and see the custom 404 error site
6. Run `curl http://localhost:8080/service-1` to see that the request is redirected to the container called "service-1"
7. Run `curl http://localhost:8080/service-2` to see that the request is redirected to the container called "service-2"
8. Run `curl http://localhost:8080/service-12` to see that the `/static/service-12/index.html` file is shown
9. Run `docker compose down service1` and `curl http://localhost:8080/service-1` to see a 502 Bad Gateway error.