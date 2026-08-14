# Granskningsunderlag – Installationssamordning och bemanning

Datum: 2026-08-14 (uppdaterad efter extern granskning)
Projekt: EcoDataCenter Borlänge
Arbetsfil: Installationssamordning_Datacenter.html

## Syfte

Detta är en fristående lokal HTML-app för installationssamordning. Den innehåller:

- samordningspunkter;
- grupper och gruppanteckningar;
- möten och beslut;
- kontakter;
- daglig bemanning;
- veckorapport;
- CSV-export;
- säkerhetskopia och återställning.

Appen använder ingen server. Uppgifter sparas lokalt i webbläsarens localStorage.

## Historik

### Omgång 1 – rapporterat fel i Bemanning

I fliken Bemanning fyllde användaren i formuläret och tryckte på **Spara bemanning**.
Formuläret stängdes, men raden verkade försvinna.

Åtgärdat:

1. Två JavaScript-funktioner hette `saveStaffing`. Lagringsfunktionen heter nu `persistStaffing`.
2. Bemanningslistan visade endast dagens datum. Vyn har nu en datumväljare och byter
   automatiskt till postens datum efter sparning.
3. Bemanningsrader är unika per datum och avdelning.

Dessa tre rättningar är verifierade och fungerar.

### Omgång 2 – extern granskning

Granskningen kördes både som kodläsning och som automatiserade tester i Chromium
(sv-SE, Europe/Stockholm, öppnad via `file://`). Följande fel hittades och rättades.

#### Allvarliga fel

1. **Dataförlust vid byte av avdelning.**
   Om en befintlig rad redigerades och avdelningen byttes till en avdelning som redan
   fanns samma datum skrevs den andra radens siffror över, samtidigt som ursprungsraden
   låg kvar. Två rader med samma innehåll uppstod och riktiga siffror försvann.
   Sammanslagningen tar nu bort ursprungsraden och användaren får bekräfta först.

2. **Id-kollisioner.**
   `newId()` byggde på `Date.now() + slumptal 0–999`. Poster som skapades i samma
   millisekund konkurrerade om 1 000 värden. Vid återställning av en säkerhetskopia med
   50 poster uppstod minst en dubblett i 74 procent av fallen, vid 100 poster i samtliga.
   Dubbletter gör att redigering och radering träffar fel rad.
   Id genereras nu från en räknare som alltid är unik, och all inläsning passerar
   `ensureUniqueIds` som delar ut nya id till dubbletter.

3. **Formuläret kunde låsa sig permanent.**
   När en anpassad validering utlöstes låg felmeddelandet kvar på fältet. Webbläsaren
   stoppar då nästa submit innan sparfunktionen hinner köra och rensa meddelandet, så
   formuläret gick inte att skicka igen. Meddelandena rensas nu så snart användaren
   ändrar något i formuläret.

#### Övriga fel

4. **Datumvalet fastnade mellan dagar.** Valt datum sparades utan att veta vilken dag det
   sattes, så gårdagens datum låg kvar nästa morgon. Datumvalet gäller nu bara innevarande
   dag, och en tydlig notis med **Gå till idag** visas när ett annat datum är valt.

5. **Mobilvyn.** Knappen **+ Uppdatera avdelning** låg utanför skärmen vid 390 px.
   **Säkerhetskopia**, **Återställ**, backupstatus, **Skriv ut / PDF** och CSV-exporten
   var helt dolda på mobil. Knappraderna radbryts nu, och säkerhetskopiering har fått en
   egen panel på Översikt som syns i alla skärmstorlekar.

6. **Veckorapporten saknade bemanning.** Rapporten innehåller nu avsnitt 7 med dagens
   bemanning per avdelning och totalsumma. Hjälpfunktionen `staffingRowsForToday` var
   definierad men användes inte alls; den används nu av rapporten.

7. **Ofullständig kontroll av siffror.** Kombinationer som 8 planerade, 6 tillgängliga och
   5 frånvarande accepterades utan invändning. Eftersom ersättare kan kallas in blockeras
   detta inte, men raden flaggas nu med en kontrollnotis. Nummerfälten har fått ett tak
   på 999.

8. **Ingen fokusfälla i dialogerna.** Tabb-tangenten tog sig ut ur öppna formulär och
   vidare till knapparna bakom. Fokus stannar nu i dialogen.

9. **Föräldralösa kopplingar.** Anteckningar som pekade på en raderad punkt behöll en
   ogiltig referens. Kopplingen nollställs nu och användaren informeras innan raderingen.

10. **CSS-specificitetsfel.** `.stat span` slog ut `.stat-icon`, så ikonbokstaven i
    nyckeltalen renderades grå i övre vänstra hörnet i stället för centrerad.

11. **Död kod.** `examplePoints()` på 89 rader anropades aldrig och är borttagen.

12. **Krasch-skydd.** Ett ogiltigt datum i lagringen kunde få `Intl.DateTimeFormat` att
    kasta fel och stoppa hela renderingen. Datum valideras nu innan de används.

13. **GDPR.** Varningen om att inte skriva orsak till frånvaro visas nu direkt i
    formuläret, inte bara under listan, och nämner att noteringen följer med i
    CSV-export och säkerhetskopia.

## Kontroller som har genomförts

- JavaScript-syntax kontrollerad med `node --check`.
- Alla statiska `byId`-referenser matchar befintliga HTML-element.
- Alla inline-händelser pekar på definierade funktioner.
- Inga dubbla funktionsnamn finns.
- Svensk synlig text kontrollerad för felaktig teckenkodning.
- 36 automatiserade tester i Chromium, samtliga godkända, utan JavaScript-fel:
  id-generering, sammanslagning av rader, validering, datumhantering, veckorapport,
  fokusfälla, radering med kopplingar, återställning med dubblett-id, mobillayout vid
  390 px samt de fem ursprungliga manuella testfallen.

## Rekommenderade manuella testfall

### Test 1 – normal sparning

1. Ladda om HTML-filen.
2. Öppna **Bemanning**.
3. Välj **+ Uppdatera avdelning**.
4. Ange dagens datum, El, planerat 8, tillgängliga 6 och frånvarande 2.
5. Spara.

Förväntat: formuläret stängs, en bekräftelse visas, datumväljaren visar samma datum,
El-raden syns och Saknas mot plan visar 2.

### Test 2 – uppdatering

1. Redigera El-raden.
2. Ändra tillgängliga från 6 till 7.
3. Spara.

Förväntat: samma rad uppdateras, ingen dubblett skapas, Saknas mot plan visar 1.

### Test 3 – annat datum

1. Skapa en Ventilation-rad för morgondagen.
2. Spara.

Förväntat: vyn byter till morgondagens datum, Ventilation-raden visas direkt, och en
notis visar att ett annat datum än idag är valt. Dagens rader visas igen via **Gå till idag**.

### Test 4 – omladdning

1. Spara en rad.
2. Ladda om sidan med Ctrl+R.

Förväntat: raden finns kvar och vyn står på dagens datum.

### Test 5 – säkerhetskopia

1. Skapa en säkerhetskopia från panelen **Säkerhetskopia** på Översikt.
2. Kontrollera att JSON-filen innehåller egenskapen `staffing`.
3. Återställ filen och kontrollera att bemanningen kommer tillbaka.

### Test 6 – sammanslagning av avdelning

1. Skapa El och Brand samma datum med olika siffror.
2. Redigera Brand-raden och byt avdelning till El.
3. Bekräfta sammanslagningen.

Förväntat: en enda El-rad återstår med de nya siffrorna, ingen Brand-rad ligger kvar.

### Test 7 – mobil

1. Öppna filen i telefonen.
2. Kontrollera att **+ Uppdatera avdelning** syns i sin helhet.
3. Kontrollera att **Säkerhetskopia** går att skapa från Översikt.

## Lagringsnycklar

- `dc-samordning-v2` – samordningspunkter
- `dc-kontakter-v1` – kontakter
- `dc-grupplogg-v1` – anteckningar, möten och beslut
- `dc-bemanning-v1` – bemanning
- `dc-bemanning-visat-datum` – valt visningsdatum, gäller endast innevarande dag
- `dc-senaste-sakerhetskopia` – datum för senaste säkerhetskopia

## Viktiga begränsningar

- Uppgifterna är knutna till den lokala webbläsaren och datorn.
- Rensning av webbdata kan ta bort uppgifterna.
- Säkerhetskopia bör därför skapas regelbundet.
- I Chrome delar alla lokala filer samma lagringsutrymme. En annan lokal HTML-app med
  samma nyckelnamn skulle kunna krocka. Firefox och Safari hanterar `file://` mer
  restriktivt, så testa i den webbläsare som faktiskt används innan bredare införande.
- Känsliga uppgifter om sjukdom, diagnos, lön eller frånvaroorsak ska inte skrivas i appen.
- Skriv endast exempelvis frånvarande och antal.
