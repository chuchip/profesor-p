---
title: Mensajería con Kafka y Spring Boot
pre: "<b>o </b>"
author: El Profe
type: post
date: 2019-01-24T15:31:06+00:00
url: /2019/01/24/mensajeria-con-kafka-y-spring-boot/
featured_image: /img/2019/01/spring-kafka.jpg
categories:
  - docker
  - java
  - kafka
  - spring boot
tags:
  - docker
  - java
  - kafka
  - spring

---
**Kafka** es un programa de mensajería pensado para comunicaciones asíncronas. Básicamente la idea es que los clientes o **consumidores** se subscriben a un tipo de noticia o **topic** y cuando un emisor o **broker** manda un mensaje sobre ese **topic** Kafka lo distribuye a los **consumidores** suscritos.
<!--more-->
![Logotipo](/img/2019/01/spring-kafka-1024x335.jpg)
Para probar este programa deberemos tener un servidor funcionando con los _topics_ ya definidos . En la página <https://kafka.apache.org/quickstart> hay un manual rápido y muy claro de como levantar uno en apenas 10 minutos.

Hay una extensa documentación sobre **Kafka** en internet, por lo cual no voy a profundizar demasiado en su funcionamiento, ni instalación. No obstante, aclarare dos conceptos básicos de Kafka.

- Siempre se trabaja sobre <strong>topics</strong>. Poniendo un símil con la prensa escrita, un <strong>topic</strong> seria el periódico al que nos hemos subscrito. Solo recibiremos las ediciones (mensajes en Kafka) de ese periódico. Por supuesto una misma persona (suscriptor) mismo puede estar subscrito a muchos periódicos.

- Los subscritores siempre forman <strong>grupos</strong> , aunque en el grupo no haya mas que una sola subscritor.Kafka se encargara que un mensaje solo sea enviado a un subscritor de un grupo.Hay que pensar que Kafka es una tecnología enfocada a la nube y lo normal es que un mismo programa (normalmente un microservicio) este ejecutándose en varios servidores, para poder tener tolerancia a fallos. Sin embargo cuando un emisor envía un mensaje de un <strong><em>topic</em></strong> solo queremos que ese mensaje sea procesado por una de las instancias del servicio y no por todas ellas.

De esta manera suponiendo que el mensaje implicaría realizar un apunte en una base de datos central, solo se realizaría un único apunte y no uno por cada una de las instancias.</li> </ol> 

En esta entrada voy a explicar como mandar y recibir mensajes usando <strong>Spring Boot</strong> a un servidor Kafka. Además usaremos <strong>Docker</strong> para realizar ciertas pruebas.

El fuente de esta entrada están en <a href="https://github.com/chuchip/KafkaTest">https://github.com/chuchip/KafkaTest</a>
Empezaremos creando un proyecto <strong>Spring Boot</strong>, con los siguientes starters.
- Web
- kafka

El starter <strong>Web</strong> lo vamos a necesitar para las pruebas que haremos, pero no lo necesitaremos en un caso <em>real</em>.

### 1. Configuración

La configuración es muy simple. Solo tendremos que poner en el fichero <em>application.properties</em>, la dirección del servidor <strong>Kafka</strong> con el parámetro <strong>spring.kafka.bootstrap-server</strong>

```
message.topic.name=${topicname}
message.topic.name2=${topicname2}
message.group.name=${groupid}
spring.kafka.bootstrap-servers=kafkaserver:9092
spring.kafka.consumer.group-id=myGroup
```


Si tuviéramos varios servidores <strong>Kakfa</strong> como suele ser el caso en producción, los indicaríamos separándolos por comas. (server1:9092,server2:9092, server3:9093 &#8230;)

Con el parámetro <strong>spring.kafka.consumer.group-id</strong> podemos definir el grupo al que por defecto pertenecerán los <em>listeners</em> pero esto es configurable en cada uno de ellos y no es necesario.

Los demás parámetros los usaremos más adelante y son solo para poder realizar pruebas.

### 2. Enviando mensajes
Los mensajes los enviaremos desde la clase <code>KafkaMessageProducer</code>, la cual pongo a continuación.
```
@Component
public class KafkaMessageProducer {
	@Autowired
	private KafkaTemplate<String, String> kafkaTemplate;

	@Value(value = "${message.topic.name:profesorp}")
	private String topicName;	
	
	public void sendMessage(String topic,String message) {
		if (topic==null || topic.trim().equals(""))
			topic=topicName;
		ListenableFuture<SendResult<String, String>> future = kafkaTemplate.send(topic, message);
		future.addCallback(new ListenableFutureCallback<SendResult<String, String>>() {
			@Override
			public void onSuccess(SendResult<String, String> result) {
				System.out.println("Sent message=[" + message + "] with offset=[" + result.getRecordMetadata().offset() + "]");
			}
			@Override
			public void onFailure(Throwable ex) {System.err.println("Unable to send message=[" + message + "] due to : " + ex.getMessage());
			}
		});
	}	l
}
```
    
 Lo primero será pedir a <strong>Spring</strong> que nos inyecte un objeto tipo <strong>KafkaTemplate</strong> .

 El <em>topic</em> por defecto, sobre el que enviaremos los mensajes lo definimos en la variable <strong>topicname</strong> que, por defecto, tendrá el valor de <strong>message.topic.name</strong> establecida en el fichero <em>properties</em> de <strong>Spring Boot</strong>

 En la función <strong>sendMessage</strong> sobre el <em>topic</em> mandado mandaremos el mensaje deseado.

 Para ello crearemos un <strong>ListenableFuture</strong> a partir de <strong>kafkaTemplate</strong>. De esta manera la llamada al servidor de Kafka será asíncrona. Para hacerla simplemente usaremos la función <strong>addCallback</strong> de la clase <strong>ListenableFuture</strong>, pasándole el interface <strong>ListenableFutureCallback</strong>.

 La función <strong>onSuccess</strong> será ejecutada si todo va bien y la función <strong>onFailure</strong> en caso de error.
 
### 3. Recibiendo mensajes

Los mensajes los recibiremos en la clase <code>KafkaTestListener</code>

```
@Component
public class KafkaTestListener {
	 	
	  @KafkaListener(topics = "${message.topic.name:profesorp}", groupId = "${message.group.name:profegroup}")
 public void listenTopic1(String message) {
     System.out.println("Recieved Message of topic1 in  listener: " + message);
 }
	
	   @KafkaListener(topics = "${message.topic.name2:profesorp-group}", groupId = "${message.group.name:profegroup}")
	   public void listenTopic2(String message) {
		   System.out.println("Recieved Message of topic2 in  listener "+message);
	   }
}
```

En la función <strong>listenTopic1</strong> con la etiqueta <strong>@KafkaListener</strong>, definiremos el <em>topics</em> , en plural pues pueden ser varios, que queremos escuchar. En este caso escucharemos los definidos en la variable <strong>message.topic.name</strong> del fichero <em>properties</em> de <strong>Spring Boot</strong>. Si esa variable no estuviera definida, tendrá el valor <code>profesorp</code>. Además especificamos el <strong>grupo</strong> al que pertenece el <em>listener</em>. Recordar si no lo definimos cogerá el que hayamos configurado con el parametro <strong>spring.kafka.consumer.group-id</strong>

En la función <strong>listenTopic2</strong> recibiremos los mensajes del <em>topic</em> <strong>message.topic.name2</strong>.

### Probando la aplicación

En la clase <code>KafkaTestController</code>he puesto un controlador <strong>REST</strong> muy sencillo que escucha en <strong>/add/{topic}</strong> . Acepta peticiones <strong>POST</strong> y lo único que hace es llamar a <strong>kafkaMessageProducer.sendMessage</strong> mandando un mensaje al servidor <strong>Kafka</strong>. Este es el código.


```
@RestController
public class KafkaTestController {
	@Autowired
	KafkaMessageProducer kafkaMessageProducer;
	
	@PostMapping("/add/{topic}")
	public void addIdCustomer( @PathVariable String topic,@RequestBody  String body)
	{
		kafkaMessageProducer.sendMessage(topic,body);
	}
}
```

Para hacer las pruebas he creado el fichero <strong>jar</strong> a través de maven (<strong><em>maven package</em></strong>) y he creado este pequeño script al que he llamado <code>javaapp.sh</code> que lo ejecuta:

```
java -Dtopicname=$TOPICNAME  -Dtopicname2=$TOPICNAME2 -Dgroupid=$GROUPID -jar  ./kafkatest.jar
```

Como variables de entorno habremos definido TOPICNAME,TOPICNAME2 y GROUPID.


```
export TOPICNAME="mytopic_1"
export TOPICNAME2="mytopic_2"
EXPORT GROUPID="profe_group"
```

Recordar que tenemos que tener creados los topics en el servidor de kafka. Eso se puede hacer con el comando


```
> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic mytopic_1
> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic mytopic_2
```

Para comprobar que están creados los <em>topics</em> podemos ejecutar el comando

```
bin/kafka-topics.sh --list --zookeeper localhost:2181
```

Una vez tengamos la aplicación corriendo en un terminal, abrimos otro, y ejecutamos la sentencia:


```
> curl --request POST localhost:8080/add/mytopic_1 -d "Mensaje para topic1 "
```  

La cual hace una petición POST a la URL <strong>localhost:8080/add/topic2</strong> . En el cuerpo de esta petición ira el texto <em>&#8220;Mensaje para topic profegroup&#8221;</em>

La salida que veremos en nuestro programa será la siguiente
```
Sent message=[Mensaje+para+topic+mytopic_1=] with offset=[13]
Recieved Message of mytopic_1 in  listener:  Mensaje+para+topic1=
```

Con lo cual vemos, como se ha enviado y luego recibido correctamente en el **topic1**.
 
Si ahora ejecutamos la sentencia:

```
curl --request POST localhost:8080/add/mytopic_2 -d "Mensaje para topic2"
```


Observaremos la siguiente salida:


```
Sent message=[Mensaje+para+topic+mytopic_2=] with offset=[32]
Recieved Message of topic1 in  listener: Mensaje+para+topic+topic2=
```  

###  Dockerizando la aplicación


Para probar la aplicación en un entorno más real vamos a dockerizar nuestra aplicación.

El fichero <code>DockerFile</code> sera el siguiente:
     
```
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ENV JAR_FILE kafkatest.jar
ENV TOPICNAMER rofesorp
ENV TOPICNAME2 profegroup
ENV GROUPID profe_group
COPY $JAR_FILE /tmp/kafkatest.jar
COPY javaapp.sh /tmp/
EXPOSE 8080
ENTRYPOINT "/tmp/javaapp.sh"
```


En el directorio donde este el fichero <strong>DockerFile</strong> deberemos tener también <strong>javaapp.sh</strong> y <strong>kafakatest.jar</strong>



Creamos la imagen con el comando:

```
docker build -t kafkatest .
```
     

Antes de ejecutarlo debemos asegurarnos que nuestro servidor de Kafka este bien configurado.

Una vez un consumidor llama al servidor, lo primero que hace este servidor es comunicarle al cliente la dirección donde esta el servidor <em>lider</em> con el que debe comunicarse. Normalmente esto no es problema, pero cuando la red es un poco compleja, como en el caso de Docker o máquinas virtuales hay que asegurarse que el nombre del <em>lider</em> que suministra Kafka, es accesible por el <em>consumidor</em>.
  
Como Docker crea siempre un interface virtual en la dirección <strong>172.17.0.1</strong>, añadiremos esta IP al fichero <strong>/etc/hosts</strong>, añadiéndole la línea


```
172.17.0.1 kafkaserver
```

Y en el fichero <strong>config/server.properties</strong> definiremos las siguientes variables:
     

```
listeners=PLAINTEXT://kafkaserver:9092
advertised.listeners=PLAINTEXT://kafkaserver:9092
```


La primera línea especifica donde debe escuchar el servidor de Kafka y la segunda el nombre que mandara al <strong>consumidor</strong> cuando pregunte por el <em>lider</em> con el que debe conectarse.

Reiniciamos el servidor de Kafka, y ejecutamos el contenedor docker con la siguiente sentencia:

```
docker run -i -t  --name=kafkatest1 -p 8880:8080  --hostname kafkatest1   -e "TOPICNAME=my_topic1"  -e "TOPICNAME2=my_topic2" -e "GROUPID=profe_group"  --add-host kafkaserver:172.17.0.1  kafkatest
```

En otro terminal ejecutamos el mismo comando cambiando el nombre del contenedor y mapeando el puerto 8080 al 8881
   
```
docker run -i -t  --name=kafkatest2 -p 8881:8080  --hostname kafkatest2   -e "TOPICNAME=my_topic2"  -e "TOPICNAME2=my_topic2" -e "GROUPID=profe_group"  --add-host kafkaserver:172.17.0.1  kafkatest
```

Si en un tercer terminal ejecutamos la sentencia
```
curl --request POST localhost:8880/add/my_topic1 -d "Mensaje para topic numero 1"
```

Observaremos que solo uno de los dos programas recibe el mensaje de Kafka pues ambos pertenecen al mismo grupo (<strong>profe_group</strong>)

Abriendo un nuevo terminal, volvemos a lanzar una nueva instancia de docker, especificando <strong>otro_grupo</strong>.

```
docker run -i -t  --name=kafkatest3 -p 8881:8080  --hostname kafkatest2   -e "TOPICNAME=my_topic1"  -e "TOPICNAME2=my_topic2" -e "GROUPID=otro_grupo"  --add-host kafkaserver:172.17.0.1  kafkatest
```

Veremos que la anterior petición <strong>curl</strong> hace que los mensajes sean recibidos en dos instancias en <strong>docker</strong>. En una de las dos, que tiene el grupo <strong>profe_group</strong> y en la que tiene el grupo <strong>otro_grupo</strong>

Por supuesto este programa se puede mejorar mucho, pues no hemos utilizado serialización para recibir objetos directamente, ni hemos hablado de las particiones de Kafka, ni del tratamiento de errores, seguridad, etc, pero como una pequeña introducción al mundo de Kafka y la mensajería. Como siempre se agradecerá cualquier comentario aquí o en mi cuenta de Twitter @chuchip.

¡¡Hasta la próxima clase!!
