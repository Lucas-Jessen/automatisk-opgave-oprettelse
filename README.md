# Automatisk opgave oprettelse i Propbinder – Forenede Service

Én statisk HTML-side (`index.html` + `logo.png`) uden backend. Den masseopretter ens opgaver i Propbinder, hver med en tidsregistrering. INNA-opgaver får i stedet en fakturalinje.

## Brug

1. Log ind med din Propbinder-konto. Bruger du Microsoft-login, kan du i stedet bruge bogmærket under "Logger du ind i Propbinder med Microsoft?".
2. Udfyld Titel, Type, Timetype, Tid pr. opgave og Antal opgaver. Beskrivelse kan udelades.
   - **Fordel opgaverne på Cibus-ejendommene** er slået til fra start. Opgaverne fordeles på skift over alle Cibus-ejendomme (ejendomme i Propbinder, hvor kunden hedder "Cibus …"), i rækkefølge efter ejendomsnummer. Siden husker i browseren, hvilken ejendom der fik den seneste opgave, og næste kørsel fortsætter derfra. Rundgangen er pr. browser, ikke fælles for alle brugere.
   - **Opret på INNA-ejendomme med tid og antal pr. ejendom** er slået fra fra start. Slås den til, åbner en liste over INNA-ejendommene, hvor man angiver tid (timer, f.eks. 0,5 eller 2) og antal ud for hver ejendom. Titel og beskrivelse bruges på alle opgaverne. Imens er Cibus-fordelingen sat på pause, og Timetype er skjult.
     INNA-opgaver **faktureres** i stedet for at blive tidsregistreret. Hver opgave får materialet 1011-103 "Ejendomsinspektør alm. timetakst" med tiden som antal (vises under Fakturering). Derudover slås **Fakturerbar** til, og standard-fakturakategorien sættes. Det er det samme, som Produkter → Tilføj materiale, Fakturerbar og Gem gør i Propbinder. Der laves ingen tidsregistrering og intet overlap-tjek. Test kun på ejendom 00000, ellers kommer testopgaven til fakturering. INNA-ejendommene er en fast liste af ejendomsnumre (`INNA_NUMBERS` i `index.html`), fordi de ikke har en fælles kunde i Propbinder. Listen skal rettes der, når porteføljen ændrer sig. Numre, der ikke findes i Propbinder, vises under boksen (pr. 18.11.2026: 37612).
   - Slås både Cibus og INNA fra, kan man angive ét ejendomsnummer (f.eks. 40104) eller lade feltet stå tomt. Valget huskes i browseren.
3. Tryk **Opret opgaver** og bekræft.

Opgaverne lægges i forlængelse af hinanden, og den sidste slutter som standard nu. Hver opgave (undtagen INNA) får en tidsregistrering på det angivne antal minutter. Opgaver på en ejendom får ejendommens kunde som juridisk lejer, ligesom når Propbinder gemmer en opgave. Som standard markeres de som udført. Siden advarer, hvis tidsrummet overlapper dine andre tidsregistreringer. Der kan oprettes højst 200 opgaver ad gangen.

## Teknik

Siden kalder Propbinders web-API (`https://propbinder-api.azurewebsites.net/`) direkte fra browseren, med brugerens eget login. API'et tillader kald fra alle domæner (CORS `*`). Partner-API'et (`propbinderpartnerapi…`, apiuser) bruges ikke, fordi det ikke kan oprette opgaver.

| Formål | Kald |
| --- | --- |
| Login | `POST api/Website/Login` `{email, password}` → `{bearer}` |
| Brugerinfo / afdeling | `GET api/WebSite/Company/User/UserInfo`, `GET api/Website/Company/User` |
| Typer / timetyper / ejendomme | `GET api/Activity/ActivityTypes`, `GET api/Website/PriceGroup`, `GET api/PropertyLease/GetPropertyList` |
| Overlap-tjek | `POST api/Activity/CheckConflict?isUpdate=false` |
| Opret opgave | `POST api/Activity` → opgave-id |
| Tidsregistrering | `POST api/Activity/AddActivityTimeSpent` |
| Materialer (INNA) | `GET api/Materials/GetMaterials` |
| Fakturalinje (INNA) | `POST api/ActivityMaterials/AddMaterialsToActivity` `{activityId, number, name, quantity, costPrice, salesPrice}` |
| Fakturerbar (INNA) | `PATCH api/Activity/MarkAsBillable/{id}` + `PATCH api/Activity/UpdateInvoiceCategory` `{activityId, invoiceCategoryId}` |
| Markér udført | `PATCH api/Activity/MarkAsDone/{id}` |

Tider sendes som UTC (ISO). Login-nøglen gemmes kun i `sessionStorage` og forsvinder, når fanen lukkes.

Det er Propbinders interne web-API, ikke et dokumenteret partner-API. Det kan ændre sig uden varsel. Holder siden op med at virke, så opret en opgave i Propbinder med browserens udviklerværktøjer åbne, og sammenlign kaldet med `createOne()` i `index.html`.

Når en opgave slettes i Propbinder, bliver dens tidsregistrering ikke slettet med. Den skal fjernes særskilt.

## Lokal test

```sh
python -m http.server 8767 --bind 127.0.0.1
```

Åbn http://127.0.0.1:8767.
