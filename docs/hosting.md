# Hosting Instructions 

## Description
Although conceptualized and put into motion, deployment of the application was far beyond the scope of the project in its current state. We hope that future groups will acomplish our misson to put our application into production. Below are some breif notes/information gathered on how to do so using existing CSU resources.

## Docker
See [Docker Intergration](docker-intergration.md) for extensive information on using Docker with the application.

## Networking
The computer science web server is run off of the Grail server. The server is an Apache instance, which by default hosts simple html page routes corresponding to the user public folder directory structure. All traffic occurs over the HTTPS port (443) or the HTTP port (port 80). Port information is hidden from the remote user, this is important to safely expose software and utilities to the outside world. Our application, if permitted and possible to host on CSU hardware, must ahere to these principles.

## Hosting Custom Applications using Apache
Below are links and resources to gain a fundemental understanding of Apache, while avoiding complex topics unessecary to hosting our application. Complex knowlege may/may not be required depedending on the route of implementation. Ensure the proper documentation version is used in coordination with the Grail sever's version of Apache. 

* https://httpd.apache.org/docs/2.4/getting-started.html
* https://httpd.apache.org/docs/2.4/bind.html
* https://httpd.apache.org/docs/2.4/configuring.html
* https://httpd.apache.org/docs/2.4/caching.html
* https://httpd.apache.org/docs/2.4/urlmapping.html
* https://httpd.apache.org/docs/2.4/env.html

## Maintenance
To host on Grail, a longterm application gaurdian must be established. That gaurdian must have professor access to the Grail server and some technical knowlege of the application. Their account must be added to the list of Docker users on the server to ensure the application can be taken down and put up again as needed for future updates and development.

## Challenges
The configuration of the application on CSU hardware will prove challenging for future maintainers of the software. Developers must have a comprehensive understanding of computer networking to ensure no vuneribilites are introduced by our application. Time must be dedicated to comunicating with and educating the Grail server admin on how to host our application. This includes teaching new skillsets and understanding network routing to ensure the existing server configuration reamins both fully stable and operational.

## Alternatives
Dicussed this Fall and Spring 26, alternatives can be established to host application to students without the use of the Grail server. However, this alternative introduces a slew of new potential provblems such as establishing new security measures, outside network access/port filtering, and a dedicated server maintainer. 



