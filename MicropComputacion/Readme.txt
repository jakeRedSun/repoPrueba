cd C:/Users/jazon/Desktop/micro1CompuNube/MicropComputacion

1. Levantar las máquinas virtuales usando vagrant up, consul se instala
	automaticamente mediante shell.

2. Loguearse en el servidor
	vagrant ssh servidorCluster

3. usar el siguiente código en el servidor del cluster

consul agent \
  -server \
  -ui \
  -bootstrap-expect=1 \
  -node=Cluster-Server \
  -bind=192.168.100.2 \
  -client=0.0.0.0 \
  -data-dir=/tmp/consul \
  -config-dir=/etc/consul.d

4. Para verificar el cluster abrir http://192.168.100.2:8500/ui/dc1/services

5. En otra consola Loguearse en el nodo 1
	vagrant ssh nodo1Cluster

6. usar el siguiente código en el nodo 1 del cluster, esto levantará el nodo.
	
consul agent \
  -node=Nodo-1 \
  -bind=192.168.100.3 \
  -enable-script-checks=true \
  -data-dir=/tmp/consul \
  -config-dir=/etc/consul.d

7. En otra consola logueamos en el servidorCluster y verificamos los miembros del cluster

consul members

8. Realizamos la comunicación agregando el cluster del nodo 1

consul join 192.168.100.3

9. En otra consola Loguearse en el nodo 2
	vagrant ssh nodo2Cluster

10. usar el siguiente código en el nodo 2 del cluster, esto levantará el nodo.
	
consul agent \
  -node=Nodo-2 \
  -bind=192.168.100.4 \
  -enable-script-checks=true \
  -data-dir=/tmp/consul \
  -config-dir=/etc/consul.d

11. Realizamos la comunicación agregando el cluster del nodo 2

consul join 192.168.100.4

12. En otra consola loguearse en nodo 1
	vagrant ssh nodo1Cluster

13. clonar github
	git clone https://github.com/omondragon/consulService

14. ir a
	cd consulService/app

15. instalar consul
	npm install consul

16. Instalar xpress
	npm install express

17. editar index.js y cambiar la ip por 192.168.100.2

18. En diferentes consolas, desde el directorio de trabajo ~/consulService/app corra 
	node index.js 3000 

19. Repetir punto 12 a 18 para el nodo 2

20. Para comprobar la respuesta se ingresa en el navegador con la ip y servicio
	http://192.168.100.3:3000/
	http://192.168.100.4:3000/

21. SOLO UNA VEZ Desde el servidor, modificar la configuracion Haproxy
	sudo vim /etc/haproxy/haproxy.cfg

22. SOLO UNA VEZ pegar al final del archivo lo siguiente 
	
frontend stats
   bind *:1936
   mode http
   stats uri /
   stats show-legends
   no log

frontend http_front
   bind *:80
   default_backend http_back

backend http_back
    balance roundrobin
    server-template mywebapp 1-10 _mymicroservice._tcp.service.consul resolvers consul    resolve-opts allow-dup-ip resolve-prefer ipv4 check

resolvers consul
    nameserver consul 127.0.0.1:8600
    accepted_payload_size 8192
    hold valid 5s

23. SOLO UNA VEZ Recargar haproxy para que tome en cuenta los cambios
	service haproxy reload

24. SOLO UNA VEZ Modificar los errores yendo a 
	/etc/haproxy/errors

25. SOLO UNA VEZ El error 503 es cuando no hay servidores disponibles
	sudo vim 503.http

26. Iniciar el Haproxy
	systemctl start haproxy

27. hacer prueba con la ip del servidor
	192.168.100.2

28. Se pueden observar estadísticas del haproxy entrando a 
	http://192.168.100.2:1936/