# Automatisk opgave oprettelse i Propbinder – Forenede Service

Én statisk HTML-side (`index.html` + `logo.png`) uden backend. Den masseopretter ens opgaver i Propbinder, hver med en tidsregistrering.

## Brug

1. Log ind med din Propbinder-konto. Bruger du Microsoft-login, kan du i stedet bruge bogmærket under "Logger du ind i Propbinder med Microsoft?".
2. Udfyld Titel, Type, Timetype, Tid pr. opgave og Antal opgaver. Beskrivelse og Ejendom kan udelades.
3. Tryk **Opret opgaver** og bekræft.

Opgaverne lægges i forlængelse af hinanden, og den sidste slutter som standard nu. Hver opgave får en tidsregistrering på det angivne antal minutter. Som standard markeres de som udført. Siden advarer, hvis tidsrummet overlapper dine andre tidsregistreringer. Der kan oprettes højst 200 opgaver ad gangen.

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
| Markér udført | `PATCH api/Activity/MarkAsDone/{id}` |

Tider sendes som UTC (ISO). Login-nøglen gemmes kun i `sessionStorage` og forsvinder, når fanen lukkes.

Det er Propbinders interne web-API, ikke et dokumenteret partner-API. Det kan ændre sig uden varsel. Holder siden op med at virke, så opret en opgave i Propbinder med browserens udviklerværktøjer åbne, og sammenlign kaldet med `createOne()` i `index.html`.

Når en opgave slettes i Propbinder, bliver dens tidsregistrering ikke slettet med. Den skal fjernes særskilt.

## Lokal test

```sh
python -m http.server 8765 --bind 127.0.0.1
```

Åbn http://127.0.0.1:8765.
