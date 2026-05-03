# Hosting Instructions 

## Description
Although conceptualized and put into motion, deployment of the application was far beyond the scope of the project in its current state. We hope that future groups will acomplish our misson to put our application into production. Below are some breif notes/information gathered on how to do so using existing CSU resources.

## Docker
See (page link here) for extensive information on using Docker with the application.

## Networking
The computer science web server is run off of the Grail server. The server is an Apache instance, which by default hosts simple html page routes corresponding to the user public folder directory structure. All traffic occurs over the HTTPS port (443) or the HTTP port (port 80). Port information is hidden from the remote user, this is important to safely expose software and utilities to the outside world. Our application, if permitted and possible to host on CSU hardware, must ahere to these principles.

## Hosting Custom Applications using Apache
Below are links and resources to gain a fundemental understanding of Apache, while avoiding complex topics unessecary to hosting our application. Complex knowlege may/may not be required depedending on the route of implementation. Ensure the proper documentation version is used in coordination with the Grail sever's version of Apache. 

https://httpd.apache.org/docs/2.4/getting-started.html
https://httpd.apache.org/docs/2.4/bind.html
https://httpd.apache.org/docs/2.4/configuring.html
https://httpd.apache.org/docs/2.4/caching.html
https://httpd.apache.org/docs/2.4/urlmapping.html
https://httpd.apache.org/docs/2.4/env.html

## Matinence
who owns it, who long, and what that entails
## Challenges
The configuration...

## Alternatives
different machine, budget required



