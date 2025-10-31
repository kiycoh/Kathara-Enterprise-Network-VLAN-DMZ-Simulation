PROJECT ARCHITECTURE:
* lab.conf - Defines the network topology
* router.startup - Configures the central router/firewall
* client_public.startup - Configures the external client
* client_private.startup - Configures the internal client
* webserver.startup - Configures the DMZ web server
* NEW! switch.startup - Configures a container that simulates a switch (works as a VLAN-aware bridge)
* shared/ - Shared folder between containers (empty)

NETWORK CONFIGURATION:
* VLAN 10 Public: 203.0.113.0/24 (simulates the internet/external users)
   * Router Gateway: 203.0.113.1
   * Public Client: 203.0.113.10
* VLAN 20 DMZ: 10.0.1.0/24 (demilitarized zone)
   * Router Gateway: 10.0.1.1
   * Web Server: 10.0.1.10
* VLAN 30 Private: 172.16.1.0/24 (internal corporate network)
   * Router Gateway: 172.16.1.1
   * Private Client: 172.16.1.10

START THE LAB -> kathara lstart

If the previous command gives a permission error -> kathara lstart --privileged

Expected router log:
--- Startup Commands Log
++ ip link set eth0 up
++ ip link add link eth0 name eth0.10 type vlan id 10
++ ip link add link eth0 name eth0.20 type vlan id 20
++ ip link add link eth0 name eth0.30 type vlan id 30
++ ip addr add 203.0.113.1/24 dev eth0.10
++ ip addr add 10.0.1.1/24 dev eth0.20
++ ip addr add 172.16.1.1/24 dev eth0.30
++ ip link set eth0.10 up
++ ip link set eth0.20 up
++ ip link set eth0.30 up
++ echo 1
++ iptables -F
++ iptables -t nat -F
++ iptables -P FORWARD DROP
++ iptables -P INPUT ACCEPT
++ iptables -P OUTPUT ACCEPT
++ iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
++ iptables -A FORWARD -i eth0.10 -o eth0.20 -d 10.0.1.10 -p tcp --dport 80 -j ACCEPT
++ iptables -A FORWARD -i eth0.30 -o eth0.20 -d 10.0.1.10 -p tcp --dport 80 -j ACCEPT
++ iptables -A FORWARD -i eth0.30 -o eth0.10 -j ACCEPT
++ iptables -t nat -A POSTROUTING -o eth0.10 -j MASQUERADE
++ echo 'Router successfully configured in mode '''Router on a Stick'''!'
Router successfully configured in 'Router on a Stick' mode!
--- End Startup Commands Log

TESTS FROM CLIENT_PRIVATE (Internal Network):
* This MUST work - Access the DMZ web server -> curl http://10.0.1.10
* This MUST work - Access the public network -> ping 203.0.113.10
TESTS FROM CLIENT_PUBLIC (External Network):
* This MUST work - Access the DMZ web server -> curl http://10.0.1.10
* This must NOT work - Access to the private network is BLOCKED -> ping 172.16.1.10
TESTS FROM WEBSERVER (DMZ):
* This must NOT work - The DMZ cannot access the private network -> ping 172.16.1.10

FIREWALL SECURITY RULES:
ALLOWED TRAFFIC:
* Public (203.0.113.0/24) -> DMZ (10.0.1.0/24): Only HTTP port 80
* Private (172.16.1.0/24) -> DMZ (10.0.1.0/24): Only HTTP port 80
* Private (172.16.1.0/24) -> Public (203.0.113.0/24): All traffic
* Return connections for established traffic (ESTABLISHED,RELATED)
BLOCKED TRAFFIC:
* DMZ -> Private: ALL (DMZ security principle)
* Public -> Private: ALL (internal network isolation)
* Any traffic not explicitly allowed

SHUTDOWN THE LAB -> kathara lclean