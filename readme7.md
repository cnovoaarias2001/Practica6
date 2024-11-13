**1º Creamos el contenedor con una imgane alpine, y con unos parámetros para que se mantenga activo.**<br>
docker run -d --name cliente alpine tail -f /dev/null<br><br>

**2º Con este comando vamos a comprobar la IP del contenedor creado**<br>
`docker inspect -f '{{range .NetworkSett
ings.Networks}}{{.IPAddress}}{{end}}' cliente`

**3º Engade ó docker-compose do DNS outro servicio (container) que faga a función de cliente.**<br><br>

version: '3'<br>

services:<br>
  dns-server:<br>
    image: internetsystemsconsortium/bind9:9.18<br>

    container_name: dns_server<br>
    networks:<br>
      bind9_subnet:<br>
        ipv4_address: 172.20.0.1  # IP del servidor DNS en la red existente<br>
    ports:<br>
      - "1053:53/udp"  # Tuve que utilizar este puerto debido a que no estaba ocupado<br>

  cliente:<br>
    image: alpine:latest<br>
    container_name: cliente<br>
    command: ["tail", "-f", "/dev/null"]<br>
    networks:<br>
      bind9_subnet:<br>
        ipv4_address: 172.20.0.2  # IP del cliente en la red existente<br>
    depends_on:<br>
      - dns-server<br>
    dns:<br>
      - 172.20.0.1  # Usar el servidor DNS en 172.20.0.1<br>

networks:<br>
  bind9_subnet:<br>
    external: true  # Indica que se usará una red existente<br>

**4º fichero bind9_subnet**<br><br>

{<br>
    "Name": "bind9_subnet",<br>
    "Id": <br>"bd54d0ecc242f9d9ba4ef483a44a2221fcfdb09663886afc21a3b5c167dc5a11",<br>
    "Created": "2023-10-10T06:28:58.162307359Z",<br>
    "Scope": "local",<br>
    "Driver": "bridge",<br>
    "EnableIPv6": false,<br>
    "IPAM": {<br>
        "Driver": "default",<br>
        "Options": {},<br>
        "Config": [<br>
            {<br>
                "Subnet": "172.20.0.0/16",<br>
                "IPRange": "172.20.5.0/24",<br>
                "Gateway": "172.20.5.254"<br>
            }<br>
        ]<br>
    },<br>
    "Internal": false,<br>
    "Attachable": false,<br>
    "Ingress": false,<br>
    "ConfigFrom": {<br>
        "Network": ""<br>
    },<br>
    "ConfigOnly": false,<br>
    "Containers": {},<br>
    "Options": {},<br>
    "Labels": {}<br>
}<br><br>

**5º named.conf.default**<br>
options {<br>
	directory "/var/cache/bind";<br>

	forwarders {<br>
	 	8.8.8.8;<br>
		1.1.1.1;<br>
	 };<br>
	 forward only;<br>

	listen-on { any; };<br>
	listen-on-v6 { any; };<br>

	allow-query {<br>
		any;<br>
	};<br>
};<br>


**6º**
 dig google.com<br>

; <<>> DiG 9.18.27 <<>> google.com<br>
;; global options: +cmd<br>
;; Got answer:<br>
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 40302<br>
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1<br>

;; OPT PSEUDOSECTION:<br>
; EDNS: version: 0, flags:; udp: 1220<br>
; COOKIE: eee897dca47d03ec5aa3268467213c251029d2c33804c721 (good)<br>
;; QUESTION SECTION:<br>
;google.com.			IN	A<br>

;; ANSWER SECTION:<br>
google.com.		177	IN	A	142.250.200.110<br>

;; Query time: 4 msec<br>
;; SERVER: 8.8.8.8#53(8.8.8.8) (UDP)<br>
;; WHEN: Tue Oct 29 19:34:25 UTC 2024<br>
;; MSG SIZE  rcvd: 83<br>

/ # 

/ # ping google.com<br>
PING google.com (142.250.185.14): 56 data bytes<br>
64 bytes from 142.250.185.14: seq=0 ttl=61 time=16.850 ms<br>
64 bytes from 142.250.185.14: seq=1 ttl=61 time=16.112 ms<br>
64 bytes from 142.250.185.14: seq=2 ttl=61 time=16.335 ms<br>


**7º Final**

dig @172.28.5.1 asircastelao.int


; <<>> DiG 9.18.27 <<>> asircastelao.int
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, ...
;; QUESTION SECTION:
;asircastelao.int.          IN  A

;; ANSWER SECTION:
asircastelao.int.    604800  IN  A   172.28.5.1
