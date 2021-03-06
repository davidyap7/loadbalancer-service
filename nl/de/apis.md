---

copyright:
  years: 2017
lastupdated: "2017-08-21"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}

# API-Referenz
Die API (Application Programming Interface) von SoftLayer® ist die Entwicklungsschnittstelle, die Entwicklern und Systemadministratoren die direkte Interaktion mit dem SoftLayer-Back-End-System ermöglicht.
{:shortdesc}

Die SoftLayer-API (SLAPI) unterstützt viele Funktionen im Kundenportal; dies bedeutet in der Regel, dass eine Interaktion, die im Kundenportal möglich ist, auch in der API ausgeführt werden kann. Da Sie mit allen Teilen der SoftLayer-Umgebung in der API programmgesteuert interagieren können, können Sie die API zum Automatisieren von Tasks verwenden. 

Die SoftLayer-API ist ein Remote Procedure Call-System. Bei jedem Aufruf werden Daten zum API-Endpunkt gesendet und anschließend strukturierte Daten empfangen.
Welches Format zum Senden und Empfangen der Daten mit der SLAPI verwendet wird, hängt von der jeweils ausgewählten Implementierung der API ab. Von der SLAPI werden derzeit SOAP, XML-RPC und REST für die Datenübertragung verwendet.  

Weitere Informationen zur SoftLayer-API und zu den APIs für den IBM Bluemix-Service für die Lastausgleichsfunktion finden Sie in den folgenden Ressourcen im SoftLayer Development Network: 
* [Übersicht über SoftLayer-API ![Symbol für externen Link](../../icons/launch-glyph.svg "Symbol für externen Link")](https://sldn.softlayer.com/article/softlayer-api-overview){: new_window} 
* [Einführung in die SoftLayer-API ![Symbol für externen Link](../../icons/launch-glyph.svg "Symbol für externen Link")](http://sldn.softlayer.com/article/getting-started){: new_window}
* [SoftLayer_Product_Package-API ![Symbol für externen Link](../../icons/launch-glyph.svg "Symbol für externen Link")](http://sldn.softlayer.com/reference/services/SoftLayer_Product_Package){: new_window}
* [SoftLayer_Network_LBaaS_LoadBalancer-API ![Symbol für externen Link](../../icons/launch-glyph.svg "Symbol für externen Link")](http://sldn.softlayer.com/reference/services/SoftLayer_Network_LBaaS_LoadBalancer){: new_window}
* [SoftLayer_Network_LBaaS_Listener-API ![Symbol für externen Link](../../icons/launch-glyph.svg "Symbol für externen Link")](http://sldn.softlayer.com/reference/services/SoftLayer_Network_LBaaS_Listener){: new_window}
* [SoftLayer_Network_LBaaS_Member-API ![Symbol für externen Link](../../icons/launch-glyph.svg "Symbol für externen Link")](http://sldn.softlayer.com/reference/services/SoftLayer_Network_LBaaS_Member){: new_window}
* [SoftLayer_Network_LBaaS_HealthMonitor-API ![Symbol für externen Link](../../icons/launch-glyph.svg "Symbol für externen Link")](http://sldn.softlayer.com/reference/services/SoftLayer_Network_LBaaS_HealthMonitor){: new_window}

In den folgenden Beispielen wird Python mit einem Zeep-SOAP-Client verwendet. 

## Beispiel für das Erstellen einer Lastausgleichsfunktion
### Produktpaket-ID und Artikelpreis abrufen
```
from zeep import Client, xsd
import sys

# Benutzername und API-Schlüssel für SLAPI-Aufruf
username = '<Your username>'
apiKey = '<Your apiKey>'

# WSDL für SoftLayer_Product_Package-API
wsdl = 'https://api.softlayer.com/soap/v3/SoftLayer_Product_Package?wsdl'
client = Client(wsdl)

# XSD für Authentifizierung
xsdUserAuth = xsd.Element(
    '{http://api.softlayer.com/soap/v3/}authenticate',
    xsd.ComplexType([
        xsd.Element('{http://api.softlayer.com/soap/v3/}username', xsd.String()),
        xsd.Element('{http://api.softlayer.com/soap/v3/}apiKey', xsd.String())
    ])
)

# XSD für Objektmaske
xsdObjectMask = xsd.Element(
    '{http://api.service.softlayer.com/soap/v3/}SoftLayer_ObjectMask',
    xsd.ComplexType([
        xsd.Element('{http://api.service.softlayer.com/soap/v3/}mask', xsd.String())
    ])
)

# XSD-Wertobjekte erstellen
userAuthValue = xsdUserAuth(username=username, apiKey=apiKey)

lbaasPackage = None
result = client.service.getAllObjects(_soapheaders=[userAuthValue])
productPackages = result['body']['getAllObjectsReturn']
for package in productPackages:
    if package.name == 'Load Balancer As A Service (LBaaS)':
        lbaasPackage = package
        print 'LBaaS product package id: %s' % lbaasPackage.id
        break

if lbaasPackage is None:
    print 'LBaaS product package cannot be found!'
    sys.exit()

xsdObjectInitPar = xsd.Element(
    '{http://api.service.softlayer.com/soap/v3/}SoftLayer_Product_PackageInitParameters',
    xsd.ComplexType([
        xsd.Element('{http://api.service.softlayer.com/soap/v3/}id', xsd.String())
    ])
)

objectInitParValue = xsdObjectInitPar(id=lbaasPackage.id)
xsdObjectMask = xsd.Element(
    '{http://api.service.softlayer.com/soap/v3/}SoftLayer_ObjectMask',
    xsd.ComplexType([
        xsd.Element('{http://api.service.softlayer.com/soap/v3/}mask', xsd.String())
    ])
)

objectMaskValue = xsdObjectMask(mask='mask[id;item.description]')

result = client.service.getItemPrices(_soapheaders=[userAuthValue,objectInitParValue,objectMaskValue])
itemPrices = result['body']['getItemPricesReturn']
for itemPrice in itemPrices:
    print 'Item Price Id: %s' % itemPrice.id
    print 'Item Description: %s\r\n' % itemPrice.item.description
```
### Bestellung der Lastausgleichsfunktion überprüfen
```
from zeep import Client, xsd
from zeep.exceptions import Fault

# Benutzername und API-Schlüssel für SLAPI-Aufruf
username = '<Your username>'
apiKey = '<Your apiKey>'
# Private Teilnetz-ID
privateSubnetId = '<Your subnet id>'

# Auftragszusatzinformationen
# Von SoftLayer_Product_Package-API abgerufene Paket-ID
# (oben angegebenes Beispiel)
lbaasPackageId = 805
# Von SoftLayer_Product_Package-API abgerufene Artikelpreis-ID
# (oben angegebenes Beispiel)
lbaasItemPrices = [{'id':199445}, {'id':199455}, {'id':199465}, {'id':205837}]
name = 'MyLoadBalancer'
subnets = [{'id': privateSubnetId}]
protocolConfigurations = [{
    'frontendProtocol':'HTTP',
    'frontendPort':80,
    'backendProtocol':'HTTP',
    'backendPort':8080,
    'loadBalancingMethod':'ROUNDROBIN',
    'maxConn':1000
}]

# WSDL für SoftLayer_Product_Package-API
wsdl = 'https://api.softlayer.com/soap/v3/SoftLayer_Product_Order?wsdl'
client = Client(wsdl)
orderDataType = client.get_type(
    'ns0:SoftLayer_Container_Product_Order_Network_LoadBalancer_AsAService'
)

# XSD für Authentifizierung
xsdUserAuth = xsd.Element(
    '{http://api.softlayer.com/soap/v3/}authenticate',
    xsd.ComplexType([
        xsd.Element('{http://api.softlayer.com/soap/v3/}username', xsd.String()),
        xsd.Element('{http://api.softlayer.com/soap/v3/}apiKey', xsd.String())
    ])  
)

# XSD-Wertobjekte erstellen
userAuthValue = xsdUserAuth(username=username, apiKey=apiKey)
orderDataValue = orderDataType(
    name=name, packageId=lbaasPackageId, prices=lbaasItemPrices,
    subnets=subnets, protocolConfigurations=protocolConfigurations
)

# SLAPI-Aufruf an SoftLayer_Product_Order::verifyOrder-API ausführen
try:
    result = client.service.verifyOrder(
        _soapheaders=[userAuthValue],
        orderData=orderDataValue
    )

    print 'The order is valid!'

except Fault as exp:
    print 'The order is INVALID!\r\n>>> %s' % exp
```
### Bestellung für Lastausgleichsfunktion aufgeben
```
from zeep import Client, xsd
from zeep.exceptions import Fault

# Benutzername und API-Schlüssel für SLAPI-Aufruf
username = '<Your username>'
apiKey = '<Your apikey>'
# Private Teilnetz-ID
privateSubnetId = '<Your subnet id>'

# Auftragszusatzinformationen
# Von SoftLayer_Product_Package-API abgerufene Paket-ID
# (oben angegebenes Beispiel)
lbaasPackageId = 805
# Von SoftLayer_Product_Package-API abgerufene Artikelpreis-ID
# (oben angegebenes Beispiel)
lbaasItemPrices = [{'id':199445}, {'id':199455}, {'id':199465}, {'id':205837}]
name = 'MyLoadBalancer'
subnets = [{'id': privateSubnetId}]
protocolConfigurations = [{
    'frontendProtocol':'HTTP',
    'frontendPort':80,
    'backendProtocol':'HTTP',
    'backendPort':8080,
    'loadBalancingMethod':'ROUNDROBIN',
    'maxConn':1000
}]

# WSDL für SoftLayer_Product_Package-API
wsdl = 'https://api.softlayer.com/soap/v3/SoftLayer_Product_Order?wsdl'
client = Client(wsdl)
orderDataType = client.get_type(
    'ns0:SoftLayer_Container_Product_Order_Network_LoadBalancer_AsAService'
)

# XSD für Authentifizierung
xsdUserAuth = xsd.Element(
    '{http://api.softlayer.com/soap/v3/}authenticate',
    xsd.ComplexType([
        xsd.Element('{http://api.softlayer.com/soap/v3/}username', xsd.String()),
        xsd.Element('{http://api.softlayer.com/soap/v3/}apiKey', xsd.String())
    ])  
)

# XSD-Wertobjekte erstellen
userAuthValue = xsdUserAuth(username=username, apiKey=apiKey)
orderDataValue = orderDataType(
    name=name, packageId=lbaasPackageId, prices=lbaasItemPrices,
    subnets=subnets, protocolConfigurations=protocolConfigurations
)

# SLAPI-Aufruf für SoftLayer_Product_Order::placeOrder-API ausführen
try:
    result = client.service.placeOrder(
        _soapheaders=[userAuthValue],
        orderData=orderDataValue,
        saveAsQuote=False
    )

    print 'Order has been accepted.'

except Fault as exp:
    print 'Place order failed:\r\n>>> %s' % exp
```

## Beispiel für das Abrufen von Lastausgleichsfunktionen
### Alle Lastausgleichsfunktionen auflisten
```
from zeep import Client

# Benutzername und API-Schlüssel für SLAPI-Aufruf
username = '<Your username>'
apiKey = '<Your apiKey>'

# WSDL für SoftLayer_Network_LBaaS_LoadBalancer-API
wsdl = 'https://api.softlayer.com/soap/v3.1/SoftLayer_Network_LBaaS_LoadBalancer?wsdl'
client = Client(wsdl)

# Authentifizierung für SOAP-Header vorbereiten
userauth = {'authenticate': {'username': username, 'apiKey': apiKey}}

# Alle Objekte der Lastausgleichsfunktion abrufen
result = client.service.getAllObjects(_soapheaders=userauth)
loadbalancers = result['body']['getAllObjectsReturn']
for loadbalancer in loadbalancers:
    print 'UUID: %s' % loadbalancer.uuid
    print 'Name: %s' % loadbalancer.name
    print 'Address: %s' % loadbalancer.address
    print 'OperatingStatus: %s' % loadbalancer.operatingStatus
    print 'ProvisioningStatus: %s\r\n' % loadbalancer.provisioningStatus
```
### Details einer bestimmten Lastausgleichsfunktion abrufen
```
from zeep import Client, xsd

# Benutzername und API-Schlüssel für SLAPI-Aufruf
username = '<Your username>'
apiKey = '<Your apiKey>'
# UUID der Lastausgleichsfunktion
uuid = '<Your load balancer uuid>'

# WSDL für SoftLayer_Network_LBaaS_LoadBalancer-API
wsdl = 'https://api.softlayer.com/soap/v3/SoftLayer_Network_LBaaS_LoadBalancer?wsdl'
client = Client(wsdl)

# XSD für Authentifizierung
xsdUserAuth = xsd.Element(
    '{http://api.softlayer.com/soap/v3/}authenticate',
    xsd.ComplexType([
        xsd.Element('{http://api.softlayer.com/soap/v3/}username', xsd.String()),
        xsd.Element('{http://api.softlayer.com/soap/v3/}apiKey', xsd.String())
    ])  
)

# XSD für Objektmaske
xsdObjectMask = xsd.Element(
    '{http://api.service.softlayer.com/soap/v3/}SoftLayer_ObjectMask',
    xsd.ComplexType([
        xsd.Element('{http://api.service.softlayer.com/soap/v3/}mask', xsd.String())
    ])  
)

# XSD-Wertobjekte erstellen
userAuthValue = xsdUserAuth(username=username, apiKey=apiKey)
objectMaskValue = xsdObjectMask(mask='mask[listeners, healthMonitors]')

# Bestimmtes Objekt einer Lastausgleichsfunktion abrufen (mit Objektmaske zum Abrufen von "Listenern")
loadbalancer = client.service.getLoadBalancer(_soapheaders=[userAuthValue,objectMaskValue], uuid=uuid)
print 'Name: %s' % loadbalancer.name
print 'Address: %s' % loadbalancer.address
print 'OperatingStatus: %s' % loadbalancer.operatingStatus
print 'ProvisioningStatus: %s' % loadbalancer.provisioningStatus
print 'Listeners: %s' % loadbalancer.listeners
print 'HealthMonitors: %s\r\n' % loadbalancer.healthMonitors
```

## Beispiel für das Aktualisieren einer Lastausgleichsfunktion
### Mitglied hinzufügen
```
from zeep import Client, xsd

# Benutzername und API-Schlüssel für SLAPI-Aufruf
username = '<Your username>'
apiKey = '<Your apikey>'
# UUID der zu aktualisierenden Lastausgleichsfunktion
uuid = '<Your load balancer uuid>'
# Hinzuzufügende Back-End-Server
serverInstances = [
    {
        'privateIpAddress': '10.121.220.141', #update with the correct IP
        'weight': 80 #weight is only applicable to Weight Round Robin listeners
    },
    {
        'privateIpAddress': '10.121.220.142'  #update with the correct IP
        # use default weight
    }
]

# WSDL für SoftLayer_Network_LBaaS_Member-API
wsdl = 'https://api.softlayer.com/soap/v3/SoftLayer_Network_LBaaS_Member?wsdl'
client = Client(wsdl)

# XSD für Authentifizierung
xsdUserAuth = xsd.Element(
    '{http://api.softlayer.com/soap/v3/}authenticate',
    xsd.ComplexType([
        xsd.Element('{http://api.softlayer.com/soap/v3/}username', xsd.String()),
        xsd.Element('{http://api.softlayer.com/soap/v3/}apiKey', xsd.String())
    ])  
)

# XSD für Objektmaske
xsdObjectMask = xsd.Element(
    '{http://api.service.softlayer.com/soap/v3/}SoftLayer_ObjectMask',
    xsd.ComplexType([
        xsd.Element('{http://api.service.softlayer.com/soap/v3/}mask', xsd.String())
    ])  
)

# XSD-Wertobjekte erstellen
userAuthValue = xsdUserAuth(username=username, apiKey=apiKey)
objectMaskValue = xsdObjectMask(mask='mask[members]')

# SLAPI-Aufruf für SoftLayer_Network_LBaaS_Member::addLoadBalancerMember-API ausführen
result = client.service.addLoadBalancerMembers(
    _soapheaders=[userAuthValue, objectMaskValue],
    loadBalancerUuid=uuid, serverInstances=serverInstances
)
print result
```
### Protokoll hinzufügen
```
from zeep import Client, xsd

# Benutzername und API-Schlüssel für SLAPI-Aufruf
username = '<Your username>'
apiKey = '<Your apiKey>'
# UUID der Lastausgleichsfunktion
uuid = '<Your load balancer UUID>'
# Neues Protokoll, das hinzugefügt werden soll
protocolConfigurations = [
    {   
        'frontendProtocol': 'TCP',
        'frontendPort': 90,
        'backendProtocol': 'TCP',
        'backendPort': 9090,
        'loadBalancingMethod': 'WEIGHTED_RR',
        'maxConn': 2000
    }   
]

# WSDL für SoftLayer_Network_LBaaS_Listener-API
wsdl = 'https://api.softlayer.com/soap/v3/SoftLayer_Network_LBaaS_Listener?wsdl'
client = Client(wsdl)

# XSD für Authentifizierung
xsdUserAuth = xsd.Element(
    '{http://api.softlayer.com/soap/v3/}authenticate',
    xsd.ComplexType([
        xsd.Element('{http://api.softlayer.com/soap/v3/}username', xsd.String()),
        xsd.Element('{http://api.softlayer.com/soap/v3/}apiKey', xsd.String())
    ])  
)

# XSD für Objektmaske
xsdObjectMask = xsd.Element(
    '{http://api.service.softlayer.com/soap/v3/}SoftLayer_ObjectMask',
    xsd.ComplexType([
        xsd.Element('{http://api.service.softlayer.com/soap/v3/}mask', xsd.String())
    ])  
)

# XSD-Wertobjekte erstellen
userAuthValue = xsdUserAuth(username=username, apiKey=apiKey)
objectMaskValue = xsdObjectMask(mask='mask[listeners]')

# SLAPI-Aufruf für SoftLayer_Network_LBaaS_Listener::updateLoadBalancerProtocols-API ausführen
result = client.service.updateLoadBalancerProtocols(
    _soapheaders=[userAuthValue, objectMaskValue],
    loadBalancerUuid=uuid, protocolConfigurations=protocolConfigurations
)
listeners = result['listeners']
print listeners
```

## Beispiel für das Abbrechen einer Lastausgleichsfunktion
### Lastausgleichsfunktion abbrechen
```
from zeep import Client, xsd
from zeep.exceptions import Fault

# Benutzername und API-Schlüssel für SLAPI-Aufruf
username = '<Your username>'
apiKey = '<Your apiKey>'
# UUID der Lastausgleichsfunktion, die abgebrochen werden soll
uuid = '<Your load balancer uuid>'

# WSDL für SoftLayer_Network_LBaaS_LoadBalancer-API
wsdl = 'https://api.softlayer.com/soap/v3/SoftLayer_Network_LBaaS_LoadBalancer?wsdl'
client = Client(wsdl)

# XSD für Authentifizierung
xsdUserAuth = xsd.Element(
    '{http://api.softlayer.com/soap/v3/}authenticate',
    xsd.ComplexType([
        xsd.Element('{http://api.softlayer.com/soap/v3/}username', xsd.String()),
        xsd.Element('{http://api.softlayer.com/soap/v3/}apiKey', xsd.String())
    ])  
)

# XSD-Wertobjekte erstellen
userAuthValue = xsdUserAuth(username=username, apiKey=apiKey)

# SLAPI-Aufruf für SoftLayer_Network_LBaaS_LoadBalancer::cancelLoadBalancer-API ausführen
try:
    result = client.service.cancelLoadBalancer(
        _soapheaders=[userAuthValue],
        uuid=uuid
    )

    if True:
        print 'The cancellation request is accepted.'

except Fault as exp:
    print 'Failed to cancel load balancer:\r\n>>> %s' % exp
```
