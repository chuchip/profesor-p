---
title: Backend (II)
author: airec69
weight: 20
pre: "<b>o </b>"
type: post
date: 2018-09-03T16:22:55+00:00
url: /2018/09/03/aplicacion-en-spring-rest-y-angular-2-parte/
categories:
  - java
  - jpa
  - netbeans
  - rest
  - spring
  - spring boot
  - tomcat
tags:
  - java
  - jndi
  - json
  - rest
  - spring
  - spring boot
  - tomcat

---
En esta segunda parte voy a empezar a explicar como hacer la parte del servidor, usando, como dije en la [primera parte de este articulo][1] utilizare  JAVA 8, apoyándome en el framework Spring, versión 5.

Esta aplicación la he realizado con NetBeans 9, usando Tomcat 9 como servidor de aplicaciones. La podía haber realizado usando Spring Boot, lo que habría realizado más fácilmente y con menos configuración pero he querido hacerlo con Tomcat como ejercicio.

Esta el código fuente en: <a href="https://github.com/chuchip/yagesserver" target="_blank" rel="noopener">https://github.com/chuchip/yagesserver</a>

Estos serán los archivos usados en mi aplicación:

![](/img/2018/09/yages.png)

Las dependencias, aparte de las básicas de Spring, son tener las siguientes librerías , de H2 Database e Hibernate.

<pre>dependency&gt;
    &lt;groupId&gt;com.fasterxml.jackson.core&lt;/groupId&gt; &lt;!--  <strong><span style="font-size: 14pt;">Jackson  (JSON) --&gt;</span></strong>
    &lt;artifactId&gt;jackson-core&lt;/artifactId&gt;
    &lt;version&gt;2.9.6&lt;/version&gt;
&lt;/dependency&gt;
&lt;dependency&gt;
    &lt;groupId&gt;com.fasterxml.jackson.core&lt;/groupId&gt;
    &lt;artifactId&gt;jackson-databind&lt;/artifactId&gt;
    &lt;version&gt;2.9.6&lt;/version&gt;
&lt;/dependedncy&gt;
&lt;dependency&gt; &lt;!-- <strong>H2 DATABASE --&gt;</strong>
    &lt;groupId&gt;com.h2database&lt;/groupId&gt;
    &lt;artifactId&gt;h2&lt;/artifactId&gt;
    &lt;version&gt;1.4.197&lt;/version&gt;        
    &lt;scope&gt;runtime&lt;/scope&gt;
&lt;/dependency&gt;
&lt;dependency&gt;  &lt;!-- <strong>LOMBOK  --&gt;</strong>
    &lt;groupId&gt;org.projectlombok&lt;/groupId&gt;
    &lt;artifactId&gt;lombok&lt;/artifactId&gt;
    &lt;version&gt;1.18.2&lt;/version&gt;
    &lt;scope&gt;provided&lt;/scope&gt;
&lt;/dependency&gt;
 &lt;dependency&gt;   &lt;!-- <strong>HIBERNATE --&gt;</strong>
       &lt;groupId&gt;org.hibernate&lt;/groupId&gt; 
        &lt;artifactId&gt;hibernate-agroal&lt;/artifactId&gt; 
        &lt;version&gt;5.3.5.Final&lt;/version&gt; 
        &lt;type&gt;pom&lt;/type&gt; 
  &lt;/dependency&gt;</pre>

Para más detalles y otras librerías menores, mirar el fichero pom.xml del proyecto.

<a id="jpa"></a>En este ejemplo **no** uso  anotaciones XML, , todo se hace con anotaciones Java. Esta es el fichero con que se configura la base de datos.

<pre>@Configuration
@EnableJpaRepositories("yages.yagesserver")
@EnableTransactionManagement
public class JpaConfig {
   
    @Bean
    public DataSource dataSource() {

        EmbeddedDatabaseBuilder builder = new EmbeddedDatabaseBuilder();
        EmbeddedDatabase db = builder
                .setType(EmbeddedDatabaseType.H2) 
                .setName("yagesh2")
                .ignoreFailedDrops(true)
                .addScript("db/sql/create-db.sql")
                .addScript("db/sql/insert-data.sql")
                .generateUniqueName(false)     
                .build();
        return db;
    }
    @Bean
    public HibernateExceptionTranslator hibernateExceptionTranslator() {
		return new HibernateExceptionTranslator();
     }
    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory(DataSource dataSource) throws NamingException {
        LocalContainerEntityManagerFactoryBean em = new LocalContainerEntityManagerFactoryBean();
        em.setDataSource(dataSource());              
      
        em.setPackagesToScan(new String[]{"yages.yagesserver", "yages.yagesserver.dao"});
        em.setPersistenceUnitName("yages-server");
        em.setJpaVendorAdapter(jpaVendorAdapter());
        em.afterPropertiesSet();
        return em;
    }
       
    @Bean
    public JpaVendorAdapter jpaVendorAdapter() {
        HibernateJpaVendorAdapter jpaVendorAdapter = new HibernateJpaVendorAdapter();
        jpaVendorAdapter.setGenerateDdl(false);
        
        jpaVendorAdapter.setDatabase(Database.H2);
        jpaVendorAdapter.setShowSql(false);
        return jpaVendorAdapter;
    }    
   
    @Bean
    public PlatformTransactionManager transactionManager(EntityManagerFactory emf) {
        JpaTransactionManager transactionManager = new JpaTransactionManager();
        transactionManager.setEntityManagerFactory(emf);
        return transactionManager;
    }
}</pre>

La configuración es una típica para cualquier base de de datos. Lo único destacable es el uso de la clase **EmbeddedDatabaseBuilder ,** en la función **dataSource.** Esta clase es muy útil para crear una base de datos embebida del tipo H2, HSQL o incluso DERBY. Como se ve en el código el uso es tan simple como especificar el tipo de base de datos, y después añadir los scripts necesarios (en este caso el de creación de las tablas y el de carga de datos). Esto nos creara un objeto **EmbeddedDatabase** que implementa el interface **DataSource.** Observerse que  los ficheros create-db.sql  e insert-db.sql, contienen sentencias SQL puras.**
  
** 

<pre>create table calendario
(
	cal_ano int,  -- Año
	cal_mes int,  -- Mes
	cal_fecini date,  -- Fecha Inicial
	cal_fecfin date,  -- Fecha Final.
	constraint ix_calendario primary  key (cal_ano,cal_mes)
);
......
NSERT INTO calendario (cal_ano,cal_mes,cal_fecini,cal_fecfin) VALUES (2018,1,{d '2018-01-01'},{d '2018-01-27'});
INSERT INTO calendario (cal_ano,cal_mes,cal_fecini,cal_fecfin) VALUES (2018,2,{d '2018-01-28'},{d '2018-02-24'});
.......</pre>

Estos dos ficheros, deberán estar ubicados en la carpeta de **resources** (es decir, en el classpath)

Una vez que tenemos nuestro objeto **DataSource** ya solo resta crear nuestro EntityManager en la función **entityManagerFactory,** apoyándonos en las demás funciones de la clase.

Ahora creare las clases que representan los objetos de la base de datos, estos objetos están en la carpeta:

  1. ### Implementando nuestras tablas **como objetos.
  
** 

En el paquete **yages.yagesserver.model**  tendremos las siguientes clases:

<span style="background-color: #00ffff;"><strong>Calendario.java</strong> </span>

<pre>@Entity 
@Table(name="calendario")
@NoArgsConstructor
@Data	
public class Calendario implements Serializable{		
	    private static final long serialVersionUID = 1L;
            
            @Column(name = "cal_fecini",nullable = false)
	    @Temporal(javax.persistence.TemporalType.DATE)
	    private Date fechaInicio;
		
	    @Column(name = "cal_fecfin",nullable = false)
	    @Temporal(javax.persistence.TemporalType.DATE)
	    private Date fechaFinal;
	    
	    @Transient
	    private Date fechaFinalSemana;
	    	 
            @EmbeddedId
	    private CalendarioKey calendarioKey;
            
	    public Calendario(int ano,int mes,Date fecInicio,Date fecFinal)
	    {
	    	calendarioKey = new CalendarioKey(ano,mes);
	    	setFechaInicio(fecInicio);
	    	setFechaFinal(fecFinal);
	    }
}</pre>

<span style="background-color: #00ffff;"><strong>CalendarioKey.java</strong></span>

<pre>@Embeddable
@Data		 
@AllArgsConstructor
@NoArgsConstructor
public class CalendarioKey implements Serializable {		  
	    private static final long serialVersionUID = 2L;
            
            @Column (name="cal_mes", nullable=false)    
	    private int mes;
	    @Column (name="cal_ano", nullable=false)
	    private int ano;		      
}
</pre>

Como se puede ver son dos simples POJOS, con una serie de anotaciones para su uso con JPA.

Solo destacar el uso de que como la tabla Calendario tiene un indice multiple, haremos uso de la anotación **EmbebedId** de tal manera que la clase **Calendario**  sera una @**Entity** y la clase **CalendarioKey** , que tendra los campos mes y ano (año 😀 ) sera la que tenga el indice y estara marcada como  @Embeddable.

En el paquete **yages.yagesserver.dao** creare mi repositoio crud con el siguiente interface:

<pre>public interface CalendarioRepository  extends
           CrudRepository&lt;yages.yagesserver.model.Calendario, yages.yagesserver.model.CalendarioKey&gt;  { }</pre>

Para mas datos sobre este interface que nos hace la vida tan fácil, podéis mirar el [siguiente articulo de mi página][2].

En la clase **CalendarioRepositorioService** creo mi clase auxiliar que hace uso del repositorio.

<pre>@Service
@Transactional
public class CalendarioRepositorioService {
    
    @Autowired
    CalendarioRepository calendarioRepository;
    
    public Optional&lt;Calendario&gt;  getCalendario(CalendarioKey calendarioKey)
    {
        return calendarioRepository.findById(calendarioKey);        
    }
    public Iterable&lt;Calendario&gt;  getAllCalendarios()
    {
        return calendarioRepository.findAll();
    }
    
}</pre>

Observese como anoto la clase  con la etiqueta **@Service** y **@Transacional.** Despues inyectamos el interface **CalendarioRepository** que es el que usare en las dos unicas funciones que tiene esta clase.

La tabla **histVentas** es prácticamente igual que la tabla **calendario**, también con un indice múltiple y la configuración es básicamente la misma, por lo cual no creo necesario explicarla.

Y por hoy ya vale. En la próxima entrada, veremos como implementar las peticiones REST

¡¡ Hasta pronto!!

 [1]: http://www.profesor-p.com/2018/08/31/aplicacion-en-spring-y-angular/
 [2]: http://www.profesor-p.com/2018/08/25/jpa-hibernate-en-spring/