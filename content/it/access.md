---
title: "Connettersi alle risorse RCF"
author: "RCF"
subtitle:    "Linux, MacOS, Windows"
description: "Accetto alle risorse computazionali RCF tramite Linux, MacOS e Windows attraverso ssh e server grafico."
date:        2020-11-15
image:       ""
tags:        []
categories:  []
showtoc: true
---

Le informazioni successive descrivono come gli utenti possono accedere alle risorse RCF. Tutti gli utenti delle risorse RCF sono responsabili della conoscenza e del rispetto della Politica dell'utente RCF. 

# Credenziali Account
Per utilizzare le risorse fornite RCF è necessario ottenere un account utente RCF. Se non si dispone già di un account RCF, consultare la pagina Guida introduttiva per ulteriori informazioni su come ottenere un account RCF.

L’account RCF utilizza il tuo nome.cognome o numero_di_matricola per il nome utente e la password usata abitualmente per i servizi informatici dell’Università degli Studi di Napoli “Parthenope”.

Importante: RCF non memorizza la password e non siamo in grado di reimpostare la password. Se hai bisogno di assistenza per la password, consulta la pagina web di Ateneo.

# Connettersi tramite ssh
Secure Shell (SSH) è un protocollo che fornisce un accesso sicuro dalla riga di comando a risorse remote come purpleJeans. Utilizzando SSH, è possibile accedere in remoto al proprio account purpleJeans e interagire con il cluster di calcolo ad alte prestazioni purpleJeans.

## Accesso tramite Macintosh e Linux
La maggior parte dei Sistemi Operativi simili a Unix (Mac OS X, Linux, ecc.) forniscono un'utilità ssh per impostazione predefinita a cui è possibile accedere digitando il comando ssh in una finestra del terminale.

Per accedere a purpleJeans da un computer Linux o Mac, aprire un terminale e dalla riga di comando immettere:

```sh
$ ssh <nome.cognome>@purplejeans.uniparthenope.it
```

Oppure

```sh
$ ssh <matricola>@purplejeans.uniparthenope.it
```

Per abilitare l'inoltro X11 quando ci si collega a purpleJeans con ssh, è necessario includere il flag -Y.

Importante: sui Mac deve essere installato il software XQuarz.

# Connettersi usando Windows ed Ssh
Gli utenti di Windows che eseguono la versione di aprile 2018 di Windows 10 avranno abilitato ssh da Powershell per impostazione predefinita. Tutti gli altri utenti Windows dovranno prima scaricare un client ssh per interagire con la riga di comando remota di Unix. Consigliamo MobaXterm, client, sebbene siano disponibili altre opzioni. Una volta installato il client MobaXterm sul tuo computer locale, apri il client MobaXterm (link: https://mobaxterm.mobatek.net) e fai clic sull'icona Sessions nell'angolo in alto a sinistra del client. Quindi eseguire i seguenti passaggi numerati, illustrati nella figura seguente, per stabilire una connessione a purpleJeans:

Fare clic sulla scheda SSH per espandere le opzioni di accesso SSH.
Nell'input del campo Host remoto: purpleJeans.uniparthenope.it (per connettersi al cluster purpleJeans)
Seleziona il pulsante Specifica nome utente e inserisci il tuo nome utente uniparthenope.
Procedere con il login facendo clic sul pulsante OK.

# Connettersi tramite XFast
È in fase di acquisizione il software per l’accesso ubiquitario tramite client dedicato o browser web). 
