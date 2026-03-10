# Ejemplos Clase 4

## Comandos Importantes

### Agregacion de enlaces
```shell
# Definimos el rango de interfaces fisicas que van a componer la interfaz logica
interface range fa0/3-5
# Establecemos el protocolo a utilizar
channel-protocol <lacp | pagp>
# Establecemos el grupo de enlaces y lo identificamos con un id 
# y establecemos el modo
channel-group <id> mode <active | passive | auto | desirable >
# Mostramos los grupos de enlaces configurados
show etherchannel
show etherchannel port-channel
# Mostramos la configuracion de un grupo de enlaces en especifico
show interfaces port-channel <id>
```

### Lista de Accesos (ACL)
#### Estandar
```shell
access-list <access-list-number> { deny | permit | remark } <source> <source-wildcard>
```

#### Extendida
```shell
access-list <numero> <permit | deny> <protocolo> <origen> <wildcard> [operador] [puerto] <destino> <wildcard> [operador] [puerto]
```

## Ejemplo 

Se configuraron **Listas de Control de Acceso (ACL)** en el router para controlar la comunicación entre distintas VLANs.  
Las reglas principales son:

- Bloquear **ping (ICMP echo)** desde la red **192.168.10.0** hacia **192.168.40.0**.
- Permitir únicamente la **respuesta del ping (echo-reply)**.
- Bloquear el tráfico IP entre **192.168.10.0** y **192.168.20.0**.
- Permitir el resto del tráfico.
- Aplicar las ACL en las subinterfaces correspondientes del router.


ACL extendida utilizada para controlar tráfico desde la red 192.168.10.0

### Rouete0

```bash
enable
configure terminal
access-list 110 deny icmp 192.168.10.0 0.0.0.255 192.168.40.0 0.0.0.255 echo
access-list 110 permit icmp 192.168.10.0 0.0.0.255 192.168.40.0 0.0.0.255 echo-reply
access-list 110 deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
access-list 110 permit ip any any
```

Explicación

- Se bloquea el envío de ping desde la red 192.168.10.0 hacia 192.168.40.0.
- Se permite únicamente la respuesta del ping.
- Se bloquea cualquier comunicación IP entre 192.168.10.0 y 192.168.20.0.
- Se permite el resto del tráfico de red.

```bash
access-list 111 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
access-list 111 permit ip any any
```

Explicación

- Se bloquea el tráfico IP desde 192.168.20.0 hacia 192.168.10.0.
- Se permite cualquier otro tráfico.


#### Aplicación de ACL 110 en la subinterfaz VLAN 10

```bash
interface gigabitEthernet 0/0.10
ip access-group 110 out
no shutdown
exit
```

Se aplica la ACL 110 en la subinterfaz correspondiente a VLAN 10, filtrando el tráfico saliente. 

#### Aplicación de ACL 111 en la subinterfaz VLAN 20

```bash
interface gigabitEthernet 0/0.20
ip access-group 111 out
exit
``` 

Explicación

La ACL 111 se aplica en la subinterfaz de VLAN 20, filtrando el tráfico que sale de esta red.


## Consideraciones sobre las ACL (Access Control Lists)

- **Denegación implícita:**  
  Todas las ACL tienen al final una regla implícita `deny ip any any`.  
  Esto significa que cualquier tráfico que no coincida con una regla permitida será bloqueado automáticamente.

- **Orden de evaluación:**  
  Las reglas de una ACL se evalúan **de arriba hacia abajo**.  
  La primera coincidencia encontrada es la que se aplica, por lo que el orden de las reglas es muy importante.

- **Reglas más específicas primero:**  
  Es recomendable colocar primero las reglas **más específicas** y después las más generales para evitar que una regla general permita o bloquee tráfico que debería tratarse de forma distinta.

- **Uso de `permit ip any any`:**  
  En muchas configuraciones se agrega esta regla al final para permitir el resto del tráfico que no se haya bloqueado explícitamente.

- **Aplicación en interfaces:**  
  Las ACL no tienen efecto hasta que se aplican a una interfaz mediante el comando: `ip access-group <numero> {in | out}`.

- **Dirección de la ACL:**  
- `in`: filtra el tráfico **antes de entrar** a la interfaz.
- `out`: filtra el tráfico **antes de salir** de la interfaz.

- **Una ACL por protocolo, dirección e interfaz:**  
Solo se puede aplicar **una ACL por protocolo, por dirección (in/out) y por interfaz**.

- **Ubicación recomendada:**  
- **ACL estándar:** cerca del **destino**.  
- **ACL extendidas:** cerca del **origen**.

- **Uso de Wildcard Mask:**  
    Las ACL utilizan **wildcard masks** en lugar de máscaras de red tradicionales.  
    Ejemplo: `192.168.10.0 0.0.0.255` donde `0` significa *debe coincidir* y `255` significa *no importa*.

- **Impacto en la red:**  
Una mala configuración de ACL puede bloquear servicios importantes, por lo que se recomienda probar la conectividad después de aplicarlas.
