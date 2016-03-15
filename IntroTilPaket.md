---
layout: post
title: "Håndtering av avhengigheter med Paket"
author: anm
tags: [CSharp, NuGet, Avhengigheter, Devops, FSharp, Paket]
img: https://cdn.rawgit.com/fsprojects/Paket/master/docs/files/img/logo.png 
---


I DIPS benytter vi, som alle andre .NET sjapper, NuGet som pakkebehandler og integrasjonspunkt. Vi bruker NuGet til alt fra tredjepartsavhengigheter til interne biblioteker, felles kontrakter og utillitykode. Vi bruker til og med NuGet til å pakke brukerdokumentasjonen vår (skrevet i [AsciiDoc](http://www.methods.co.nz/asciidoc/)). 

NuGet fungerer utmerket til det aller meste, men har også sine klare svakheter. Blant annet kan det fort bli vanskelig å holde oversikt over hvilke pakker og versjoner som gjelder på kryss av bolker med titalls solutions, hver med sine hundretalls prosjekter. 

Sommeren 2015 kom vi over [Paket](https://fsprojects.github.io/Paket/). Paket er en kryssplattform  avhengighetshåndterer som fungerer som et overbygg over NuGet. Prosjektet er [open source](https://github.com/fsprojects/Paket), skrevet i F# av [Steffen Forkmann](https://github.com/forki) som blant annet står bak det utmerkete byggesystemet [FAKE](https://github.com/fsharp/FAKE). Så hva kan Paket tilby oss som ikke NuGet allerede gjør? La oss se på ting NuGet ikke nødvendigvis er så flinke på. 

### Hva gode gamle NuGet ikke er så god på og hvordan Paket kan hjelpe

#### Transitive avhengigheter 

NuGet håndterer organisering av transitive avhengigheter heller dårlig. Hentes det ned en pakke som er avhengig av en annen pakke, som kanskje igjen er avhengig av en tredje pakke, havner alle pakkene på likt nivå listet ned i den lokale `packages.config` filen. Det er vanskelig å vite hvilke pakker som er avhengige av hverandre, som igjen kan skape trøbbel med å finne ut hva som drar inn hva. Enda verre blir det om to pakker har avhengighet til samme pakke. NuGet vil da velge ut siste versjon av denne pakken uten at du har kontroll og kan gjøre noe med det. Paket håndterer dette ved å liste opp alle avhengigheter, komplett med transitive avhengigheter og tillat versjonsspenn.

#### Versjonsnummre i filnavn og prosjektfiler

Som standard inneholder en nedlastet og utpakket NuGet-pakke versjonsnummeret sitt som en del av mappenavnet. Dette vil i praksis si at ``hitpath``-elementent som havner i ``csproj`` filen inneholder et hardt versjonsnummer. Dette kan fort skape trøbbel ved oppgradering av pakken. Paket på sin side gir oss kun pakkenavn i stien og legger det samme til i prosjektfilen. Dette gjør oppgradering uproblematisk.

#### Forskjellige versjoner av pakker på kryss av prosjekter

Hvert prosjekt som konsumerer NuGet-pakker får som standard sin egen ``packages.config`` fil av NuGet, komplett med pakkeoversikt og versjoner. Dette gjør det mer komplisert å håndtere versjoner på kryss av prosjekter: Oppgraderer vi Newtonsoft.Json fra versjon 7 til 8 må vi huske å gjøre dette *alle* steder vi bruker pakken (som fort er veldig mange plasser i dette tilfellet). Paket er bygget opp rundt konseptet om sentral håndtering av pakker og versjoner. Hvert prosjekt oppgir hvilke pakker den trenger - og kun det. Ingen versjoner spesifiseres her, det gjøres sentralt. 

#### Avhengighetshåndtering basert på Semver 

Har en pakke transitive avhengigheter vil NuGet ta en defansiv tilnærming når den plukker ut versjonen den skal bruke. Den vil gjerne velge laveste versjon av avhengigheten sin for å "være på den sikre siden". Paket følger [SemVer](http://semver.org/) når den regner seg fram til hvilke versjoner den skal ta (men støtter også NuGet's defansive modell). 

#### "Enkel" pakking av prosjekter som ``nupkg`` filer

Bygger man prosjekter som skal distribueres som NuGet-pakker er det ingen automatisk måte å gjøre dette på: Man må selv opprette ``nuspec``-filer og sørge for å holde disse oppdatert. Paket tilbyr en enkel fil som basert på prosjektfilene dine kan lage og publisere pakker for deg. 

### Oppsett av Paket

Å komme i gang med Paket er svært enkelt. På *rota* av repoet ditt, utenfor ``src`` mappen, oppretter vi en mappe som heter ``.Paket/``. Stikk så bort på GitHub og last ned siste versjon av [Paket.bootstrapper.exe](https://github.com/fsprojects/Paket/releases/latest). Ved å kjøre denne fila blir selve `Paket.exe` lastet ned. Her er det viktig å sjekke inn bootstrapperen i versjonskontroll, ikke ``Paket.exe``. Dette for å sikre at vi til en hver tid har siste versjon av Paket tilgjengelig - prosjektet er i høyeste grad levende og det kommer ofte nye versjoner ut.

Når dette er gjort, hopp tilbake til rota av repoet og lag en ``Paket.dependencies`` fil. Dette er "the golden file" som inneholder informasjon om avhengighetene vi har i alle solutions og prosjekter i repoet vårt. Denne kan se slik ut: 

	source https://nuget.org/api/v2
	framework: >= net45
	
	nuget Castle.Windsor-log4net >= 3.2
	nuget newtonsoft.json @~> 8.0.0
	nuget NUnit

Fila beskriver at vi ønsker å bruke [NuGet.org](https://www.nuget.org/) som kilde til pakkene våre.
Neste linje forteller Paket at vi kun er interesserte i versjoner av pakkene for .NET Framework 4.5 og høyere. 
Videre Ønsker vi oss ``Castle.Windsor-log4net`` større eller lik versjon `3.2`, mens newtonsoft.json skal være en versjon mellom `8.0.0` og `8.1.0`. 
``NUnit`` vil vi ha "latest and greatest" av. 

Neste skritt er å navigere inn til *prosjektfilene* våre. på samme nivå som prosjektene lager vi en `Paket.references` fil. Denne fila inneholder de pakkene vi trenger i det respektive prosjektet. For testprosjektet vil vår `Paket.references` fil inneholde: 

	NUnit
	newtonsoft.json

Legg merke til at det ikke er noe versjonsnummer angitt her, det løser Paket sentralt. `Paket.references` sørger for at referanser til de oppgitte pakkene blir skutt inn i prosjektfilene.

Når både `Paket.dependencies` og `Paket.references` er lagret kan vi installere avhengighetene våre. (Her representert med mono, Paket er tross alt kryssplattform): 

	$ mono .Paket/Paket.exe install
	Paket version 2.51.11.0
	Resolving packages for group Main:
	 - newtonsoft.json 8.0.2
	 - Castle.Windsor-log4net 3.3.0
	 - NUnit 3.2.0
	 - Castle.Core-log4net 3.3.3
	 - Castle.LoggingFacility 3.3.0
	 - log4net 1.2.10
	 - Castle.Core 3.3.3
	 - Castle.Windsor 3.3.0
	Locked version resolution written to /Users/andreasmosti/Dev/PaketDemo/Paket.lock
	Downloading Castle.Windsor-log4net 3.3.0
	Downloading Castle.LoggingFacility 3.3.0
	Downloading Castle.Core-log4net 3.3.3
	Downloading Castle.Core 3.3.3
	Downloading Castle.Windsor 3.3.0
	Downloading NUnit 3.2.0
	8 seconds - ready.

Outputen viser oss at vi har fått pakker innenfor de kravene vi har spesifisert i `Paket.dependencies`. 

Når `install` kjøres opprettes en `Paket.lock` fil ved siden av `Paket.dependencies`. Denne fila inneholder som navnet tilsier en *låst* tilstand av de pakkene vi nå fikk lastet ned. I dette tilfellet ser den slik ut: 

	FRAMEWORK: >= NET45
	NUGET
	  remote: https://www.nuget.org/api/v2
	  specs:
	    Castle.Core (3.3.3)
	    Castle.Core-log4net (3.3.3)
	      Castle.Core (>= 3.3.3)
	      log4net (1.2.10)
	    Castle.LoggingFacility (3.3.0)
	      Castle.Core (>= 3.3.0)
	      Castle.Windsor (>= 3.3.0)
	    Castle.Windsor (3.3.0)
	      Castle.Core (>= 3.3.0)
	    Castle.Windsor-log4net (3.3.0)
	      Castle.Core-log4net (>= 3.3.0)
	      Castle.LoggingFacility (>= 3.3.0)
	    log4net (1.2.10)
	    Newtonsoft.Json (8.0.2)
	    NUnit (3.2.0)
		

Denne filen fungerer nå som fasit for avhengighetene våre, både direkte og transitive. En flott måte å holde oversikt over eksakt hvilke versjoner som er i sving i *alle* prosjekter repoet vårt består av. Sjekkes denne filen inn kan kommandoen `paket.restore` brukes for å hente ned avhengighetene våre i nøyaktig samme versjon, som igjen sikrer at andre utviklere så vel som byggeservere bruker pakker. 

### Sluttord

I dette innlegget er helt grunnleggende teori og oppsett beskrevet. Paket kan gjøre så mye mere. Prosjektsiden [https://fsprojects.github.io/Paket/index.html](https://fsprojects.github.io/Paket/index.html) er en kjemperessurs om du ønsker å gå igang med Paket, eller bare er interessert i å lese mere.

Så langt er vi kjempefornøyde med Paket og har fått mye større kontroll over pakkene våre enn hva NuGet alene kan tilby. Å ha tilgang til det transitive treet gjør jobben med å renske ut gamle pakker mye enklere for oss. Paket har også vist seg å være et svært levende prosjekt som tar support på alvor. Vi opprettet en del issues på GitHub av cornercaser som ikke var godt støttet der og da, og hver gang ble problemene løst og ny versjon ute få timer etter innrapportering. Som utvikler må man bare elske slik entusiasme som teamet bak Paket viser!  

Ta kontroll over avhengighetene dine - bruk Paket! 
 
 