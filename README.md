1. Explica metodos para abrir una shell en un contenedor ejecutandose.---

Puedes hacerlo por comando o de forma gráfica.
Para hacerlo por comandos, usas "docker exec -it [nombre-contenedor] bash"
Para hacerlo de forma gráfica, vas al contenedor en el que quieras abrir una consola, le damos click derecho y elegimos la opción: "Attach shell".



2. En el contenedor anterior con que opciones tiene que haber sido arrancado para poder interactuar con las entradas y salidas.---

Para poder interactuar con el contenedor, tendremos que iniciarlo primero, por lo tanto tendremos que hacer un comando desde la terminal para arrancarlo. Por lo general, si el contenedor ya ha sido creado, habráq que usar el comando: "docker start [nombre-contenedor]". Con esto ya lo tendríamos funcionando.



3. ¿Cómo sería un fichero docker-compose para que dos contenedores se comuniquen entre si en una red solo de ellos?---


services:

  bind9:

    image: ubuntu/bind9
    container_name: asir2_bind9

    ports:

      - 53:53/tcp

      - 53:53/udp

    networks:

      bind9_subnet_exam:

        ipv4_address: 172.28.5.1

    volumes:

      - ./conf:/etc/bind

      - ./zonas:/var/lib/bind

    environment:

      - TZ=Europe/Paris

  cliente:

    container_name: asir_cliente

    image: alpine

    tty: true

    stdin_open: true

    networks:

      bind9_subnet_exam:

        ipv4_address: 172.28.5.2

networks:

  bind9_subnet_exam:

    external: true



Necesitaremos crear una red, por lo tanto usaremos el siguiente comando:


docker network create --driver=bridge --subnet=172.28.0.0/16 --ip-range=172.28.5.0/24 --gateway=172.25.5.254 [nombre-red]


4. ¿Qué hay que añadir al fichero anterior para que un contenedor tenga la IP fija?

En el apartado de networks del docker-compose, tenemos que poner despúes del nombre de la red conectada a los contenedores, lo siguiente:

ipv4_address:[IP-a-poner]

Con esto ya tendríamos una ip fija para el contenedor, hay que tener en cuenta que hay que poner una IP válida, como el el siguiente ejemplo:

networks:
  asir_red_examen:
    ipv4_address: 172.28.5.1

Esto tendremos que ponerselo al apartado de network de cada contenedor.


5.¿Que comando de consola puedo usar para saber las ips de los contenedores anteriores?

Con el comando "docker network inspect [nombre-red]". Luego para verificarlo, tendremos que buscar el nombre del contenedor en la salida.


6. ¿Cual es la funcionalidad del apartado "ports" en docker compose?

Como su nombre indica, su fucionalidad es para poner puertos al los contenedores. Lo que haces con ports es mapear que puertos tiene que usar el contenedor.


7. ¿Para que sirve el registro CNAME? Pon un ejemplo

Lo que hace CNAME es asociar dominios específicos con optos ya existentes. Esto se suele usar por ejemplo para que varios dominios tengan la misma dirección IP.

Un ejemplo rápido sería:


alias IN CNAME test

test IN A 172.28.5.4

www IN A 172.28.5.2

prueba IN A 172.28.5.7


Si hacemos un "dig CNAME @[ip-server] [nombre-dominio] [nombre-base-datos].int." debería devolvernos algo como alias.asirdatabase.int


8. ¿Como puedo hacer para que la configuración de un contenedor DNS no se borre si creo otro contenedor?

Para eso se utilizan los volúmenes, que son archivos con una configuración ya guardada que evita que se borre dicha configuración a la hora de borrar el contenedor, o se cree otro contenedor.

Para utilizar estos archivos hay que mapearlos dentro del docker-compose, en el apartado de volúmenes, como se puede ver en el siguiente ejemplo:


volumes:


- ./conf:/etc/bind

- ./zona:/var/lib/bind


9. Añade una zona tiendadeelectronica.int en tu docker DNS que tenga

www a la IP 172.16.0.1

owncloud sea un CNAME de www

un registro de texto con el contenido "1234ASDF"

Comprueba que todo funciona con el comando "dig"

Muestra en los logs que el servicio arranca correctamente


Para eso, en la base de datos localizada en zonas tenemos que poner lo siguiente:


$TTL 38400	; 10 hours 40 minutes

@		IN SOA	www.tiendaelectronica.int. some.email.address. (

				10000002   ; serial

				10800      ; refresh (3 hours)

				3600       ; retry (1 hour)

				604800     ; expire (1 week)

				38400      ; minimum (10 hours 40 minutes)

				)

@		IN NS	www.tiendaelectronica.int.

www		IN A		172.28.5.1

owncloud	IN CNAME	www

texto	IN TXT		"123456ASD"



Al hacer esto, ya tenemos la mitad del ejercicio hecho. Ahora solo queda iniciar el dokcer compose si no se ha hecho ya, abrir una consola, instalar los dnsutils y comprobar su funcionamiento con dig, Que saldrá algo parecido a esto:

; <<>> DiG 9.18.18-0ubuntu0.23.04.1-Ubuntu <<>> CNAME @172.28.5.1 owncloud.tiendaelectronica.int

; (1 server found)

;; global options: +cmd

;; Got answer:

;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 47082

;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:

; EDNS: version: 0, flags:; udp: 1232

; COOKIE: d234bf7e0a1320120100000065524cd593a31a565f2ec6a3 (good)
;; QUESTION SECTION:

;owncloud.tiendaelectronica.int.        IN      CNAME

;; ANSWER SECTION:

owncloud.tiendaelectronica.int. 38400 IN CNAME  www.tiendaelectronica.int.

;; Query time: 0 msec

;; SERVER: 172.28.5.1#53(172.28.5.1) (UDP)

;; WHEN: Mon Nov 13 17:20:37 CET 2023

;; MSG SIZE  rcvd: 105


Para ver los logs, tendremos que hacer un cat en /var/log/auth.log


10. Realiza el apartado 9 en la máquina virtual con DNS.


Para empezar instalaremos bind9 desde la terminal de la máquina virtual con el comando "sudo apt install bind9" 
. Luego comprobaremos su estado con el comando "systemctl status bind9" y debería estar corriendo.


Cuando esté todo listo, copiaremos todos el contenido de los archivos conf en sus respectivos ficheros, en este caso en /etc/bind, y la base de datos de tiendaelectronica.int lo tendremos que copiar en /var/lib/bind.


Cuando esté todo listo, probaremos su funcionamiento con el dig, como en la práctica anterior, nos tendría que dar lo mismo.


Ahora para poder hacer un dig desde el host a la máquina virtual, tenemos que poner esta ultima en adaprtador puente con el modo promiscuo.


Si tenemos la misma ip tanto dentro como fuera de la  máquina podremos hacer un dig a esta máquina sin problema.

La respuesta del dig sería algo así:


; (1 server found)

;; global options: +cmd

;; Got answer:

;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 22205

;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, 
ADDITIONAL: 1


;; OPT PSEUDOSECTION:

; EDNS: version: 0, flags:; udp: 1232

; COOKIE: a79df51fcd9337570100000065525348504829e2f604ab74 (good)

;; QUESTION SECTION:

;test.tiendaelectronica.int.    IN      A

;; ANSWER SECTION:
test.tiendaelectronica.int. 38400 IN    A       172.28.5.3


;; Query time: 0 msec

;; SERVER: 172.28.5.1#53(172.28.5.1) (UDP)

;; WHEN: Mon Nov 13 17:48:08 CET 2023

;; MSG SIZE  rcvd: 99


