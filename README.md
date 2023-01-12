# Pi-hole Setup

## Notes

- When running a deployment in the k3s cluster, iptables are overwritten by docker when using the -p flag, meaning ufw rules on the host node become out-of-date.

    > Example:
    >
    > *on local machine* \
    > nc -zv 192.168.1.92 5333 \
    > nc: connectx to 192.168.1.92 port 5333 (tcp) failed: Connection refused
    >
    > *on node1* \
    > kubectl apply -f deployment.yml
    >
    > *on local machine* \
    > nc -zv 192.168.1.92 5333 \
    > Connection to 192.168.1.92 port 5333 [tcp/\*] succeeded!

- Even with ufw installed, the master node for the Kubernetes cluster will have ports 80 and 443 open because of traefik installed with k3s. Traefik is a modern HTTP reverse proxy and load balancer. Reverse proxy meaning it takes requests from external clients and sends it to the correct server behind the firewall.
- Even with ufw specifying protocol for specific port, if you don't specifically bind a port to the hostIP in the deployment YAML file, the UDP rule in ufw will get overwritten to the TCP default.

    > Example:
    >
    > *on the node*
    > pi@raspberrypi2:~ sudo ufw status verbose
    > Status: active
    > Logging: on (low)
    > Default: deny (incoming), allow (outgoing), deny (routed)
    > New profiles: skip
    > 
    > To                         Action      From
    > --                         ------      ----
    > 22/tcp                     ALLOW IN    Anywhere
    > 8080/tcp                   ALLOW IN    Anywhere
    > 53/tcp                     ALLOW IN    Anywhere
    > 53/udp                     ALLOW IN    Anywhere
    > 8088/tcp                   ALLOW IN    Anywhere
    > 5533/udp                   ALLOW IN    Anywhere
    > 5533/tcp                   ALLOW IN    Anywhere
    > 31234/tcp                  ALLOW IN    Anywhere
    > 31153/udp                  ALLOW IN    Anywhere
    > 31153/tcp                  ALLOW IN    Anywhere
    > 5333/udp                   ALLOW IN    Anywhere
    > 22/tcp (v6)                ALLOW IN    Anywhere (v6)
    > 8080/tcp (v6)              ALLOW IN    Anywhere (v6)
    > 53/tcp (v6)                ALLOW IN    Anywhere (v6)
    > 53/udp (v6)                ALLOW IN    Anywhere (v6)
    > 8088/tcp (v6)              ALLOW IN    Anywhere (v6)
    > 5533/udp (v6)              ALLOW IN    Anywhere (v6)
    > 5533/tcp (v6)              ALLOW IN    Anywhere (v6)
    > 31234/tcp (v6)             ALLOW IN    Anywhere (v6)
    > 31153/udp (v6)             ALLOW IN    Anywhere (v6)
    > 31153/tcp (v6)             ALLOW IN    Anywhere (v6)
    > 5333/udp (v6)              ALLOW IN    Anywhere (v6)
    >
    > *on the master node*
    > *checking pod status*
    > pi@raspberrypi1:~/k3s/pihole $ sudo kubectl describe pod pihole-5ffcdf7fc7-xnv4g -n pihole
    > ...
    > Containers:
    >   pihole:
    >     ...
    >     Ports:          80/TCP, 53/TCP
    >     Host Ports:     31234/TCP, 5333/TCP
    >
    > *on local machine*
    > nc -zv 192.168.1.92 5333
    > Connection to 192.168.1.92 port 5333 [tcp/\*] succeeded!

- Pi-hole required port 53 for the FTL listener. I had to map the container port 53 to a different host port (ie 5333) because systemd-resolved enabled a DNSStubListener on port 53. Instead of disabling it for raspberrypi2-4, I opted to bind it to a different host port. However, this might cause issues with getting pi-hole to work on my phone. Unfortunately, it seems like I can't set port numbers in the DNS settings in iPhone nor my Macbook. May need to alter my approach and disable DNSStubListener on each pi device.
- Altered the hostPort for dns to match 53. Works on iPhone. Can I get it to work with YouTube ads though? :(
- On each pi device, I opened port 53 for /udp and /tcp however was only able to bind the dns service to 53/udp. 53/tcp shows as closed using nmap.
