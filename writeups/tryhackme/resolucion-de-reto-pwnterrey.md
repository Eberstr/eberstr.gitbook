---
hidden: true
---

# Resolución de reto pwnterrey

FLAG: FLAG{S1empr3\_0bserv4\_l4s\_C00ki3s}

login: \
usuario: pwnterrey \
contraseña: P@ssword123\
\
Al ver el panel de login lo primero que pensé fue en ver el código fuente de la página en por si había un comentario con credenciales o cualquier otra cosa de utilidad pero no hallé nada.

<figure><img src="../../.gitbook/assets/imagen (71).png" alt=""><figcaption></figcaption></figure>

Después intenté con usar algunas de las credenciales comunes como root:root, admin:admin, admin:contraseña, etc. pero no tuve éxito.\
También traté con inyecciones SQL, usé variaciones de los payloads más simples como  `' or 1=1-- -`, usé ambos tipos de comillas, diferentes formas de hacer comentarios, punto y coma. Ninguno de estos payloads me funcionó.

<figure><img src="../../.gitbook/assets/imagen (72).png" alt=""><figcaption></figcaption></figure>



Así que me puse a explorar un poco más la página. Hice un poco de exploración manual, busqué entrar directo a [https://challenge.pwnterrey.net](https://challenge.pwnterrey.net) pero me redirigía a de nuevo al login, también intente con rutas como /admin, /register, /signin, /security, /sitemaps hasta que se me ocurrió ver si en /robots.txt podría encontrar más rutas. Ahí me tope con la ruta **/backup.db.**

<figure><img src="../../.gitbook/assets/imagen (74).png" alt=""><figcaption></figcaption></figure>

Al descargar el archivo vi que había dos tablas, en una de las cuales se hallaba el usuario **pwnterrey** con una contraseña hasheada. Puse el hash en la pagina de hashes.com y dio como resultado **P@ssword123**\


<figure><img src="../../.gitbook/assets/imagen (75).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/imagen (76).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/imagen (78).png" alt=""><figcaption></figcaption></figure>

Con esas credenciales logré iniciar sesión.

<figure><img src="../../.gitbook/assets/imagen (79).png" alt=""><figcaption></figcaption></figure>

Haciéndole caso al titulo de la web "_Reto de Manipulación de Cookies_", con BurpSuite intenté hacer varios cambios a las cookies **PHPSESSID** y **role**, las deje vacías, use valores aleatorios, replique el mismo valor en ambas pero nada de eso funcionó.

<figure><img src="../../.gitbook/assets/imagen (80).png" alt=""><figcaption></figcaption></figure>

Después note que el valor de la cookie **role** (Z3Vlc3Q%3D) tenía un %3D lo cual me recordó a url-encode, así que en el decoder de burpsuite la decodifique, viendo que "%3D" equivale a "=", me di cuenta que la cookie estaba en base64, obtuve el valor de **guest.**\


<figure><img src="../../.gitbook/assets/imagen (81).png" alt=""><figcaption></figcaption></figure>

Por ultimo, realice el proceso inverso pero con la palabra "admin". Lo codifique en base64 y le agregue el %3D, quedando **YWRtaW4%3D.** Sustituí el valor de la cookie con el nuevo y conseguí la flag.\


<figure><img src="../../.gitbook/assets/imagen (82).png" alt=""><figcaption></figcaption></figure>
