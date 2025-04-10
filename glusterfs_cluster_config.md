# Instalación y Configuración de GlusterFS en el Clúster

> **Nota:** Todos los nodos deben configurarse de manera similar. Las IPs privadas mencionadas corresponden a cada nodo del clúster.
> Para ver la IP privada de un nodo, puedes usar el comando:  
> `ip a | grep inet` o `hostname -I`  
> Busca una IP en el rango de tu red local, por ejemplo `10.x.x.x`.

## 1. Instalación de GlusterFS

Instalar los paquetes necesarios en **cada nodo del clúster**:

```bash
sudo apt install glusterfs-cli glusterfs-client glusterfs-common glusterfs-server
```

---

## 2. Iniciar y habilitar el servicio GlusterFS

Activar el servicio para que arranque automáticamente en cada nodo:

```bash
sudo systemctl start glusterd
sudo systemctl enable glusterd
```

---

## 3. Configuración de /etc/hosts

Edita el archivo `/etc/hosts` en cada nodo para que se reconozcan entre sí por nombre:

```bash
sudo vi /etc/hosts
```

Agrega las IPs privadas (estas IPs son ejemplos, reemplázalas por las IPs reales de tus nodos):

```
10.0.0.5 Gates     # IP privada del nodo Gates
10.0.0.6 Hopper    # IP privada del nodo Hopper
10.0.0.4 Turing    # IP privada del nodo Turing
```

> Puedes verificar conectividad entre nodos con:
```bash
ping Gates
```

---

## 4. Verificar instalación de GlusterFS

```bash
dpkg -l | grep glusterfs
```

---

## 5. Comprobación de conexión entre nodos (Peer)

Desde un nodo (por ejemplo, Gates), agregar los otros nodos al clúster:

```bash
sudo gluster peer probe Hopper    # Nodo Hopper
sudo gluster peer probe Turing    # Nodo Turing
```

Verificar estado de los peers:

```bash
sudo gluster peer status
```

Ver estado del servicio:

```bash
sudo systemctl status glusterd
```

---

## 6. Crear carpeta para almacenamiento

En **cada nodo**, crear la carpeta donde GlusterFS almacenará los datos:

```bash
sudo mkdir -p /glusterfs/johnsonvol
```

---

## 7. Crear el volumen GlusterFS

Desde uno de los nodos (por ejemplo Gates), crear el volumen:

```bash
sudo gluster volume create johnsonvol replica 2 arbiter 1 transport tcp \
Gates:/glusterfs/johnsonvol \
Hopper:/glusterfs/johnsonvol \
Turing:/glusterfs/johnsonvol force
```

Explicación:
- `replica 2`: Dos nodos tendrán los datos completos.
- `arbiter 1`: El tercer nodo solo guarda metadatos.
- `transport tcp`: Comunicación vía TCP.
- Los nombres (`Gates`, `Hopper`, `Turing`) deben estar definidos en `/etc/hosts`.

---

## 8. Iniciar el volumen

```bash
sudo gluster volume start johnsonvol
```

Verificar estado:

```bash
sudo gluster volume status
```

---

## 9. Montaje del volumen en cada nodo

### Crear el punto de montaje:

```bash
sudo mkdir -p /mnt/glusterfs
```

### Montaje manual del volumen:

> Reemplaza `10.0.0.5` con la IP privada del nodo Gates, que actuará como punto de acceso.

```bash
sudo mount -t glusterfs 10.0.0.5:/johnsonvol /mnt/glusterfs
```

Verificar montaje:

```bash
df -h | grep gluster
```

---

## 10. Configurar montaje automático

Editar el archivo `/etc/fstab`:

```bash
sudo nano /etc/fstab
```

Agregar al final:

```
10.0.0.5:/johnsonvol /mnt/glusterfs glusterfs defaults,_netdev 0 0
```

> Nuevamente, `10.0.0.5` es la IP del nodo Gates. Puedes usar la IP de cualquier nodo que esté en línea.

Guardar y verificar:

```bash
sudo umount /mnt/glusterfs
sudo mount -a
```

---

## 11. Pruebas de funcionamiento

### Crear archivo de prueba:

```bash
sudo touch /mnt/glusterfs/testfile
```

Verificar que aparece en otro nodo:

```bash
ls -l /mnt/glusterfs
```

### Probar replicación:

```bash
echo "GlusterFS funcionando correctamente" | sudo tee /mnt/glusterfs/prueba_gluster.txt
```

En otro nodo:

```bash
cat /mnt/glusterfs/prueba_gluster.txt
```

### Apagar un nodo y verificar disponibilidad:

```bash
sudo systemctl stop glusterd
```

Desde otro nodo:

```bash
cat /mnt/glusterfs/prueba_gluster.txt
```

Reiniciar el servicio:

```bash
sudo systemctl start glusterd
```

---

## 12. Información del volumen

```bash
sudo gluster volume info
```

---

## 13. Verificación general

- Ver peers del clúster:

```bash
sudo gluster peer status
```

- Ver estado del volumen:

```bash
sudo gluster volume status
```

- Verificar montaje:

```bash
df -h | grep gluster
```

- Crear y validar archivo replicado:

```bash
sudo touch /mnt/glusterfs/archivo_replicado.txt
```

Revisar en otro nodo:

```bash
ls -l /mnt/glusterfs/
cat /mnt/glusterfs/archivo_replicado.txt
```
