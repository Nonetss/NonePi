# **Instalación de Armbian en Orange Pi y Migración a NVMe**

Este tutorial describe cómo instalar **Armbian** en una **Orange Pi**, configurar el arranque desde un **NVMe** y establecer una **IP fija**.

---

## **1. Flashear la tarjeta SD**

Flashea la imagen de Armbian en una tarjeta SD utilizando **balenaEtcher** o el programa que prefieras.

---

## **2. Crear un usuario y conectarse por SSH**

### **Verificar la IP de la Orange Pi**

Ejecuta el siguiente comando desde otro dispositivo en la red local para encontrar la dirección IP asignada a la Orange Pi:

```sh
nmap -p 22 --open -T5 192.168.1.0/24
```

### **Eliminar clave SSH si hay un error de autenticación**

Si ya te habías conectado antes a otro dispositivo en la misma IP, podrías recibir un error de clave SSH. Para solucionarlo:

```sh
ssh-keygen -R 192.168.1.X
```

### **Conectarse por SSH**

```sh
ssh root@192.168.1.X
```

> Contraseña por defecto: `1234`  
> Al ingresar por primera vez, se te pedirá cambiar la contraseña y crear un usuario nuevo.

### **Actualizar el sistema**

```sh
apt update --fix-missing && apt upgrade -y
```

---

## **3. Configurar el montaje permanente del NVMe**

### **Crear el directorio de montaje**

```sh
mkdir -p /mnt/nvme
```

### **Verificar el identificador del NVMe**

```sh
lsblk
```

> Busca un dispositivo con nombre similar a `nvme0n1`.

### **Crear una partición en el NVMe**

```sh
fdisk /dev/nvme0n1
```

#### **Pasos en `fdisk`**:

1. Mostrar opciones: `m`
2. Borrar todas las particiones existentes (si las hay): `d` (repetir hasta que no queden particiones)
3. Crear una nueva partición: `n` (aceptar valores por defecto)
4. Guardar cambios: `w`

### **Verificar que la partición se creó correctamente**

```sh
lsblk
```

### **Formatear la nueva partición**

```sh
mkfs.ext4 /dev/nvme0n1p1
```

### **Montar temporalmente el NVMe**

```sh
mount /dev/nvme0n1p1 /mnt/nvme
```

### **Obtener el UUID del NVMe**

```sh
blkid
```

### **Añadir entrada en `/etc/fstab` para montaje permanente**

Edita el archivo `/etc/fstab`:

```sh
vim /etc/fstab
```

Añade la siguiente línea al final del archivo:

```
UUID=uuid-particion-nvme /mnt/nvme ext4 defaults 0 2
```

> Sustituye `uuid-particion-nvme` por el UUID obtenido en el paso anterior.

### **Reiniciar y verificar que el NVMe está montado**

```sh
sudo reboot
```

Después del reinicio, verifica que el NVMe está montado correctamente:

```sh
lsblk
```

---

## **4. Copiar el sistema de la SD al NVMe**

Ejecuta el siguiente comando para copiar todo el sistema de la SD al NVMe (excluyendo archivos y carpetas volátiles):

```sh
sudo rsync -aAXv / /mnt/nvme --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"}
```

---

## **5. Cambiar el disco de arranque a NVMe**

### **Obtener el UUID del NVMe**

```sh
blkid
```

### **Modificar el archivo de configuración de arranque**

Edita el archivo `/boot/armbianEnv.txt`:

```sh
vim /boot/armbianEnv.txt
```

> Usa vim, nano o el editor de texto que prefieras

Cambia la línea:

```
rootdev=UUID=uuid-particion-nvme
```

> Sustituye `uuid-particion-nvme` por el UUID obtenido anteriormente.

### **Reiniciar y verificar que el sistema arranca desde el NVMe**

```sh
sudo reboot
```

Tras el reinicio, verifica con:

```sh
lsblk
```

Si `/` (root) aparece en `nvme0n1p1`, el sistema está arrancando desde el NVMe.

---

## **6. Establecer IP fija**

Ejecuta:

```sh
nmtui
```

Sigue los pasos del vídeo para configurar una IP estática en la interfaz de red.

---

### **Verificación final**

Después de realizar todos los pasos:

- Verifica que `/` está en el NVMe con `lsblk`
- Asegúrate de que la IP fija funciona con `ip a`
- Confirma que el NVMe está montado automáticamente con `df -h`

---

# **Installing Armbian on Orange Pi and Migrating to NVMe**

This tutorial explains how to install **Armbian** on an **Orange Pi**, configure boot from **NVMe**, and set a **static IP**.

---

## **1. Flashing the SD Card**

Flash the Armbian image onto an SD card using **balenaEtcher** or any other preferred program.

---

## **2. Creating a User and Connecting via SSH**

### **Check the Orange Pi's IP Address**

Run the following command from another device on the local network to find the IP address assigned to the Orange Pi:

```sh
nmap -p 22 --open -T5 192.168.1.0/24
```

### **Remove SSH Key if Authentication Error Occurs**

If you previously connected to another device with the same IP, you might get an SSH key error. To fix it:

```sh
ssh-keygen -R 192.168.1.X
```

### **Connect via SSH**

```sh
ssh root@192.168.1.X
```

> Default password: `1234`  
> On the first login, you will be prompted to change the password and create a new user.

### **Update the System**

```sh
apt update --fix-missing && apt upgrade -y
```

---

## **3. Configure Persistent NVMe Mounting**

### **Create the Mount Directory**

```sh
mkdir -p /mnt/nvme
```

### **Check the NVMe Identifier**

```sh
lsblk
```

> Look for a device with a name similar to `nvme0n1`.

### **Create a Partition on the NVMe**

```sh
fdisk /dev/nvme0n1
```

#### **Steps in `fdisk`**:

1. Show options: `m`
2. Delete all existing partitions (if any): `d` (repeat until no partitions remain)
3. Create a new partition: `n` (accept default values)
4. Save changes: `w`

### **Verify the Partition was Created Successfully**

```sh
lsblk
```

### **Format the New Partition**

```sh
mkfs.ext4 /dev/nvme0n1p1
```

### **Temporarily Mount the NVMe**

```sh
mount /dev/nvme0n1p1 /mnt/nvme
```

### **Get the NVMe UUID**

```sh
blkid
```

### **Add an Entry in `/etc/fstab` for Permanent Mounting**

Edit the `/etc/fstab` file:

```sh
vim /etc/fstab
```

Add the following line at the end of the file:

```
UUID=nvme-partition-uuid /mnt/nvme ext4 defaults 0 2
```

> Replace `nvme-partition-uuid` with the UUID obtained in the previous step.

### **Reboot and Verify the NVMe is Mounted**

```sh
sudo reboot
```

After rebooting, check if the NVMe is mounted correctly:

```sh
lsblk
```

---

## **4. Copy the System from the SD Card to the NVMe**

Run the following command to copy the entire system from the SD card to the NVMe (excluding volatile files and folders):

```sh
sudo rsync -aAXv / /mnt/nvme --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"}
```

---

## **5. Change Boot Device to NVMe**

### **Get the NVMe UUID**

```sh
blkid
```

### **Modify the Boot Configuration File**

Edit the `/boot/armbianEnv.txt` file:

```sh
vim /boot/armbianEnv.txt
```

Replace the following line:

```
rootdev=UUID=nvme-partition-uuid
```

> Replace `nvme-partition-uuid` with the UUID obtained earlier.

### **Reboot and Verify the System Boots from NVMe**

```sh
sudo reboot
```

After rebooting, verify with:

```sh
lsblk
```

If `/` (root) appears under `nvme0n1p1`, the system is booting from the NVMe.

---

## **6. Set a Static IP Address**

Run:

```sh
nmtui
```

Follow the steps in the interface to configure a static IP address on the network interface.

---

### **Final Verification**

After completing all steps:

- Verify that `/` is on the NVMe with `lsblk`
- Ensure the static IP works with `ip a`
- Confirm the NVMe mounts automatically with `df -h`

---
