# Maquina Master

Lo primero que deberemos de hacer será escanear los posibles puertos que se encuentre abiertos en la máquina:

```shell 
nmap -p- --open -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
`
