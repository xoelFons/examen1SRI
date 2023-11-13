1.- Explica metodos para abrir una shell en un contenedor ejecutandose.---

Puedes hacerlo por comando o de forma gráfica.
Para hacerlo por comandos, usas "docker exec -it [nombre-contenedor] bash"
Para hacerlo de forma gráfica, vas al contenedor en el que quieras abrir una consola, le damos click derecho y elegimos la opción: "Attach shell".



2.- En el contenedor anterior con que opciones tiene que haber sido arrancado para poder interactuar con las entradas y salidas.---

Para poder interactuar con el contenedor, tendremos que iniciarlo primero, por lo tanto tendremos que hacer un comando desde la terminal para arrancarlo. Por lo general, si el contenedor ya ha sido creado, habráq que usar el comando: "docker start [nombre-contenedor]". Con esto ya lo tendríamos funcionando.



3.- ¿Cómo sería un fichero docker-compose para que dos contenedores se comuniquen entre si en una red solo de ellos?---

El fichero docker debe tener unas características en específico, entre ellas tener en el apartado de red, unas ips que sean de la misma red, tanto en un contenedor como en otro. A parte de eso, tendremos que crear la red en sí, por lo que usaremos este comando:

docker network create --driver=bridge --subnet=172.28.0.0/16 --ip-range=172.28.5.0/24 --gateway=172.25.5.254 [nombre-red]

Esas son las 2 características más importantes a la hora de hacer un docker-compose con 2 contenedores conectados en una red solo para ellos.


4.- ¿Qué hay que añadir al fichero anterior para que un contenedor tenga la IP fija?

En el apartado de networks del docker-compose, tenemos que poner despúes del nombre de la red conectada a los contenedores, lo siguiente:

ipv4_address:[IP-a-poner]

Con esto ya tendríamos una ip fija para el contenedor, hay que tener en cuenta que hay que poner una IP válida, como el el siguiente ejemplo:

networks:
  asir_red_examen:
    ipv4_address: 172.28.5.1

Esto tendremos que ponerselo al apartado de network de cada contenedor.






