# MÁQUINA FIRST HACKING

Lo primeroq que tendremos que hacer será realizar un escaneo de puertos a la máquina víctima usando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/First%20Hacking/foto.jpg)

Unicamente esta abierto el puerto 21, y no tenemos usuario ni nada, sin embargo podemos ver la versión, podemos buscar si esta versión tiene alguna vulnerabilidad con searchsploit:

```shell
searchsploit sftpd 2.3.4
```

```shell
searchsploit -m unix/remote/49757.py
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/First%20Hacking/foto2.jpg)

Una vez descargado el exploit lo ejecutamos:

![ROOT](https://github.com/Isma-yo/photos/blob/main/First%20Hacking/foto3.jpg)

Y ya somos root!

