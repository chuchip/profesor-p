# profesor-p
pagina web
<subsystem xmlns="urn:jboss:domain:datasources:5.0">
            <datasources>
                <datasource jta="true" jndi-name="java:jboss/datasources/ExampleDS" pool-name="ExampleDS" enabled="true" use-java-context="true"> <connection-url>jdbc:h2:mem:test;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE</connection-url> <driver>h2</driver> <security> <user-name>sa</user-name> <password>sa</password> </security> </datasource>
				<xa-datasource jndi-name="java:/jboss/datasources/rhpamDS_EJBTimer" pool-name="ejb_timer-EJB_TIMER" enabled="true" use-java-context="true"> 
					<xa-datasource-property name="URL">jdbc:sqlserver://bpm-db-svc</xa-datasource-property> 
					<driver>mssql</driver> 
					<transaction-isolation>TRANSACTION_READ_COMMITTED</transaction-isolation> 
					<xa-pool> 
						<min-pool-size>10</min-pool-size> 
						<max-pool-size>50</max-pool-size> 
					</xa-pool> 
					<security> 
						<user-name>****</user-name> 
						<password>****</password> 
					</security> 
				</xa-datasource>
				<datasource jta="true" jndi-name="java:/jboss/datasources/rhpamDS" pool-name="rhpam-RHPAM" enabled="true" use-java-context="true"> 		<connection-url>jdbc:sqlserver://bpm-db-svc;DatabaseName=PICASSO_BPM</connection-url> 
						<driver>mssql</driver> 
						<pool> 
							<min-pool-size>10</min-pool-size> 
							<max-pool-size>50</max-pool-size> 
						</pool> 
						<security> 
						<user-name>****</user-name> 
						<password>****</password> 
						</security> 
				</datasource>

                <drivers>
                    <driver name="h2" module="com.h2database.h2">
                        <xa-datasource-class>org.h2.jdbcx.JdbcDataSource</xa-datasource-class>
                    </driver>
                    <driver name="mysql" module="com.mysql">
                        <xa-datasource-class>com.mysql.jdbc.jdbc2.optional.MysqlXADataSource</xa-datasource-class>
                    </driver>
                    <driver name="postgresql" module="org.postgresql">
                        <xa-datasource-class>org.postgresql.xa.PGXADataSource</xa-datasource-class>
                    </driver>
                     <driver name="mssql" module="com.microsoft"><xa-datasource-class>com.microsoft.sqlserver.jdbc.SQLServerXADataSource</xa-datasource-class><driver-class>com.microsoft.sqlserver.jdbc.SQLServerDriver</driver-class></driver><!-- ##DRIVERS## -->
                </drivers>
            </datasources>
        </subsystem>
		
		
		
		
		
	<timer-service thread-pool-name="default" default-data-store="ejb_timer-EJB_TIMER_ds">                
	<data-stores>                    
		<file-data-store name="default-file-store" path="timer-service-data"relative-to="jboss.server.data.dir"/>                    <database-data-store name="ejb_timer-EJB_TIMER_ds" datasource-jndi-name="java:/jboss/datasources/rhpamDS_EJBTimer" database="mssql" partition="ejb_timer-EJB_TIMER_part" refresh-interval="60000"/>        
		<!-- ##DATASTORES## -->                
		</data-stores>            
	</timer-service>
