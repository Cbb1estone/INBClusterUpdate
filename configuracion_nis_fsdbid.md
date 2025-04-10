# Implementación de NIS (Network Information Service) con Maestro (hanh), Esclavo (nyquist) y Clientes

Esta guía documenta paso a paso la instalación, configuración y prueba de un entorno NIS con:

- **Servidor Maestro (hanh)**: Publica los mapas NIS (usuarios, grupos, etc.)
- **Servidor Esclavo (nyquist)**: Replica los mapas y atiende peticiones como respaldo o para balanceo
- **Clientes**: Usan los mapas para autenticación

---

## [SERVIDOR MAESTRO - hanh] Configuración Inicial

### 1. Instalar NIS
```bash
sudo apt-get install nis make
```

### 2. Definir el dominio NIS
```bash
sudo nano /etc/defaultdomain
```
```
Fsdbid
```

### 3. Configurar NIS como maestro
```bash
sudo nano /etc/default/nis
```
```
NISSERVER=master
NISCLIENT=false
```

### 4. Editar Makefile para incluir UIDs/GIDs
```bash
sudo nano /var/yp/Makefile
```
Modificar:
```makefile
MINUID=1000
MINGID=1000
```

### 5. Reiniciar y generar mapas NIS
```bash
sudo service ypserv restart
sudo make -C /var/yp/
```

---

## [SERVIDOR ESCLAVO - nyquist] Configuración

### 1. Instalar NIS
```bash
sudo apt-get install nis make
```

### 2. Definir dominio
```bash
sudo nano /etc/defaultdomain
```
```
Fsdbid
```

### 3. Configurar como esclavo
```bash
sudo nano /etc/default/nis
```
```
NISSERVER=slave
NISCLIENT=true
```

### 4. Definir servidor maestro
```bash
sudo nano /etc/yp.conf
```
```
domain Fsdbid server hanh
```

### 5. Configurar NSS (Name Service Switch)
```bash
sudo nano /etc/nsswitch.conf
```
```
passwd:         files nis
group:          files nis
shadow:         files nis
```

### 6. Editar archivos para permitir acceso NIS
```bash
sudo nano /etc/passwd
+::::::

sudo nano /etc/group
+:x::

sudo nano /etc/shadow
+::::::::

```

### 7. Inicializar sincronización desde el maestro
```bash
sudo ypinit -s hanh
```

### 8. Reiniciar servicios
```bash
sudo service ypbind restart
sudo service ypserv restart
```

---

## [CLIENTE] Configuración

### 1. Instalar NIS
```bash
sudo apt-get install nis
```

### 2. Definir dominio NIS
```bash
sudo nano /etc/defaultdomain
```
```
Fsdbid
```

### 3. Configurar cliente
```bash
sudo nano /etc/default/nis
```
```
NISSERVER=false
NISCLIENT=true
```

### 4. Apuntar al servidor NIS
```bash
sudo nano /etc/yp.conf
```
```
domain Fsdbid server hanh
# o para esclavo:
domain Fsdbid server nyquist
```

### 5. Configurar nsswitch
```bash
sudo nano /etc/nsswitch.conf
```
```
passwd:    files nis
group:     files nis
shadow:    files nis
```

### 6. Agregar entradas +
```bash
sudo nano /etc/passwd
+::::::

sudo nano /etc/group
+:x::

sudo nano /etc/shadow
+::::::
```

### 7. Reiniciar servicio
```bash
sudo service ypbind restart
```

---

## Verificación (en cliente o esclavo)

### Ver servidor conectado
```bash
ypwhich
```

### Ver usuarios disponibles
```bash
getent passwd
```

### Ver mapas directamente
```bash
ypcat passwd
```

---

## [Opcional] Crear Usuario NIS (en hanh)

### 1. Crear usuario y asignar password
```bash
sudo useradd -d /home/user124 -s /bin/bash -m user124
sudo passwd user124
```

### 2. Ajustar permisos
```bash
sudo chown user124 /home/user124
sudo chgrp user124 /home/user124
```

### 3. Regenerar mapas
```bash
sudo make -C /var/yp/
```

---

## [Opcional] Autenticación SSH sin contraseña

### 1. Generar clave en cliente
```bash
ssh-keygen -b 4096 -t rsa
```

### 2. Copiar al servidor
```bash
ssh-copy-id user124@hanh
```

### 3. Verificar en el servidor
```bash
cat ~/.ssh/authorized_keys
```

---

## Balanceo y Redundancia entre hanh y nyquist

### Opción 1: Configuración múltiple en `/etc/yp.conf`
```bash
domain Fsdbid server hanh
domain Fsdbid server nyquist
```

### Opción 2: Repartir manualmente clientes
- La mitad apunta a hanh
- La otra mitad a nyquist

### Opción 3: DNS round-robin
```dns
nis.Fsdbid. A 192.168.0.10
nis.Fsdbid. A 192.168.0.20
```

En `/etc/yp.conf`:
```bash
domain Fsdbid server nis.Fsdbid
```

Esto reparte carga automáticamente según la respuesta DNS.

---

## Conclusión

Con esta arquitectura puedes tener:
- Redundancia: si hanh falla, nyquist sigue sirviendo.
- Balanceo: distribuir clientes entre hanh y nyquist.
- Escalabilidad: puedes añadir más esclavos si crece tu red.

Puedes complementar esta guía con scripts automatizados para que nuevos clientes se configuren más rápido.
