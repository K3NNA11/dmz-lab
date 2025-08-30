
# Informe de configuración de DMZ con Cisco Packet Tracer

## 1. Objetivo del laboratorio
Configurar una DMZ segura usando un router Cisco ISR, aplicando NAT y ACLs para controlar el tráfico entre LAN, DMZ y red externa.

## 2. Topología implementada
La red implementada consta de tres zonas:

- **LAN interna (192.168.1.0/24):** contiene un PC interno, representa la red privada de la organización.
- **DMZ (192.168.2.0/24):** contiene un servidor web (192.168.2.10) que debe ser accesible desde Internet.
- **Red externa (192.168.3.0/24):** contiene un PC externo (192.168.3.10), simula el Internet.

📌 **Cantidad de redes:** 3  
📌 **Dispositivos usados:** 1 router Cisco (Router_FW), 3 switches, 2 PCs, 1 servidor.

## 3. Plan de direccionamiento IP

| Dispositivo              | IP            | Máscara        | Gateway     |
|---------------------------|--------------|----------------|-------------|
| PC_Internal              | 192.168.1.10 | 255.255.255.0  | 192.168.1.1 |
| Server_DMZ (Web)         | 192.168.2.10 | 255.255.255.0  | 192.168.2.1 |
| PC_External              | 192.168.3.10 | 255.255.255.0  | 192.168.3.1 |
| Router_FW Gi0/0 (LAN)    | 192.168.1.1  | 255.255.255.0  | – |
| Router_FW Gi0/1 (DMZ)    | 192.168.2.1  | 255.255.255.0  | – |
| Router_FW Gi0/2 (Ext)    | 192.168.3.1  | 255.255.255.0  | – |

## 4. Configuración aplicada (resumen)

### Interfaces del Router_FW
```bash
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.2.1 255.255.255.0
 no shutdown
 ip nat inside

interface GigabitEthernet0/2
 ip address 192.168.3.1 255.255.255.0
 no shutdown
 ip nat outside
```

### NAT estático
```bash
ip nat inside source static 192.168.2.10 192.168.3.1
```

### ACLs configuradas
- Permitir acceso HTTP desde Internet a la DMZ:
```bash
access-list 101 permit tcp any host 192.168.3.1 eq 80
interface GigabitEthernet0/2
 ip access-group 101 in
```

- Bloquear acceso desde DMZ hacia LAN interna:
```bash
access-list 100 deny ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255
access-list 100 permit ip any any
interface GigabitEthernet0/1
 ip access-group 100 in
```

## 5. Verificaciones realizadas
- **Ping desde PC_Internal al router (192.168.1.1):** ✅ exitoso.  
- **Ping desde Web_DMZ al router (192.168.2.1):** ✅ exitoso.  
- **Acceso HTTP desde PC_External (192.168.3.10) a 192.168.3.1:** ✅ exitoso (traducción NAT funciona).  
- **Ping desde PC_External a 192.168.3.1:** ❌ bloqueado por ACL.  
- **Acceso desde DMZ hacia PC_Internal:** ❌ bloqueado por ACL.  

## 6. Conclusiones y recomendaciones
Con este laboratorio se comprobó cómo implementar una **zona desmilitarizada (DMZ)** en Packet Tracer usando un router Cisco.  
Se aplicaron correctamente **NAT estático y ACLs** para:  
- Exponer un servidor web al exterior.  
- Restringir el acceso no autorizado desde la DMZ hacia la LAN.  

**Recomendaciones:**  
- Añadir soporte para HTTPS (puerto 443).  
- Habilitar registros (logs) de seguridad en el router.  
- Comprobar siempre la conectividad básica antes de aplicar ACLs para evitar bloqueos no deseados.

## 7. Evidencias
Las capturas de pantalla de las pruebas (ping, HTTP y bloqueos) se encuentran en la carpeta `evidencias/`.
