# socat

## Allgemeiner Aufruf

socat \<src\>,\<opt1\>,\<opt2\> \<dst\>,\<opt1\>,\<opt2\>

## Szenario 1: Consolen-Input weiterleiten an entfernten Host

* Auf Node 1 läuft socat
* Auf Node 2 läuft socat
* der Input von Node1 soll auf Node2 ausgegeben werden über Port 8080

Node1:

```
socat - tcp4:node2:8080
```

Nod2: 

```
socat tcp4-listen:8080 -
```

## Szenario 2: Consolen Input weiterleiten an entfernten Host von mehreren Quellen

Node1:

```
socat - tcp4:node2:8080
```

Nod2: 

```
socat tcp4-listen:8080,fork,reuseaddr -
```

Node3:

```
socat - tcp4:node2:8080
```

## Szenario 3: Portweiterleitung 

Node 2 - http Server starten auf localhost

```
python -m http.server
```

Node2: socat:

```
socat -v tcp4-listen:8080,fork,reuseaddr,bind=node2 tcp4:localhost:8080
```

Node 3: 

```
curl node2:8080
```

## Szenario 5: Portweiterleitung via Jumphost

Node1 -> Node2:8080 -> Node3:8080

Node3 - http server starten

```
python -m http.server
```

Node 2: (Jumphost)

```
socat tcp4-listen:8080,fork,reuseaddr,keepalive,keepidle=60,keepintvl=60 tcp4:node3:8080,keepalive,keepidle=60,keepintvl=60
```
socat -dd Option hilfreich beim debuggen

Node1: 

```
curl node2:8080
```

## Szenario 6: Portweiterleiduntg zw. IPv4 und IPv6

Node2: 

```
socat tcp6-listen:8080,fork,reuseaddr,ipv6only=1 tcp4:node3:8080
```

Node 1: 

```
curl [ipv6]:8080
```

## Szenario 7: TLS 

Node1 -> Node2:8443 (TLS) -> Node3:8080 (ohne tls)

Node3: 
```
python -m http.server 
```

Node2: 
```
openssl req -x509 -days 365 -newkey ed25519 -keyout rootCA.key -nodes out rootCA.crt (selfsigned)

openssl req -newkey ed25519 -nodes -keyout server.key -out server.csr

vi server.ext
subjectAltName = @alt_names
[alt_names]
DNS.1 = node2

vi client.ext
subjectAltName = @alt_names
[alt_names]
DNS.1 = node1


# for servercetrificate
openssl x509 -req -CA rootCa.crt -CAkey rootCA.key -in server.csr -out server.crt -days 365 -CAcreateserial -extfile server.ext

cat server.key server.crt > server.pem

openssl x509 -req -CA rootCa.crt -CAkey rootCA.key -in client.csr -out client.crt -days 365 -CAcreateserial -extfile client.ext

cat client.key client.crt > client.pem

scp client.pem node1:
scp rootCA.crt node1:
```

Node 2
```
socat -dd openssl-listen:8443,reuseaddr,fork,cert=server.pem,cafile=rootCA.crt tcp4:node3:8080
```

Node 1:
```
curl https://node2:8443 --cacert rootCA.crt --cert client.pem
```

## Szenario 8: encrypted tunnel 

Node 2:
```
sudo socat openssl-listen:8443,reuseaddr,fork,cert=server.pem,cafile=rootCA.crt tun:169.254.0.2/24,up &
```

Node 1: 
```
sudo socat tun:169.254.0.1/24,up openssl-connect:node2:8443,cert=client.pem,cafile=rootCA.crt &
```

