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


# Introduzione

Per iniziare ad utilizzare il servizio IBM Bluemix Load Balancer, seleziona **Rete > Programmi di bilanciamento del carico > IBM Bluemix Load Balancer** dal catalogo Bluemix ed esegui la seguente procedura:

1. Seleziona il tuo data center e controlla il piano di servizio. Fai clic su **Avanti**.
2. Selezionare la sottorete a cui vuoi distribuire il tuo programma di bilanciamento del carico. Il tuo programma di bilanciamento del carico avrà una delle proprie interfacce di rete in questa sottorete. Assicurati i tuoi server dell'applicazione siano su questa sottorete o raggiungibili da essa. Se necessario, abilita lo spanning delle VLAN. Fai clic su **Avanti**.
3. Definisci i tuoi parametri del servizio di base, come il nome, la descrizione, le porte e i protocolli dell'applicazione di backend e frontend e il metodo di bilanciamento del carico. Puoi definire un massimo di due protocolli durante la creazione del servizio iniziale. Puoi definire fino a dieci protocolli dopo aver creato il servizio. Devi inoltre utilizzare una porta di frontend univoca. Fai clic su **Avanti**.
4. Modifica i tuoi parametri del controllo di integrità se lo desideri, altrimenti utilizza le impostazioni predefinite. Fai clic su **Avanti**.
5. Associa una o più istanze del server dietro il tuo programma di bilanciamento del carico. Devi visualizzare soltanto le istanze del server locali nel tuo data center. Fai clic su **Avanti**.
6. Controlla la pagina di riepilogo e fai quindi clic su **Crea**. 


La pagina di riepilogo visualizza l'istanza del servizio del programma di bilanciamento del carico che hai appena creato. Prendi nota del campo **Stato**. Uno stato di `Offline` implica che il programma di bilanciamento del carico non è nel servizio. Non può essere effettuata alcuna nuova configurazione e non può essere eseguito il provisioning di alcun alcun servizio di bilanciamento del carico finché lo stato non viene modificato in `Online`. Potresti dover aggiornare la tua schermata per visualizzare lo stato corrente.
 
Facendo clic sul nome del servizio su questa pagina ti sposti alla pagina della panoramica del servizio. Puoi passare alle schede **Protocolli**, **Controlli integrità** e **Istanze del server** per modificare la tua configurazione esistente.
