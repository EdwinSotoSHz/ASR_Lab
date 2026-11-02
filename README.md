# Configuraciones de Red (Sucursal y Gerencia)

<details>
<summary>
    <b>Router Sucursal</b>
</summary>

```ini
enable
configure terminal

hostname Sucursal
enable secret cisco

interface g0/0
 ip address 172.16.10.1 255.255.0.0
 no shutdown
 exit

interface g0/1
 ip address 10.0.0.1 255.0.0.0
 no shutdown
 exit

interface s0/0/0
 ip address 200.10.10.1 255.255.255.252
 no shutdown
 exit

router ospf 1
 router-id 1.1.1.1
 network 172.16.10.1 0.0.255.255 area 1
 network 10.0.0.1 0.255.255.255 area 1
 network 200.10.10.0 255.255.255.252 area 1
 passive-interface g0/0
 passive-interface g0/1
 exit

privilege exec level 5 show ip route
username cliente privilege 5 secret cliente

line console 0
 login local
 exit
```

</details>

<details>
<summary>
    <b>Router Gerencia</b> 
</summary>

```ini
enable
configure terminal

hostname Gerencia
enable secret cisco

interface g0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 exit

interface g0/1
 ip address 10.10.0.1 255.255.0.0
 no shutdown
 exit

interface s0/0/0
 ip address 200.10.10.2 255.255.255.252
 no shutdown
 exit

router ospf 2
 router-id 2.2.2.2
 network 192.168.10.1 0.0.0.255 area 1
 network 10.10.0.1 0.0.255.255 area 1
 network 200.10.10.0 255.255.255.252 area 1
 passive-interface g0/0
 passive-interface g0/1
 exit

ip access-list extended PERMISOS
 deny tcp 192.168.10.0 0.0.0.255 host 10.10.0.3 eq www
 deny tcp 192.168.10.0 0.0.0.255 host 10.10.0.3 eq 443
 permit ip 192.168.10.0 0.0.0.255 any
 exit

interface g0/0
 ip access-group PERMISOS in
 exit
```
</details>

- **ACL `PERMISOS`** (entrante en `g0/0` de Gerencia): **deniega** HTTP/HTTPS desde `192.168.10.2/24` y `192.168.10.3/24` hacia `10.10.0.3`, y **permite todo lo demás**, incluido **FTP** (no se bloquea).

- **OSPF:** Se configura área única, con `router-id` único por router, anunciando las redes conectadas y marcando las interfaces LAN como `passive-interface` para que no envíen actualizaciones OSPF

- **Privilegio 5:** hereda los comandos de niveles 0 y 1, por eso `show ip route` funciona; pero `show running-config` es de nivel 15 y **no** está disponible en nivel 5. Comprobable con `show privilege`.