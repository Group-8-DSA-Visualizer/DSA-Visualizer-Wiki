# Docker Intergration
## Brief Overview
DSA-Visualizer is operating system independent, relying only on a comptaible version of Docker to be installed. The application can be run with any user as long as the desired user has the proper permissons to load and compile algorithms from the base system. The custom algorithms directory can be set within the enviroment variables of the docker-compose file. For instructions on using and installing Docker, use the link here: https://docs.docker.com/engine/install/

## Docker Compose file 
Below is a template file to host an instance of the application. Run "docker compose up -d" within the same directory as the compose file (note the file must remain named "docker-compose.yml") and the application instance will be created.

<insert file template>


## Building the Image
Developers can adjust the image parameters as needed for development and release. However, distriubution of the software to external users must be done through docker hub. Instructions on how to compile and submit an image to docker hub can be found here: <insert link>
