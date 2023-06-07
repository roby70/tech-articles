---
title: Trasformare una FAQ in una chat interattiva tramite Azure Bot (e Teams)
date: 2023-02-16
---

L'utilizzo di FAQ (frequently asked questions) è diffuso in vari ambiti per
fornire un primo supporto agli utenti.  
Con l'utilizzo del [servizio Azure Bot](https://azure.microsoft.com/products/bot-services/) è possibile trasformare l'esperienza utente in modo semplice e rendere queste informazioni facilmente accessibili in modo interattivo.

Il servizio di chat può essere facilmente integrato in un sito web, su Microsoft Teams o in tanti altri canali. Puoi trovare maggiori informazioni sui canali supportati [qui](https://learn.microsoft.com/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0).

Di seguito viene fornita una breve introduzione all'argomento e una guida
per la configurazione del servizio.

## Perchè trasformare la FAQ in un chatbot

Ci sono molti motivi per preferire l'utilizzo di una chat interattiva rispetto ad una semplice FAQ testuale.  

Tuttavia, esistono anche dei motivi per non farlo, come ad esempio il costo (seppur non elevato), ma ritengo che il principale sia legato ai contenuti.  
Se le informazioni che desideriamo fornire sono limitate
e possono essere presentate all'utente tramite una piccola pagina web, allora non è necessario trasformare la stessa in una chat.

Supponiamo invece di considerare una base di conoscenza abbastanza ampia attraverso la chat. Quali sono i motivi potrebbero spingerci a scegliere questo tipo di strumento?

Il motivo principale dovrebbe essere quello di semplificare l'esperienza utente.
In questo senso, la chat mette a disposizione una *ricerca di tipo semantico*.
Utilizzando le funzionalità di analisi del linguaggio naturale, si consente all'utente di trovare le risposte che cerca esprimendo le richieste nel proprio linguaggio.  

Considerando una pagina di FAQ molto lunga: non tutti gli utenti conoscono le funzionalità di ricerca del browser e, anche se le conoscono, la ricerca si limita a trovare corrispondenze esatte nel testo. Pertanto, se non si conoscono i termini utilizzati nel documento, la ricerca diventa difficoltosa.

Come potenziale conseguenza, migliorando la capacità dell'utente di trovare autonomamente le informazioni necessarie, si riducono le richieste all'help desk.

Un altro motivo potrebbe essere la raccolta di dati sui tipi di domande per migliorare il servizio stesso. Con il servizio di chat, le domande possono essere registrate per identificare eventuali domande senza risposta. Ciò consente di migliorare il servizio integrando le informazioni (aggiungendo nuove informazioni o semplicemente inserendo forme alternative alle domande già disponibili).

> -- DA QUI CHIEDERE REVISIONE A CHATGPT --
> -- DA QUI RIVEDERE I CONTENUTI --

## Ma come faccio ad aggiornare il bot? Devo mantenere una FAQ separata?

Con l'utilizzo di Azure Bot e di [Language Studio](https://language.cognitive.azure.com/home) la gestione della base di conoscenza è molto semplice.

Si può decidere di utilizzare una FAQ già esistente (sia esso un documento Word, PDF o una pagina HTML), oppure si può decidere di utilizzare le funzionalità di Language Studio per gestire la redazione delle domande e delle risposte.

### 
Tra le funzionalità di rilievo di Language Studio troviamo la possibilità di 

## Come creare il bot e testarlo

La procedura per creare un bot è descritta dettagliatamente (e a mio avviso in modo efficace) nel tutorial
[Build a bot with the Language Service and Azure Bot Service - MS Learn](https://learn.microsoft.com/en-us/training/modules/build-faq-chatbot-qna-maker-azure-bot-service/), 
ed in particolare nell'esercizio collegato
[Explore question answering](https://microsoftlearning.github.io/AI-900-AIFundamentals/instructions/04d-create-a-bot.html).  
Pertanto in questa sede mi limito a riportare un piccolo riassunto dei passi da seguire e
di alcuni aspetti da tenere in considerazione.

Supponendo di avere già a disposizone una sottoscrizione Azure i passi da seguire sono i seguenti:

1) Creare un nuovo [Language service](https://learn.microsoft.com/en-us/azure/cognitive-services/language-service/overview)
   Questo servizio è la base per la gestione del riconoscimento del testo ed il punto di partenza
   per implementare soluzioni che utilizzano tale tecnologia. Il servizio di per se riunisce diverse funzionalità,
   nel nostro caso siamo interessati a utilizzare quelle collegate alla QnA.
2) Utilizzare [Language Studio](https://language.cognitive.azure.com/home) per creare un *progetto* ed addestrare il sistema
   fornendo l'elenco delle domande e risposte della nostra FAQ
3) Testare il funzionamento del sistema con la funzionalità di Test integrata, eventualmente integrare le domande con
   domande alternative per affinare le capacità di riconoscimento del sistema.
4) Dopo aver validato in Language Studio il funzionamento del sistema, si effettua il Deploy (sul language service collegato), che consiste
   nel creare un modello su Azure basato sulle coppie di domande e risposte fornite.
5) A questo punto si può creare un [Azure Bot](https://learn.microsoft.com/en-us/azure/bot-service/?view=azure-bot-service-4.0).
   Per comodità da Language Studio nella pagina di deploy è già presente disponibile un link
   che utilizza un template per creare il bot e l'infrastruttura relativa.

A questo punto si ha a disposizione un servizio Azure Bot da integrare nei sistemi di interesse (come ad es. Teams).

### Personalizzazione

#### Messaggio di benvenuto

Se avete impostato la lingua in italiano è opportuno fare in modo che anche il messaggio iniziale sia in italiano.  
Di default infatti il bot inizia la conversazione (almeno sui canali web) con la frase "Hello and Welcome!".

La personalizzazione di questa frase non è immediata. Infatti occorre andare a configurare l'*App Service* associato
al bot. Nel caso specifico occorre aggiungere l'impostazione `DefaultWelcomeMessage` nella sezione *Configuration > Application Settings".





### Punti di attenzione

La procedura come si può vedere è tutto sommato semplice, considerando il tipo di servizio che si riesce ad ottenere,
ci sono però alcune cose da tenere in considerazione.

Language Service richiede l'utilizzo di un [Azure Cognitive Search Service](https://learn.microsoft.com/azure/search/search-what-is-azure-search), nella sottoscrizione è possibile attivare un servizio di questo tipo utilizzando un tier gratuito (ed uno solo). Per cui nel caso sia già utilizzato un servizio di questo tipo e si sfrutti il tier gratuito sarà necessario attivare uno degli altri piani a pagamento.

## Come integrare il bot in teams

Una volta creato l'Azure Bot service è possibile andare a integrare lo stesso in teams in modo relativamente semplice.

Questa operazione si fa attraverso la voce di menu Channels andando a selezionare *Microsoft Teams*

## Come integrare il bot in una web application

Sempre attraverso i Channels è possibile configurare la chat di modo che sia inclusa in un sito web.
Per fare questo considerare [Connect a bot to Web Chat](https://learn.microsoft.com/azure/bot-service/bot-service-channel-connect-webchat?view=azure-bot-service-4.0#get-your-bot-secret-key).

## Azure Bot non serve solo a questo ...

Quanto presentato nell'articolo è solo una delle tante possibilità di utilizzo del servizio.

Esistono molte altre possibilità per sfruttare questa tecnologia, e sicuramente una tra le più
interessanti è quella di integrare il bot in una applicazione per far interagire
gli utenti con l'applicazione stessa.

Altra possibilità è quella di creare dei classificatori di testo automatici ecc.

