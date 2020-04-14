# Simple docker helloworld

It will run:
- A nodejs container
- On port 8080
- And return "Hello world from server: ${hostname}"

To use:
- Build: `docker build -t gregbk/hello`
- Run: `docker run --name hello -p 8080:8080 gregbk/hello`
- Curl localhost:8080

Based on [project](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/)



