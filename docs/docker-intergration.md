# Docker Intergration

## Brief Overview
DSA-Visualizer is operating system independent, relying only on a comptaible version of Docker to be installed. The application can be run with any user as long as the desired user has the proper permissons to load and compile algorithms from the base system. The custom algorithms directory can be set within the enviroment variables of the docker-compose file. For instructions on using and installing Docker, use the link here: 
* https://docs.docker.com/engine/install/

## Docker Compose File 
Below is a template file to host an instance of the application. Run "docker compose up -d" within the same directory as the compose file (note the file must remain named "docker-compose.yml") and the application instance will be created.

```yaml
services:
  dsa-visualizer:
    build: .
    ports:
      - "8080:8080"
    volumes:
      - ${CUSTOM_ALGORITHMS_DIR:-./custom-algorithms}:/custom-algorithms:ro
      - ./src/algorithms:/app/src/algorithms:ro
      - ./src:/app/src
      - ./tests:/app/tests
      - ./test.py:/app/test.py
    environment:
      - PYTHONUNBUFFERED=1
      - CUSTOM_ALGORITHMS_DIR=/custom-algorithms
      - ALGO_MAX_CONCURRENT=${ALGO_MAX_CONCURRENT}
      - ALGO_TIMEOUT=${ALGO_TIMEOUT}
      - ALGO_MAX_MEMORY_MB=${ALGO_MAX_MEMORY_MB}
      - ARRAY_MAX_LENGTH=${ARRAY_MAX_LENGTH}
      - ARRAY_MAX_VALUE=${ARRAY_MAX_VALUE}
      - ARRAY_MIN_VALUE=${ARRAY_MIN_VALUE}
      - DEBUG=${DEBUG}
      - DEMO=${DEMO}
    restart: unless-stopped
```


## Building the Image
Developers can adjust the image parameters as needed for development and release. However, distriubution of the software to external users must be done through docker hub. Instructions on how to compile and submit an image to docker hub can be found at the links below: 

* https://docs.docker.com/get-started/introduction/build-and-push-first-image/
* https://docs.docker.com/get-started/docker-concepts/building-images/build-tag-and-publish-an-image/

