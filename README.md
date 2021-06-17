# Zoom-Clone

## Implementing WebRTC and coturn to form a Personal 1-1 Video Conference:

<br />

Two web-servers with different global IP addresses are required therefore the use of Virtual Machines from AWS, Azure or GCP are suggested for deployment.

Two domains are required, this could be done using a domain for the signalling server and a sub-domain for the turn server or two seperate domains for each server, SSL certificates are required for both domains, they can be obtained for free using [acme.sh](https://github.com/acmesh-official/acme.sh), if subdomains are used then wildcard certificates are advised, all acme.sh certificates are valid for 90 days before they need to be renewed, this process can be automated.

This has all been tested on Ubuntu 20.04 systems and all instructions are based on that deployment.

<br />

### Signalling Server-
- 
  Ports |Protocol | Purpose
  ------|---------|----------
  22    | TCP     | SSH during development  
  443   | TCP/UDP | HTTP(s) Communication 
- ```sh
  sudo apt install nodejs
  sudo apt install npm
  npm install -g nodemon
  git clone https://github.com/aaro1270/Zoom-Clone.git
  ```   
- ```sh
  cd Zoom-Clone
  npm install
  ```
- Add the SSL certificates to the SSL folder with the private key being called certKey.key and the certificate being called cert.crt
- Add the domain URL in public/script.js to your turn servers domain, replacing where it says turn.domain on line 16
  - ```sh
    nano public/script.js
    ```
- ```sh
  sudo nodemon index.js 
  ```

<br />

### Turn Server- 
-
   Ports      | Protocol | Purpose
  ------------|----------|---------------------
  22          | TCP      |SSH for development
  443         | TCP/UDP  | HTTP(s) Communication
  3478        | TCP/UDP  | Backup Listening Port
  32769-65535 | TCP/UDP  |Media Relay
- ```sh
  git clone https://github.com/aaro1270/Zoom-Clone.git
  ```    
- ```sh
  sudo apt-get install coturn
  sudo systemctl stop coturn
  ```
- ```sh
  cd Zoom-Clone/turn 
  nano turnserver.conf
  ```
  - Replace the following values:
      - localIP with the machines local IP address of the server.
      - externalIP with the global IP address of the server.
      - USER with the username being used of the machine.
      - DOMAIN with the domain name (including the top level domain) that the server is being deployed under.
  - Lastly add test users using the format on the last line.
- ```sh
  sudo mv /etc/turnserver.conf /etc/turnserver.conf.original
  sudo mv turnserver.conf /etc/
  cd ..
  ```
- Add the SSL certificates to the SSL folder with the private key being called certKey.pem and the certificate being called cert.pem

- ```sh
  sudo nano /etc/default/coturn
  ```
  - Uncomment the TURNSERVER_ENABLED=1 line by removing the #
- ```sh
  sudo systemctl start coturn 
  ```
- Check coturn status: 
  - ```sh
    sudo systemctl status coturn
    ```

<br />

#### Common Turn Issues:
- If the status check shows a fail to bind to port 443 then:
    - ```sh
      sudo systemctl stop coturn
      sudo rm /lib/systemd/system/coturn.service
      sudo systemctl daemon-reload
      sudo systemctl start coturn
      ```
- If a peer has joined but no video and audio is being transferred- check the webrtc-internals on the browser
    - If an error 701 is seen, this can also be tested with the [Triccle Ice](https://webrtc.github.io/samples/src/content/peerconnection/trickle-ice/) site.
        - This is either due to incorrest SSL certificates for the domain that it is deployed to. 
        - Or more commonly the relay ports not correctly being opened.   
        
<br />

### DNS Management-
- Port forwarding for the two domains to the corresponding server 

Allow for the DNS to propergate and then test out the system by going to the domain corresponding to the Signalling Server, log in and have a peer on a different network do the same, using the same Room ID and then you should be in a 1-1 call with them, if not investigate the webrtc-internals for what the issue is, logs are made of all steps in the console tab of the webpage inspection so all steps during a call can be monitored for any issues. 