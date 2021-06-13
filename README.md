# Zoom-Clone

Implementing WebRTC and coturn to form a Video Conferencing system.

How To Set Up Personal 1-1 Video Conference:

Two web-servers with differnet global IP addresses are required therefore the use of Virtual Machines from AWS, Azure or GCP are suggested for deployment.

Two domains are required, this could be done using a domain for the signalling server and a sub-domain for the turn server or two seperate domains for each server, SSL certificates are required for these domains, they can be obtained for free using acme.sh(LINK), if subdomains are used then wildcard certificates are advised, all acme.sh certificates are valid for 90 before they need to be renewed, this process can be automated.

This has all been tested on Ubuntu 20.04 systems and all instructions are based on that deployment.

git clone https://github.com/aaro1270/Zoom-Clone.git


Signalling Server-
    - ports- 22 for SSH during development & 443 to allow for HTTPS traffic is required 
    - cd Zoom-Clone
    - Obtain SSL certificates for the domain of choice
    - Place the certificates in the X folder with them being named 
    - modify the domain name being used for the turn server
    - Install nodemon
    - Possibly modify the package, install node, rund node of the package.json
    - run sudo nodemon index.js 


Turn Server- 
    - Ports, 22 for ssh during development, 443- https communication, 3478- backup listening port, 32769-65535 for relay
    - sudo apt-get install coturn
    - Edit the turnserver.conf to contain the local IP address and global IP address
        - nano turnserver.conf
        - Insert the values
            - localIP = local IP address commands
            - globalIP = global IP address commands
            - add the domain that it is being deployed under
    - add ssl certificates to the installation
    - mv turnserver.conf /etc...
    
    - modify the file to enable it
    - sudo systemctl start coturn
    - check status using sudo systemctl status coturn
    - common issue if it says fail to bind to port 443 then:
        - sudo rm /lib/systemd/system/coturn.service
        - sudo systemctl daemon-reload
        - sudo systemctl start coturn
    - common issues is if a peer has joined but no video and audio is being transferred can be spotted in webrtc-internals
        - error 701, this can also be observed with the triccle ice site as well
            - this is either due to incorrest SSL certificates for the domain that it is deployed to 
            - or more commonly the relay ports not correctly being opened 
        

DNS Management-
    - port forwarding for the two domains to the corresponding server 

Allow for the DNS to propergate and then test out the system by going to the domain corresponding to the Signalling Server, log in and have a peer on a different network do the same, using the same Room ID and then you should be in a 1-1 call with them, if not investigate the webrtc-internals for what the issue is, logs are made of all steps in the console tab of the webpage inspection so all steps during a call can be monitored for any issues. 