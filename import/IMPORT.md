# Einleitung
Der Samply.Store ist als Webservice implementiert und hat noch keine grafische Oberfläche. Daten werden als XML modelliert und über eine REST-Schnittstelle übergeben. 

# Systemvoraussetzungen
* Docker runnable
* Intel Xeon E5-2630 oder besser, 64 GB RAM oder mehr, 500 GB Festplattenspeicher (je nach Datenvolumen) (unverbindliche Empfehlung)
* Netzwerkkommunikation/Firewall: Ausgehend http und https. Proxies werden unterstützt. Keine VPN oder eingehende Ports notwendig.

# Datenmodell
![Datamodel](import/datamodel-sample.png)

# REST-Schnittstelle
Beim ersten Start des Tomcat-Servers, der REST-Schnittstelle, wird in der Datenbank das Schema angelegt. Zudem wird ein User angelegt:

* Nutzer: "local_admin"
* Passwort: "local_admin"

Für einen Datenimport in den Store, schickt man eine entsprechende XML-Datei als POST-Request an die REST-Schnittstelle: http://localhost:8080/gba-store/import. Im Header muss der Nutzer mit seinem Passwort als "Basic Auth" übermittelt werden.

Die technischen Mittel sind frei wählbar, z.B. besitzt Talend Open Studio ein Rest-Interface. Für den manuellen (ersten) Test empfehlen wir Postman oder Insomnia.

Das XML wird beim Import gegen eine XSD-Datei validiert. Dieses XSD kann man per GET-Request von der REST-Schnittstelle erhalten:

http://localhost:8080/gba-store/importXSD

Zusätzlich werden standardmäßig die Einträge gegen das MDR validiert. Wenn Sie einen Proxy verwenden, so müssen Sie diesen beim Start angeben.

Alternativ kann auch ein "curl"-Befehl verwendet werden. Für Windows muss dieser nachinstalliert werden (falls nach Visual C gefragt wird: vc_redist.x64.exe), Bei Ubuntu und Debian muss curl ggf. per (sudo apt-get install curl) installiert werden.

Export der XSD
```
curl --output importXSD.xsd -X GET http://localhost:8080/gba-store/importXSD
```

Import einer XML
```
curl -H "Content-Type:application/xml" -d @export.xml http://local_admin:local_admin@localhost:8080/gba-store/import
```

# Struktur der XML 
Ein Beispiel für die Importdatei gibt es ganz unten.

Das äußere Tag ist ein \<store xmlns="http://schema.samply.de/store">-Tag. 

Innen kann jede Entität mit einem Tag entsprechend ihres Namens angegeben werden. Zusätzlich werden immer die id und ggf. Fremdschlüssel auf andere Entitäten im XML übermittelt. Die eigentlichen Felder werden durch ein \<genericAttribute key="urn:mdr16:dataelement:23:1">-Tag übermittelt, wobei der "key" für das Feld im MDR steht, dessen Wert immer ein String ist.

Beispielhaft sieht ein Eintrag für eine Collection (zu einer Biobank mit id="Bio11") wie folgt aus:

```
<collection id="Col22" biobankId="Bio11"> 
<genericAttribute key="urn:mdr16:dataelement:2:1">PD Dr. Schulz</genericAttribute>
</collection>
```

Derzeit existieren die Entitäten: Biobank, Collection, Sample, SampleContext, Event und Donor. 


## IDs
Beim Import spielen die ID's des XML-Files eine entscheidende Rolle. Sie sind die Werte die in den Tags der einzelnen Entitäten angegeben werden - als id (id) oder Fremdschlüssel (z.B. biobankId). Sie werden als Key/Value-Pair im JSON-Format in der Datendank gespeichert. Mit dem festen Key: "samply_store_unique_name". (Dieser Key wird in zukünftigen Releases umbenannt in "local_id")

Samply.Store generiert für jeden Datensatz automatisch eine laufende Nummer (id in der Datenbank). Sie wird nur intern verwendet (z.B. für Fremdschlüsselbeziehungen) und tritt nach außen nicht in Erscheinung. 

Wichtig: Die Id's aus dem XML (id) und Fremdschlüssel (biobankId) beziehen sich auf die Einträge im XML. Durch diese werden die 1-n-Beziehungen zwischen den Entitäten festgelegt und anhand der XSD wird überprüft, ob alle Fremdschlüssel valide sind. Es findet jedoch keine inhaltliche Validierung statt. 

## Überprüfen
Derzeit ist keine Möglichkeit implementiert, um die Daten im Store zu überprüfen. Man kann die Überprüfung aber direkt auf der PostgreSQL-Datenbank vornehmen. Einfach geht dies mittels des Tools "pgAdmin", dieses ist nicht in der Installation inbegriffen und muss manuell installiert werden.

Nutzt man pgAdmin, muss man sich zuerst mit dem bei der Installation erstellen Postgres-Server verbinden: Rechtsklick → Create Server. Unter dem Reiter Connection muss angegeben werden:

* Host Name/address: localhost
* Port: 5434
* Username: samplystore
* Passwort: samplystore



## Update-Verhalten
Wenn eine Entität bereits im Store gespeichert ist und abermals mit einem XSD geschickt wird, so werden die Felder wie folgt aktualisiert:

Wenn ein Attribut bereits vorhanden ist, wird es durch die neu übermittelten Werte ersetzt (ein oder mehrere)
Wenn ein Attribut noch nicht vorhanden ist, wird es hinzugenommen
Wenn ein Attribut vorhanden war aber nicht neu mitgeschickt wird, bleibt es erhalten
Die Entität wird anhand ihres "samply_store_unique_name" eindeutig identifiziert (siehe oben).


Beispiel:

Eine Biobank wurde mit den Attributen importiert:

key="xyz:def:ghi:1:5"       >Frankfurt
key="xyz:def:ghi:2:5"       >Lübeck
key="xyz:def:ghi:2:5"       >Hamburg
key="xyz:def:ghi:3:5"       >Münster
 
 
Die Biobank wird mit weiteren Attributen nochmal importiert:

key="xyz:def:ghi:1:5"       >Heidelberg
key="xyz:def:ghi:2:5"       >Lübeck
key="xyz:def:ghi:4:5"       >Hannover
 

Ergebnis

key="xyz:def:ghi:1:5"       >Heidelberg
key="xyz:def:ghi:2:5"       >Lübeck 
key="xyz:def:ghi:3:5"       >Münster 
key="xyz:def:ghi:4:5"       >Hannover



# Importdatei (Beispiel) 
```
<?xml version="1.1" encoding="UTF-8"?>
<store xmlns="http://schema.samply.de/store">

<biobank id="Bio11">
<genericAttribute key="urn:mdr16:dataelement:2:1">Prof. Andrea Maria Musterfrau</genericAttribute>
<genericAttribute key="urn:mdr16:dataelement:3:1">http://www.google.de</genericAttribute>
</biobank>

<collection id="Col21" biobankId="Bio11">
<genericAttribute key="urn:mdr16:dataelement:5:1">Brustkrebsstudie</genericAttribute>
</collection>

<collection id="Col22" biobankId="Bio11">
<genericAttribute key="urn:mdr16:dataelement:5:1">Herz-Kreislauf-Prävention für Raucher</genericAttribute>
</collection>

<sampleContext id="CON31" donorId="SNS51">
<genericAttribute key="urn:mdr16:dataelement:20:1">liquid_sample_context</genericAttribute>
</sampleContext>

<sampleContext id="CON32" donorId="SNS51">
<genericAttribute key="urn:mdr16:dataelement:20:1">liquid_sample_context</genericAttribute>
</sampleContext>

<sample id="S43" sampleContextId="CON32" collectionId="Col22">

<!--Entnahmedatum-->
<genericAttribute key="urn:mdr16:dataelement:12:1">01.01.2002</genericAttribute>
</sample>

<sample id="S44" sampleContextId="CON32" collectionId="Col22">
<!--Entnahmedatum-->
<genericAttribute key="urn:mdr16:dataelement:12:1">01.01.2003</genericAttribute>
</sample>

<event id="EvN61" donorId="SNS51">

<genericAttribute key="urn:mdr16:dataelement:26:1">30.05.2002</genericAttribute>
<!--Diagnosedatum-->
<genericAttribute key="urn:mdr16:dataelement:27:1">A15.1</genericAttribute>
</event>

<event id="EvN62" donorId="SNS51">
<genericAttribute key="urn:mdr16:dataelement:26:1">01.05.2003</genericAttribute>
<!--Diagnosedatum-->
<genericAttribute key="urn:mdr16:dataelement:27:1">I02.0</genericAttribute>
</event>

<donor id="SNS51">
<genericAttribute key="urn:mdr16:dataelement:22:1">24.12.1950</genericAttribute>
<!--Geburtsdatum-->
<genericAttribute key="urn:mdr16:dataelement:23:1">female</genericAttribute>
<!–Geschlecht biologisch-->
</donor>

</store>
```