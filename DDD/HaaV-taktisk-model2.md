# HaaV - Taktisk domænemodel: Salg og ordre

**Gruppe:** ______________  **Par 1 (Kurv):** ____  

---

## Opgave A - Begreberne

**E** = entitet · **V** = værdiobjekt · **X** = hører til i en anden kontekst

| Nr. | Begreb | E/V/X | Begrundelse (kun hvis den ikke er indlysende) | Hvis X: hvilken kontekst, og hvad kender vi den ved? |
|---|---|---|---|---|
| 1 | Kurv | | | |
| 2 | Kurvlinje | | | |
| 3 | Ordre | | | |
| 4 | Ordrelinje | | | |
| 5 | Vare | | | |
| 6 | Varebeskrivelse m. billeder | | | |
| 7 | Medlem | | | |
| 8 | Provisionssats | | | |
| 9 | Kunde | | | |
| 10 | Beløb | | | |
| 11 | Antal | | | |
| 12 | Leveringsadresse | | | |
| 13 | Leveringsmåde | | | |
| 14 | Afhentningskode | | | |
| 15 | Ordrestatus | | | |
| 16 | Sidste salgsdato | | | |

**Vi er uenige om:** _______________________________________

---

## Opgave B - Aggregatet

**Vores aggregat:** ~~☐~~ Kurv (par 1)  ☐ Ordre (par 2)

|                                                 |                                                                                                        |
|-------------------------------------------------|--------------------------------------------------------------------------------------------------------|
| **Aggregate Root**                              | Kurv                                                                                                   |
| **Børn (entiteter der ikke findes uden roden)** | Kurvlinje                                                                                              |
| **Værdiobjekter**                               | Antal<br/>Beløb (pris)                                                                                 |
| **Refereret ved Id (og hvorfor)**               | KundeId og vareId, fordi Kunde og Vare ligger uden for Kurv-aggregatet og derfor kun refereres ved Id. |

### Invarianter

Regler der **altid** skal være sande, når en ændring er færdig. Skriv dem, som en købmand ville sige dem.

| # | Invariant                                   | Involverer rod + barn? | Beskyttes af metoden |
|---|---------------------------------------------|------------------------|----------------------|
| 1 | Antal vare skal være mindst 1               | Ja                     | ÆndrAntal()          |
| 2 | Samme vare må kun have én kurvlinje         | Ja                     | LægVareIKurv()       |
| 3 | Prisen på en kurvlinje må ikke være negativ | Ja                     | LægVareIKurv()       |

### Metoder på roden

| Metode (forretningshandling) | Hvad den gør | Hvilken invariant den håndhæver |
|------------------------------|---|---|
| LægVareIKurv()               |Tilføjer en vare til kurven. Hvis varen allerede findes, øges antallet på den eksisterende kurvlinje |Samme vare må kun have én kurvlinje |
| ÆndrAntal()                  |Ændrer antallet på en eksisterende kurvlinje |Antal på en kurvlinje skal være mindst 1 |

**Hvad går i stykker, hvis to brugere ændrer aggregatet samtidigt?**
Normalt vil to forskellige brugere have hver sin kurv og derfor ændre hvert sit Kurv-aggregate, men hvis to brugere derimod har adgang til den samme kurv (ægtepar fx), kan samtidige ændringer overskrive hinanden og medføre forkert antal eller indhold i kurven.

## Opgave C - Hændelser og kommandoer

| Domænehændelse (datid)          | Intern / Integration | Hvilken kontekst lytter? | Hvad gør den så?                            |
|---------------------------------|----------------------|--------------------------|---------------------------------------------|
| Antal i kurv blev ændret        | Intern               | Salg og ordre            | Antallet af en vare i kurven bliver ændret  |
| Vare blev tilføjet til kurv     | Intern               | Salg og ordre            | Tilføjer varen og antal til kurv            |
| Vare blev fjernet fra kurv      | Intern               | Salg og ordre            | Fjerner varer fra kurv                      |
| Kurv blev konverteret til ordre | Integration          | Levering                 | Forbereder levering ud fra ordreoplysninger |

**Kommandoer (bydeform):**

- Tilføj vare til kurv
- Fjern vare fra kurv

**Forespørgsel:**

- Hent kurv

**Et sted hvor eventuel konsistens gør en forretningsmæssig forskel:**
Når en vare tilføjes eller fjernes for en kurv kan systemet godt blive forsinket og det okay, men ved konvertering til en ordre skal informationen være korrekt

## Opgave D - Kollisionen

**1. Kurv og Ordre — ét aggregat eller to? Vores beslutning og begrundelse:**

**2. Hændelsen der krydser grænsen:**

**3. Hvad betaler Lise for rugbrødet — og hvor i modellen bor svaret?**

**4. Opdelingen i to afregninger: på `Ordre`, eller i en domænetjeneste? Hvorfor?**

**Bonus — Jørns ene lerskål. Hvilket aggregat, og i hvilken kontekst, beskytter den regel?**

**Fælles skitse:** indsæt foto eller diagram her.

---

## Opgave E - Hjemme

### Klassediagram

<details>
<summary>PlantUML-skabelon</summary>

```plantuml
@startuml
skinparam classAttributeIconSize 0
hide empty members

package "Aggregat: Kurv" {
  class Kurv <<Aggregate Root>> {
    - Id : Guid
    - KundeId : Guid
    --
    + TilføjVare(vareId : Guid, antal : Antal, beløb : Beløb)
    + FjernVare(vareId : Guid)
  }

  class Kurvlinje <<Value Object>> {
    - Antal : Antal
    - Beloeb : Beløb
    - VareId : Guid
  }

  Kurv "1" -- "0.." Kurvlinje : indeholder
}

class Antal <<Value Object>> {
  + Værdi : int
}

class Beløb <<Value Object>> {
  + Værdi : decimal
  + Valuta : string
}

Kurvlinje ..> Antal
Kurvlinje ..> Beløb

note right of Kurvlinje
  VareId er en reference
  til et andet aggregat -
  ikke et objekt
end note

note right of Kurv
  KundeId er en reference
  til et andet aggregat -
  ikke et objekt
end note

@enduml
```

</details>

### C#-skelet

Sti til klassebibliotek i repo'et: _______________________________________

### Unit test af en invariant

Testens navn: _______________________________________

**Fejler testen, hvis I fjerner reglen fra modellen?** ☐ Ja ☐ Nej — hvis nej: reglen bor ikke, hvor I tror.

### Repository-interface

```csharp
public interface IOrdreRepository
{
    // Kun roden. Ingen metoder der henter eller gemmer et barn direkte.
}
```

**Hvorfor er `HentAlleOrdrelinjer()` en dårlig idé her?**

### Valgfrit: BPMN

Indsæt diagram. Hvert trin annoteret med kommando og hændelse.
