---
title: Aplicación en Spring REST y Angular – 5ª Parte
author: El Profe
type: post
date: 2018-09-06T06:08:12+00:00
url: /2018/09/06/aplicacion-en-spring-rest-y-angular-5a-parte/
categories:
  - java
  - jdbc
  - jndi
  - jpa
  - spring

---
En esta ultima parte de la parte servidor hablare de como se crean los objetos que en el anterior articulo se devolvían.

Estos objetos eran del tipo _VentasAnoBean _y un _ArrayList_ de _VentasSemanaBean. _Para conseguirlos se llamaban a sendas funciones en  la clase **YagesBussines, **que eran las que construían esos objetos.

Empezare describiendo la clase

<pre>@Component
public class YagesBussines {  
    @Autowired
    CalendarioRepositorioService calendarioRepositorio;

    @Autowired
    HistVentasRepository histVentasRepository;

    @Autowired
    DataSource dataSource;        
  
     @Autowired
    private JdbcOperations jdbc;

    @Bean
    public JdbcTemplate jdbcTemplate() {
        return new JdbcTemplate(dataSource);
    }
.....</pre>

Lo primero, marcamos nuestra clase como _@Component_ para que Spring la cree y este disponible para inyectarla en otras clases. De otra manera nuestra clase _YagesController _ no podría usarla.

Ahora inyectamos los diferentes objetos que vamos a usar, incluyendo el objeto _jdbc_ que es de tipo **JdbcOperations .** Para que Spring pueda crear ese objeto debemos tener la función **jdbcTemplate**, que, usando la variable  _dataSource ._ devolverá un objeto tipo _JdbcTemplate_ el cual implementa el interface  _JdbcOperations_ para usarlo en la variable **jdbc**, ya que la función esta marcarla con la anotación @Bean y eso hará que Spring sepa de donde sacar un objeto tipo _JdbcOperations_.

  * ##### Función: getVentasAno

<pre>public VentasAnoBean <strong>getVentasAno</strong>(int ano) {
        List&lt;VentasMesBean&gt; cal = getKilosPorMes(ano);
        VentasAnoBean ventasAno = new VentasAnoBean();
        cal.forEach((v) -&gt; {
            ventasAno.addMes(v);
        });
       
        if (ventasAno.getKilosVentaAct() == 0) {
            throw new VentasNotFoundException(ano);
        }
        return ventasAno;
    }
    
</pre>

La función **getVentasAno**, simplemente llama a la función **getKilosPorMes, **la cual devuelve una lista de objectos tipo VentasMesBean (List<VentasMesBean>),  para después ir añadiéndolo al objeto tipo **VentasAno.**

Obsérvese que la función comprueba si el acumulado de kilos de venta,  a través de la función getKilosVentaAct(), es igual a cero. Si es cero, lanza una excepción del tipo **VentasNotFoundException**

Esa excepción se define en el paquete yages.yagesserver.bussines.

<pre>@ResponseStatus(HttpStatus.NOT_FOUND)
class VentasNotFoundException extends RuntimeException {
	private static final long serialVersionUID = 1L;

	public VentasNotFoundException(int ano) {
		this(ano, 0);
	}
	public VentasNotFoundException(int ano, int mes) {
		super("No encontradas ventas para año '" + ano + "'"+ (mes&gt;0?" y Mes: "+mes:""));
	}
	public VentasNotFoundException(String msgError,int ano, int mes) {
		super("Error al buscar ventas para año '" + ano + "'"+ (mes&gt;0?" y Mes: "+mes:"")+" \n Error: "+msgError);
	}
}</pre>

Lo destacable de esta clase es la anotación @ResponseStatus, la cual hará que cuando se lance, a través de una sentencia _throw, _sea capturada por Spring de tal manera que la respuesta a la peticion HTTP sea lo indicado, en este caso un HttpStatus.NOT_FOUND (el tipico 404: Página no encontrada). De esta manera si la consulta no devuelve ningún dato en vez de devolver un null, devolveremos un mensaje de error para que el cliente pueda tratarlo más fácilmente.

En otras palabras, cuando se lance una excepción del tipo _VentasNotFoundException_ el cliente recibirá un error del tipo 404 (NOT FOUND).

La función **getKilosPorMes**, realiza una serie de consultas a través de _jdbc. _

<pre>public List&lt;VentasMesBean&gt; getKilosPorMes(int ano) {                  
          List&lt;VentasMesBean&gt; cal =
                jdbc.query(
                "SELECT c.cal_ano,c.cal_mes , sum(hve_kilven) as kilosVenta,sum(hve_impven) as impVenta,sum(hve_impgan) as impGanancia "
                    + "FROM Calendario c,  histventas h "
                    + " where  c.cal_ano =  ?" 
                    + " and h.hve_fecini between c.cal_fecini and c.cal_fecfin "
                    + " and h.hve_fecfin between c.cal_fecini and c.cal_fecfin "
                    + " group by cal_ano,cal_mes "
                    + " order by c.cal_mes",  new Object[] { ano },
                (rs, rowNum) -&gt; new VentasMesBean(rs.getInt("cal_ano"),rs.getInt("cal_mes"),rs.getDouble("kilosVenta"),rs.getDouble("impVenta"),rs.getDouble("impGanancia"))
                        );
             List&lt;VentasMesBean&gt; calAnt =
                jdbc.query(
                "SELECT c.cal_ano,c.cal_mes , sum(hve_kilven) as kilosVenta,sum(hve_impven) as impVenta,sum(hve_impgan) as impGanancia "
                    + "FROM Calendario c,  histventas h "
                    + " where  c.cal_ano =  ?" 
                   + " and h.hve_fecini between c.cal_fecini and c.cal_fecfin "
                    + " and h.hve_fecfin between c.cal_fecini and c.cal_fecfin "
                    + " group by cal_ano,cal_mes "
                    + " order by c.cal_mes",  new Object[] { ano - 1},
                  (rs, rowNum) -&gt; new VentasMesBean(rs.getInt("cal_ano"),rs.getInt("cal_mes"),rs.getDouble("kilosVenta"),rs.getDouble("impVenta"),rs.getDouble("impGanancia"))
                        );
            
            
             cal.forEach((v) -&gt; {
                 for (VentasMesBean vAnt : calAnt) {
                     if (v.getMes() == vAnt.getMes()) {
                         v.setKilosVentaAnt(vAnt.getKilosVentaAct());
                         v.setImpVentaAnt(vAnt.getImpVentaAct());
                         v.setGananAnt(vAnt.getGananAct());
                     }
                 }
            });
            return cal; 
    }

</pre>

Obsérvese el uso intensivo de sentencias _lamba. _ Os recuerdo que tenéis un par de entradas sobre como este tema <a href="http://www.profesor-p.com/2018/08/23/usando-lambdas/" target="_blank" rel="noopener">aquí</a> y <a href="http://www.profesor-p.com/2018/09/04/expresiones-lamba-en-spring-jdbc-data-mejorando-la-explicacion/" target="_blank" rel="noopener">aquí</a>

Esta función consulta primero todas las ventas del año mandado, metiéndolas en un objeto _List<VentasMesBean>_, después hace la misma consulta sobre el año anterior al mandado, metiendo el resultado en otro objeto List<VentasMesBean>. Después recorre la primera lista a, con la función forEach, y si existe el mismo mes en el segundo objeto, establece los parámetros del año anterior en el primer objeto _VentasMesBean._ Por ultimo devuelve la lista de objetos (_List<VentasMesBean>_)

  * ##### Función: getDatosSemana

Esta función es la llamada cuando queremos buscar los datos de venta de un año y semana. Ofreciendo, semana por semana, los datos del periodo solicitado.

<pre>public ArrayList&lt;VentasSemanaBean&gt; getDatosSemana(int mes, int ano) {
        Calendario calAnterior = null;
        
        Optional&lt;Calendario&gt; calOpc = calendarioRepositorio.getCalendario(new CalendarioKey(mes, ano - 1));
        if (calOpc.isPresent()) {
            calAnterior = calOpc.get();
            try {
                ajustaFechas(calAnterior, ano - 1, mes);
            } catch (ParseException e) {
                throw new VentasNotFoundException(e.getMessage(), ano, mes);
            }
        }
......
    }</pre>

No es mi idea explicar paso a paso la lógica de esta función solo incidir en el uso de JPA, usando la clase **CalendarioRepositorioService**, que se limita a usar la clase generada _automagicamente_ por Spring, la cual   implementara el interface **CalendarioRepository**.

El código por si estáis interesados en curiosear como se buscan los datos lo tenéis en <a href="https://github.com/chuchip/yagesserver" target="_blank" rel="noopener">mi pagina de GitHub</a>

Y con esto termino las entradas, explicando la parte servidor de esta aplicación.

Espero que haya sido de utilidad y no os perdáis las siguientes sobre como hacer la parte cliente con Angular.

¡¡ Hasta pronto !!