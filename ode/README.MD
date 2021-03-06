# Installiere Tomcat inkl. Ode und Axis2
- Download [Tomcat Server](https://tomcat.apache.org/download-90.cgi)
- Download [ODE War Distribution](http://ode.apache.org/getting-ode.html)
- Download [Axis2 War Distribution](https://axis.apache.org/axis2/java/core/download.html)
- War Distributionen nach `apache-tomcat-x.x.x/webapps` kopieren
- Server �ber `apache-tomcat-9.0.7-x.x.x/bin/startup.bat` starten
- Ode Web-Interface erreichbar �ber `localhost:8080/ode` 
- Axis2 Web-Interface erreichbar �ber `localhost:8080/axis2`
   - Admin Panel: admin, axis2   
- [Jersey Bundle](https://mvnrepository.com/artifact/com.sun.jersey/jersey-bundle/1.19.4) herunterladen und in den `lib` Ordnder der Axis2 Webapp kopieren (ist n�tig f�r den FeedbackService) und ggf. neustarten
- Tomcat Port unter `$TOMCAT_ROOT/conf/server.xml` �ndern (Connector port)
   
# BPEL Editor (Eclipse)
- Eclipse von [hier](https://www.eclipse.org/downloads/) downloaden
- Designer von Update Site `http://download.eclipse.org/bpel/site/` in Eclipse installieren
- Ode Server hinzuf�gen
   - Tomcat Home angeben
   - Ode Home angeben (`${tomcat_path}/webapps/ode`)
   - unter Run-Configurations ggf. noch folgende Libs unter User Entries hinzuf�gen
      - `${tomcat_path/bin/tomcat-juli.jar}`
	  - `${tomcat_path/bin/commons-deamon.jar}`
      - `${tomcat_path/bin/bootstrap.jar}`
   - Ode Server sollte sich jetzt auch �ber Eclipse starten und stoppen lassen

# Axis2 Service
- Services deployen
   - `*.aar` Datei in `apache-tomcat-x.x.x/webapps/axis2/WEB-INF/services` kopieren
   - Services sind standardm��ig hot deployable
- [Beispiel Services](https://github.com/apache/axis2-java/tree/329ac88407e73d63c1831c85a1228abc588d8f39/modules/samples)
- wenn Service deployed wurde, l�sst sich die entsprechende `wsdl` �ber das Axis2 Webinterface herunterladen
- wenn ein Parameter auf `locked` gestellt ist, verhindert dies die �nderung der Werte f�r alle Parameter (im Admin Panel)
- Beans m�ssen �ber Getter/Setter verf�gen, sonst wird die wsdl nicht korrekt erzeugt
   - Felder k�nnen �ber Eintrag in der `service.xml` ignoriert werden
   ```
	<parameter name="beanPropertyRules">
		<bean class="sample.Address" excludeProperties="street,number" />
		<bean class="sample.AddressParent" excludeProperties="zipcode" />
	</parameter>
	```

# AXISFeedbackService
- Build �ber `mvn build`
- `FeedbackService.aar` aus Target Ordner in den Axis2 Services Ordner kopieren
   - Context Uri und FeedbackService Uri k�nnen im Admin Panel oder in der `service.xml` (innerhalb der aar) angepasst werden.
- sicherstellen, dass [Feedback-Service](https://github.com/IoTUDresden/feedback-service) l�uft
- enth�lt auch einen einfachen Service, der das Senden von Commands an OpenHAB erm�glicht
- in der `service.xml` oder im Admin Panel l�sst sich �ber die Service Class in den Debug Modus schalten. Einfach `Debug` an den Klassennamen anh�ngen. Es werden dann nur Nachrichten geloggt und keine Aktionen mit dem FBS oder OH ausgef�hrt 

# Modellierung Tips
- [HelloWorld](http://www-inf.it-sudparis.eu/SIMBAD/courses/doku.php?id=teaching_assistant:bpel:deploying_hello_world_bpel_process_with_eclipse_bpel_designer_and_apache_ode)
- nur Content der im Ordner `bpelContent` liegt, wird mit Server synchronisiert (Rechtsklick auf Server und dann auf Publish)

# Ausf�hren und Testen
- Sicherstellen, dass die [Eclipse Java EE Developer Tools](https://marketplace.eclipse.org/user/nitind/listings) installiert sind
- Rechtsklick auf `*.wsdl` &rarr; Webservices &rarr; Test with Web Service Explorer
   - beim Durchnavigieren der Services ggf. mal auf Refresh im Navigator klicken, falls die entsprechende Seite nicht geladen wird
   
## Logging
- [Listener f�r Events](http://ode.apache.org/ode-execution-events.html)
- `io.tudresden.ode.logging` mit `mvn package` compilieren
- lib aus target Ordner zu `webapps/ode/WEB-INF/lib` kopieren
- `webapps/ode/WEB-INF/lib/conf/ode-axis2.properties` anpassen: 
   - `ode-axis2.event.listeners=io.tudresden.ode.logging.OdeLogging`
- `webapps\ode\WEB-INF\classes\log4j.xml` anpassen
  ```xml		
  <Logger name="io.tudresden.ode" level="debug" additivity="false">
    <AppenderRef ref="FILE"/>
  </Logger>
  ```
   
# BPEL cheat sheet
- f�r die automatisch erstellte wsdl muss f�r den Process die richtige Adresse unter ODE angegeben werden &rarr; `http://localhost:8080/ode/processes/<ProcessName>`
- in `deploy.xml` m�ssen die entsprechenden Ports f�r die Links angegeben werden

## Aktionen
- `invoke` anderen WS aufrufen
- `receive` empfange von anderen Client 
- `reply` antworte anderen Client
- `assign` Variablen zuweisen, manipulieren
- `wait` warten
- `terminate` kompletten Prozess beenden
- `throw` Fehler und Ausnahmen anzeigen

## Kontrollkonstrukte
- `sequence` Aktivit�ten, welche in geordneter Swquenz ausgef�hrt werden
- `flow` Aktivit�ten, welche parallel ausgef�hrt werden
- `switch` um Branches zu implementieren
- `while` Schleife
- `pick` um einen von mehreren Pfaden zu w�hlen

## andere Services einbinden
- andere Services werden `partner Web service` genannt
- `partner links` geben jeweils an, welcher web service genutzt wird
   - Rollen k�nnen auch f�r jeden Partnerlink angegeben werden (eigene Rolle und Rolle des Partner Links)
   - Rollen k�nnen f�r jeden PortType angegeben werden (definiert die eig. Funktion etwas genauer, z.B. callback)
- `port type` definiert eine Reihe von Methoden, die �ber den Web Service erreichbar sind. Kann man auch mit Interface vergleichen.
   - `input` und `output` sind die entsprechenden Parameter die der Service ben�tigt
- `binding` beschreibt, wie der port type zu nutzen ist (z.B. SOAP Action). HTTP Binding (z.B. f�r POST) sollte eig. auch unterst�tzt werden.
- in `deploy.xml` muss der entsprechende Axis2 Endpoint angegeben werden &rarr; `...HttpSOAP11...` (sicherstellen, dass f�r diesen Endpoint auch die Adresse angegeben ist)


## Links
- [http://www.oracle.com/technetwork/articles/matjaz-bpel1-090575.html](http://www.oracle.com/technetwork/articles/matjaz-bpel1-090575.html)
- [https://docs.oracle.com/cd/E19182-01/821-0539/ggbxz/index.html](https://docs.oracle.com/cd/E19182-01/821-0539/ggbxz/index.html)
- [http://www-inf.it-sudparis.eu/cours/WebServices/TP_BPEL/installationBPELODE.html](http://www-inf.it-sudparis.eu/cours/WebServices/TP_BPEL/installationBPELODE.html)
- [https://www.codeproject.com/Articles/33746/An-Example-Of-BPEL-invoking-WebService](http://www-inf.it-sudparis.eu/cours/WebServices/TP_BPEL/installationBPELODE.html)