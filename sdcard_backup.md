# 📌 ¿Cómo clonar la tarjeta SD en Ubuntu?

El objetivo es crear una copia de seguridad de la tarjeta SD original que trae la PicoCalc, para poder restaurarla en otra SD en caso de fallo o pérdida de datos.

---

## Paso 1: Identificar el dispositivo SD

Primero, conectamos la tarjeta y listamos los discos:

```bash
lsblk -o NAME,SIZE,MODEL,TRAN,MOUNTPOINTS
```

Esto muestra todos los discos, sus tamaños y puntos de montaje.
En mi caso, la tarjeta apareció como **/dev/sdf**, con dos particiones:

```
/dev/sdf1  → 29.5G (FAT32)
/dev/sdf2  → 32M  (Linux)
```

---

## Paso 2: Desmontar la SD antes de copiar

Hay que asegurarse de desmontar las particiones para evitar errores:

```bash
sudo umount /dev/sdf1
sudo umount /dev/sdf2
```

---

## Paso 3: Crear la imagen de la SD (respaldo)

Usamos **ddrescue** en lugar de `dd`, porque maneja mejor los errores de lectura. Si no lo tienes en tu sistema, lo puedes instalar con:

```bash
sudo apt install gddrescue
```
Una vez instalado, ejecutamos el siguiente comando:

```bash
sudo ddrescue -d -D --force /dev/sdf ~/sdcard_picocalc.img ~/ddrescue.log
```
Donde:
* `/dev/sdf` → dispositivo de la SD
* `~/sdcard_picocalc.img` → archivo de imagen resultante
* `~/ddrescue.log` → log para continuar si hay errores

Esto genera la copia completa de la tarjeta en un archivo `.img`.

---

## Paso 4: Volcar la imagen en una nueva SD

Si además queremos hacer una copia de esta imagen en una tarjeta nueva, insertamos la SD de destino y verificamos su nombre con `lsblk` otra vez.
En mi caso, apareció nuevamente como **/dev/sdf** (sin montar).

Para restaurar, escribimos la imagen completa en el dispositivo:

```bash
sudo ddrescue -D --force ~/sdcard_picocalc.img /dev/sdf /tmp/ddrescue-write.log
sync
```

Esto sobrescribe la SD entera con el contenido exacto de la original.

---

## Paso 5: Verificar la copia

De manera opcional, podemos comprobar que los primeros bytes de la SD coinciden con la imagen:

```bash
IMG=~/sdcard_picocalc.img
SIZE=$(stat -c%s "$IMG")
sudo cmp -n "$SIZE" "$IMG" /dev/sdf && echo "Verificado OK"
```

Si vemos el mensaje **“Verificado OK”**, la copia es idéntica.

---

## Paso 6: Expulsar la SD de forma segura

Finalmente, expulsamos para que se actualice la tabla de particiones:

```bash
sudo udevadm settle
sudo eject /dev/sdf
```

---

# Resultado

La nueva tarjeta SD está lista para usarse en la PicoCalc, funcionando igual que la original.
Este procedimiento sirve para **clonar cualquier tarjeta SD** (Raspberry Pi, consolas retro, etc.).

