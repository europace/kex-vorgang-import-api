# KEX-Vorgang-Import-API

Die Schnittstelle ermöglicht das automatisierte Anlegen von Vorgängen in **Kredit**Smart.

Unter https://github.com/europace/kex-vorgang-api-schema liegt das zugehörige JSON-Schema, das zur Codegenerierung genutzt werden kann.

> ⚠️ Diese Schnittstelle wird kontinuierlich weiterentwickelt. Daher erwarten wir
> von allen Nutzern dieser Schnittstelle, dass sie das "[Tolerant Reader Pattern](https://martinfowler.com/bliki/TolerantReader.html)" nutzen, d.h.
> tolerant gegenüber kompatiblen API-Änderungen beim Lesen und Prozessieren der Daten sind:
>
> 1. unbekannte Felder dürfen keine Fehler verursachen
>
> 2. Strings mit eingeschränktem Wertebereich (Enums) müssen mit neuen, unbekannten Werten umgehen können
>
> 3. sinnvoller Umgang mit HTTP-Statuscodes, die nicht explizit dokumentiert sind
>

<!-- https://opensource.zalando.com/restful-api-guidelines/#108 -->

## Anlegen eines neuen Vorgangs

Neue Vorgänge können per **HTTP POST** angelegt werden.

Die URL für das Anlegen von Echtgeschäftsvorgängen ist:

    https://kex-vorgang-import.ratenkredit.api.europace.de/vorgang?environment=PRODUCTION

Die URL für das Anlegen von Testvorgängen ist:

    https://kex-vorgang-import.ratenkredit.api.europace.de/vorgang

Die Daten werden als JSON Dokument im Body des POST Requests übermittelt.

Ein erfolgreicher Aufruf resultiert in einer Response mit dem HTTP Statuscode **201 CREATED**.

## Authentifizierung

Für jeden Request ist eine Authentifizierung erforderlich. Die Authentifizierung erfolgt über den OAuth 2.0 Client-Credentials Flow.

| Request Header Name | Beschreibung           |
|---------------------|------------------------|
| Authorization       | OAuth 2.0 Bearer Token |

Das Bearer Token kann über die [Authorization-API](https://docs.api.europace.de/privatkredit/authentifizierung/) angefordert werden. Dazu wird ein Client benötigt, der vorher von einer berechtigten
Person über das Partnermanagement angelegt wurde. Eine Anleitung dafür befindet sich im [Help Center](https://europace2.zendesk.com/hc/de/articles/360012514780).

Damit der Client für diese API genutzt werden kann, muss im Partnermanagement die Berechtigung **KreditSmart-Vorgänge anlegen/verändern** (Scope `privatkredit:vorgang:schreiben`) aktiviert sein.

Schlägt die Authentifizierung fehl, erhält der Aufrufer eine HTTP Response mit Statuscode **401 UNAUTHORIZED**.

Hat der Client keine Berechtigung die Resource abzurufen, erhält der Aufrufer eine HTTP Response mit Statuscode **403 FORBIDDEN**.

## TraceId zur Nachverfolgbarkeit von Requests

Für jeden Request soll eine eindeutige ID generiert werden, die den Request im EUROPACE 2 System nachverfolgbar macht und so bei etwaigen Problemen oder Fehlern die systemübergreifende Analyse
erleichtert.

Die Übermittlung der X-TraceId erfolgt über einen HTTP Header. Dieser Header ist optional, wenn er nicht gesetzt ist, wird eine ID vom System generiert.

| Request Header Name | Beschreibung                    | Beispiel    |
|---------------------|---------------------------------|-------------|
| X-TraceId           | eindeutige Id für jeden Request | sys12345678 |

## Content-Type

Die Schnittstelle akzeptiert Daten mit Content-Type "**application/json**".

Entsprechend muss im Request der Content-Type Header gesetzt werden. Zusätzlich das Encoding, wenn es nicht UTF-8 ist.

| Request Header Name | Header Value           |
|---------------------|------------------------|
| Content-Type        | application/json       |

## Beispiel

### POST Request

    POST https://kex-vorgang-import.ratenkredit.api.europace.de/vorgang
    Authorization: Bearer xxxxxxx
    Content-Type: application/json;charset=utf-8
    {
        "kundenbetreuer": {
            "partnerId": "TEST"
        },
        "antragsteller1": {
            "personendaten": {
                "vorname": "Max",
                "nachname": "Mustermann"
            }
        }
    }

### POST Response

    201 CREATED
    {
        "vorgangsnummer": "AB1234",
        "messages": [],
        "antragsteller1": {
            "id": "abcd123-xyz"
        }
    }

## Fehlercodes

Wenn der Request nicht erfolgreich verarbeitet werden konnte, liefert die Schnittstelle einen Fehlercode, auf den die aufrufende Anwendung reagieren kann, zurück.

Achtung: Im Fehlerfall wird kein Vorgang in **Kredit**Smart importiert und angelegt.

| Fehlercode | Nachricht            | Erklärung                                                  |
|------------|----------------------|------------------------------------------------------------|
| 401        | Unauthorized         | Authentifizierung ist fehlgeschlagen                       |
| 422        | Unprocessable Entity | Es wurde keine gültige Kundenbetreuer-Partner-ID angegeben |

Weitere Fehlercodes und ihre Bedeutung siehe Wikipedia: [HTTP-Statuscode](https://de.wikipedia.org/wiki/HTTP-Statuscode)

## Request Format

Die Angaben werden als JSON im Body des Requests gesendet.

Für eine bessere Lesbarkeit wird das Gesamtformat in *Typen* aufgebrochen, die an anderer Stelle definiert sind, aber an verwendeter Stelle eingesetzt werden müssen. Die Attribute innerhalb eines
Blocks können in beliebiger Reihenfolge angegeben werden.

Für einen erfolgreichen Request gibt es derzeit nur ein definiertes Pflichtfeld (siehe „Vorgang“).

Alle übermittelten Daten werden in **Kredit**Smart übernommen, mit Ausnahme von:

* Angaben, die aufgrund eines abweichenden Formats nicht verstanden werden (z. B. "1" statt "true", "01.01.2016" statt "2016-01-01"), und
* Angaben, die aufgrund der Datenkonstellationen überflüssig bzw. unstimmig sind (z. B. Angabe beim 1. Antragsteller zu gemeinsamerHaushalt).

An verschiedenen Stellen im Request ist die Angabe eines Landes oder der Staatsangehörigkeit notwendig:
Die Übermittlung erfolgt im Format [ISO-3166/ALPHA-2](https://de.wikipedia.org/wiki/ISO-3166-1-Kodierliste)

## Vorgang

    {
        "kundenbetreuer": Partner,
        "bearbeiter": Partner,
        "tippgeber": Partner,
        "leadquelle": String,
        "benachrichtigung": Benachrichtigung
        "eigeneVorgangsnummer": String,
        "baufiSmartVorgangsnummer": String,
        "antragsteller1": Antragsteller,
        "antragsteller2": Antragsteller,
        "haushalt": Haushalt,
        "finanzbedarf": Finanzbedarf,
        "kommentare": [ String ]
    }

Das Feld *kundenbetreuer.partnerId* ist ein Pflichtfeld.

### Partner

    {
        "partnerId": String
    }

Die Europace 2 PartnerID ist 5-stellig und hat das Format ABC12.

### Benachrichtigung

    {
       "aktiv": true | false
    }

Wenn die Benachrichtigung auf aktiv gesetzt ist, bekommt der Kundenbetreuer eine E-Mail als Bestätigung.

### Antragsteller

    {
        "herkunft": Herkunft,
        "personendaten": Personendaten,
        "wohnsituation": Wohnsituation,
        "beschaeftigung": Beschäftigung
    }

### Herkunft

    {
        "staatsangehoerigkeit": "ALPHA-2 Isocode",
        "steuerId": String,
        "inDeutschlandSeit": "YYYY-MM-DD",
        "aufenthaltstitel": "VISUM" | "AUFENTHALTSERLAUBNIS" | "NIEDERLASSUNGSERLAUBNIS" | "ERLAUBNIS_ZUM_DAUERAUFENTHALT_EU",
        "aufenthaltBefristetBis": "YYYY-MM-DD",
        "arbeitserlaubnisVorhanden": true | false,
        "arbeitserlaubnisBefristetBis": "YYYY-MM-DD"
    }

### Personendaten

    {
        "titel": [ "DOKTOR" | "PROFESSOR" ]
        "anrede": "FRAU" | "HERR",
        "telefonGeschaeftlich": String,
        "geburtsdatum": "YYYY-MM-DD",
        "telefonPrivat": String,
        "geburtsort": String,
        "geburtsland": "ALPHA-2 Isocode"
        "vorname": String,
        "geburtsname": String,
        "nachname": String,
        "familienstand": "LEDIG" | "VERHEIRATET" | "GESCHIEDEN" | "VERWITWET" | "GETRENNT_LEBEND" | "EHEAEHNLICHE_LEBENSGEMEINSCHAFT" | "EINGETRAGENE_LEBENSPARTNERSCHAFT",
        "email": String
    }

### Wohnsituation

    {
        "anschrift": {
            "strasse": String,
            "hausnummer": String,
            "plz": String,
            "ort": String,
            "wohnhaftSeit": "YYYY-MM-DD"
        },
        "gemeinsamerHaushalt": true | false,
        "wohnart": "ZUR_MIETE" | "ZUR_UNTERMIETE" | "IM_EIGENEN_HAUS" | "BEI_DEN_ELTERN",
        "anzahlPersonenImHaushalt": Integer,
        "anzahlPkw": Integer,
        "voranschrift": {
            "strasse": String,
            "hausnummer": String,
            "plz": String,
            "ort": String,
            "wohnhaftSeit": "YYYY-MM-DD"
        }
    }

Die Angabe *gemeinsamerHaushalt* ist nur beim zweiten Antragsteller relevant. Die Angabe *voranschrift* ist nur relevant, sofern die *wohnhaftSeit* bei der Anschrift noch keine 3 Jahre zurück liegt.

### Beschaeftigung

    {
        "beschaeftigungsart": "ANGESTELLTER" | "ARBEITER" | "ARBEITSLOSER" | "BEAMTER" | "FREIBERUFLER" | "HAUSFRAU" | "RENTNER" | "SELBSTSTAENDIGER",
        "arbeiter": Arbeiter,
        "angestellter": Angestellter,                
        "arbeitsloser": Arbeitsloser,
        "beamter": Beamter,
        "selbststaendiger": Selbstständiger,
        "freiberufler": Freiberufler,
        "hausfrau": Hausfrau,
        "rentner": Rentner
    }

Die `Beschaeftigungsart` bestimmt die Beschäftigung und damit das dazu korrespondierende Feld. Beispielsweise wird für die `beschaeftigungsart=ARBEITER`
die Daten unter dem Knoten `arbeiter` genutzt, bei der `beschaeftigungsart=BEAMTER` entsprechend der Knoten `beamter`. Werden darüber hinaus weitere Felder befüllt so werden diese ignoriert.
Ist keine `Beschaeftigungsart` gesetzt oder der zur angegebenen Beschäftigungsart passende Knoten nicht befüllt, werden alle Felder ignoriert.

#### Arbeiter

    {
        "beschaeftigungsverhaeltnis": {
            "berufsbezeichnung": String,
            "nettoeinkommenMonatlich": Decimal,
            "arbeitgeber": Firma,
            "beschaeftigtSeit": "YYYY-MM-DD",
            "befristung": "BEFRISTET" | "UNBEFRISTET",
            "befristetBis": "YYYY-MM-DD",
            "inProbezeit": true | false
        },
        "vorherigesBeschaeftigungsverhaeltnis": {
            "arbeitgeber": Firma,
            "beschaeftigtSeit": "YYYY-MM-DD",
            "beschaeftigtBis": "YYYY-MM-DD"
        }
    }

#### Angestellter

    {
        "beschaeftigungsverhaeltnis": {
            "berufsbezeichnung": String,
            "nettoeinkommenMonatlich": Decimal,
            "arbeitgeber": Firma,
            "beschaeftigtSeit": "YYYY-MM-DD",
            "befristung": "BEFRISTET" | "UNBEFRISTET",
            "befristetBis": "YYYY-MM-DD",
            "inProbezeit": true | false
        },
        "vorherigesBeschaeftigungsverhaeltnis": {
            "arbeitgeber": Firma,
            "beschaeftigtSeit": "YYYY-MM-DD",
            "beschaeftigtBis": "YYYY-MM-DD"
        }
    }

#### Arbeitsloser

    {
        "sonstigesEinkommenMonatlich": Decimal
    }

#### Beamter

    {
        "beschaeftigungsverhaeltnis": {
            "berufsbezeichnung": String,
            "inProbezeit": true | false,
            "nettoeinkommenMonatlich": Decimal,
            "verbeamtetSeit": "YYYY-MM-DD",
            "arbeitgeber": Firma,
            "beschaeftigtSeit": "YYYY-MM-DD"
        },
        "vorherigesBeschaeftigungsverhaeltnis": {
            "arbeitgeber": Firma,
            "beschaeftigtSeit": "YYYY-MM-DD",
            "beschaeftigtBis": "YYYY-MM-DD"
        }
    }

#### Selbstständiger

    {
        "berufsbezeichnung": String,
        "selbststaendigSeit": "YYYY-MM-DD",
        "firma": Firma,
        "nettoeinkommenJaehrlich": Decimal,
        "bruttoEinkommenLaufendesJahr": Decimal,
        "einkommenssteuerLaufendesJahr": Decimal,
        "abschreibungenLaufendesJahr": Decimal,
        "bruttoEinkommenLetztesJahr": Decimal,
        "einkommenssteuerLetztesJahr": Decimal,
        "abschreibungenLetztesJahr": Decimal,
        "einkommenssteuerVor2Jahren": Decimal,
        "bruttoEinkommenVor2Jahren": Decimal,
        "abschreibungenVor2Jahren": Decimal,
        "bruttoEinkommenVor3Jahren": Decimal,
        "einkommenssteuerVor3Jahren": Decimal,
        "abschreibungenVor3Jahren": Decimal
    }

#### Freiberufler

    {
        "berufsbezeichnung": String,
        "selbststaendigSeit": "YYYY-MM-DD",
        "firma": Firma,
        "nettoeinkommenJaehrlich": Decimal,
        "bruttoEinkommenLaufendesJahr": Decimal,
        "einkommenssteuerLaufendesJahr": Decimal,
        "abschreibungenLaufendesJahr": Decimal,
        "bruttoEinkommenLetztesJahr": Decimal,
        "einkommenssteuerLetztesJahr": Decimal,
        "abschreibungenLetztesJahr": Decimal,
        "einkommenssteuerVor2Jahren": Decimal,
        "bruttoEinkommenVor2Jahren": Decimal,
        "abschreibungenVor2Jahren": Decimal,
        "bruttoEinkommenVor3Jahren": Decimal,
        "einkommenssteuerVor3Jahren": Decimal,
        "abschreibungenVor3Jahren": Decimal
    }

#### Hausfrau

    {
        "sonstigesEinkommenMonatlich": Decimal
    }

#### Rentner

    {
        "staatlicheRenteMonatlich": Decimal,
        "rentnerSeit": "YYYY-MM-DD",
        "rentenversicherung": {
            "name": String,
            "anschrift": Anschrift
        }
    }

#### Firma

    {
        "name": String,
        "anschrift": Anschrift,
        "branche": "BAUGEWERBE" | "DIENSTLEISTUNGEN" | "ENERGIE_WASSERVERSORGUNG_BERGBAU" | "ERZIEHUNG_UNTERRICHT" | "GEBIETSKOERPERSCHAFTEN" | "GEMEINNUETZIGE_ORGANISATION" | "GESUNDHEIT_SOZIALWESEN" | "HANDEL" | "HOTEL_GASTRONOMIE" | "INFORMATION_KOMMUNIKATION" | "KREDITINSTITUTE_VERSICHERUNGEN" | "KULTUR_SPORT_UNTERHALTUNG" | "LANDWIRTSCHAFT_FORSTWIRTSCHAFT_FISCHEREI" | "OEFFENTLICHER_DIENST" | "PRIVATE_HAUSHALTE" | "VERARBEITENDES_GEWERBE" | "VERKEHR_LOGISTIK"
    }

### Anschrift

    {
        "strasse": String,
        "hausnummer": String,
        "plz": String,
        "ort": String,
        "land": Country
    }

### Country

Die Übermittlung erfolgt im Format [ISO-3166/ALPHA-2](https://de.wikipedia.org/wiki/ISO-3166-1-Kodierliste)

Zusätzlich gibt es den Wert "SONSTIGE"

    "AD" | "AE" | "AF" | "AG" | "AL" | "AM" | "AO" | "AR" | "AT" | "AU" | "AZ" | "BA" | "BB" | "BD" | "BE" | "BF" | "BG" | "BH" | "BI" | "BJ" | "BN" | "BO" | "BR" | "BS" | "BT" | "BW" | "BY" | "BZ" | "CA" | "CD" | "CF" | "CG" | "CH" | "CI" | "CK" | "CL" | "CM" | "CN" | "CO" | "CR" | "XK" | "CU" | "CV" | "CY" | "CZ" | "DE" | "DJ" | "DK" | "DM" | "DO" | "DZ" | "EC" | "EE" | "EG" | "ER" | "ES" | "ET" | "FI" | "FJ" | "FM" | "FR" | "GA" | "GB" | "GD" | "GE" | "GH" | "GM" | "GN" | "GQ" | "GR" | "GT" | "GW" | "GY" | "HN" | "HR" | "HT" | "HU" | "ID" | "IE" | "IL" | "IN" | "IQ" | "IR" | "IS" | "IT" | "JM" | "JO" | "JP" | "KE" | "KG" | "KH" | "KI" | "KM" | "KN" | "KP" | "KR" | "KW" | "KZ" | "LA" | "LB" | "LC" | "LI" | "LK" | "LR" | "LS" | "LT" | "LU" | "LV" | "LY" | "MA" | "MC" | "MD" | "ME" | "MG" | "MH" | "MK" | "ML" | "MM" | "MN" | "MR" | "MT" | "MU" | "MV" | "MW" | "MX" | "MY" | "MZ" | "NA" | "NE" | "NG" | "NI" | "NL" | "NO" | "NP" | "NR" | "NU" | "NZ" | "OM" | "PA" | "PE" | "PG" | "PH" | "PK" | "PL" | "PS" | "PT" | "PW" | "PY" | "QA" | "RO" | "RS" | "RU" | "RW" | "SA" | "SB" | "SC" | "SD" | "SE" | "SG" | "SI" | "SK" | "SL" | "SM" | "SN" | "SO" | "SR" | "SS" | "ST" | "SV" | "SY" | "SZ" | "TD" | "TG" | "TH" | "TJ" | "TL" | "TM" | "TN" | "TO" | "TR" | "TT" | "TV" | "TZ" | "UA" | "UG" | "US" | "UY" | "UZ" | "VA" | "VC" | "VE" | "VN" | "VU" | "WS" | "YE" | "ZA" | "ZM" | "ZW" | "SONSTIGE"

### Haushalt

    {
        "verbindlichkeiten": {
            "geschaeftskredite": [ Geschäftskredit ],
            "kontokorrentkredite": [ Kontokorrentkredit ],
            "kreditkarten": [ Kreditkarte ],
            "dispositionskredite": [ Dispositionskredit ],
            "ratenkredite": [ Ratenkredit ],
            "leasings": [ Leasing ],
            "sonstigeVerbindlichkeiten": [ Sonstige Verbindlichkeit ]
        },
        "vermoegen": {
            "depotvermoegen": [ Depotvermögen ],
            "sonstigeVermoegenswerte": [ Sonstiger Vermögenswert ],
            "bankUndSparguthaben": [ Bank- und Sparguthaben ],
            "lebensversicherungen": [ Lebensversicherung ],
            "bausparvertraege": [ Bausparvertrag ]
        },
        "ausgaben": {
            "privateKrankenversicherungen": [ Private Krankenversicherung ],
            "unterhaltsverpflichtungen": [ Unterhaltsverpflichtung ],
            "sonstigeAusgaben": [ Sonstige Ausgabe ],
            "mietausgaben": [ Mietausgabe ]
        },
        "einnahmen": {
            "einkuenfteAusNebentaetigkeit": [ Einkunft aus Nebentätigkeit ],
            "ehegattenunterhalt": [ Ehegattenunterhalt ],
            "sonstigeEinnahmen": [ Sonstige Einnahme ],
            "einkuenfteAusKapitalvermoegen": [ Einkunft aus Kapitalvermögen ],
            "unbefristeteZusatzrenten": [ Unbefristete Zusatzrente ]
        },
        "immobilien": [ Immobilie ],
        "kinder": [ kind ],
        "kontoverbindung": {
            "iban": String,
            "bic": String,
            "kreditinstitut": String,
            "gehoertZuAntragsteller": Antragstellerzuordnung
        }
    }

#### Antragstellerzuordnung

    "ANTRAGSTELLER_1" | "ANTRAGSTELLER_2" | "BEIDE"

#### Ratenkredit, Geschäftskredit und Sonstige Verbindlichkeit

    {
        "rateMonatlich": Decimal,
        "schlussrate": Decimal,
        "datumErsteZahlung": "YYYY-MM-DD",
        "datumLetzteRate": "YYYY-MM-DD",
        "restschuld": Decimal,
        "urspruenglicherKreditbetrag": Decimal,
        "glaeubiger": String,
        "gehoertZuAntragsteller": Antragstellerzuordnung,
        "abloesen": true | false,
        "iban": String,
        "bic": String,
        "kreditinstitut": String
    }

#### Kontokorrentkredit

    {
        "beanspruchterBetrag": Decimal,
        "verfuegungsrahmen": Decimal,
        "glaeubiger": String,
        "zinssatz": Decimal,
        "gehoertZuAntragsteller": Antragstellerzuordnung,
    } 

#### Kreditkarte

    {
        "beanspruchterBetrag": Decimal,
        "verfuegungsrahmen": Decimal,
        "rateMonatlich": Decimal,
        "glaeubiger": String,
        "zinssatz": Decimal,
        "gehoertZuAntragsteller": Antragstellerzuordnung,
        "abloesen": true | false,
        "iban": String,
        "bic": String,
        "kreditinstitut": String
    }

#### Dispositionskredit

    {
        "beanspruchterBetrag": Decimal,
        "verfuegungsrahmen": Decimal,
        "glaeubiger": String,
        "zinssatz": Decimal,
        "gehoertZuAntragsteller": Antragstellerzuordnung,
        "abloesen": true | false,
        "bic": String,
        "iban": String,
        "kreditinstitut": String
    }

#### Leasing

    {
        "rateMonatlich": Decimal,
        "schlussrate": Decimal,
        "datumLetzteRate": "YYYY-MM-DD",
        "glaeubiger": String,
        "gehoertZuAntragsteller": Antragstellerzuordnung
    }

#### Depotvermögen

    {
        "betrag": Decimal,
        "gehoertZuAntragsteller": Antragstellerzuordnung
    }

#### Sonstiger Vermögenswert

    {
        "betrag": Decimal,
        "gehoertZuAntragsteller": Antragstellerzuordnung
    }

#### Bank- und Sparguthaben

    {
        "betrag": Decimal,
        "gehoertZuAntragsteller": Antragstellerzuordnung
    }

#### Lebensversicherung

    {
        "rueckkaufswert": Decimal,
        "praemieMonatlich": Decimal,
        "gehoertZuAntragsteller": Antragstellerzuordnung
    }

#### Bausparvertrag

    {
        "sparbeitragMonatlich": Decimal,
        "angesparterBetrag": Decimal,
        "gehoertZuAntragsteller": Antragstellerzuordnung
    }

#### Private Krankenversicherung und Unterhaltsverpflichtung

    {
        "betragMonatlich": Decimal,
        "gehoertZuAntragsteller": "ANTRAGSTELLER_1" | "ANTRAGSTELLER_2"
    }

#### Sonstige Ausgabe und Mietausgabe

    {
        "betragMonatlich": Decimal,
        "gehoertZuAntragsteller": Antragstellerzuordnung
    }

#### Einkunft aus Nebentätigkeit

    {
        "betragMonatlich": Decimal,
        "beginnDerTaetigkeit": "YYYY-MM-DD",
        "gehoertZuAntragsteller": "ANTRAGSTELLER_1" | "ANTRAGSTELLER_2"
    }

#### Ehegattenunterhalt, Sonstige Einnahme, Einkunft aus Kapitalvermögen und Unbefristete Zusatzrente

    {
        "betragMonatlich": Decimal,
        "gehoertZuAntragsteller": "ANTRAGSTELLER_1" | "ANTRAGSTELLER_2"
    }

#### Immobilie

    {
        "mieteinnahmenWarmMonatlich": Decimal,
        "vermieteteWohnflaeche": Integer,
        "gehoertZuAntragsteller": Antragstellerzuordnung,
        "nebenkostenMonatlich": Decimal,
        "wert": Decimal,
        "nutzungsart": "EIGENGENUTZT" | "VERMIETET" | "EIGENGENUTZT_UND_VERMIETET",
        "mieteinnahmenKaltMonatlich": Decimal,
        "immobilienart": "EIGENTUMSWOHNUNG" | "EINFAMILIENHAUS" | "MEHRFAMILIENHAUS" | "BUEROGEBAEUDE",
        "bezeichnung": String,
        "wohnflaeche": Integer,
        "darlehen": [
            {
                "restschuld": Decimal,
                "zinsbindungBis": "YYYY-MM-DD",
                "rateMonatlich": Decimal
            }
        ]
    }

#### Kind

    {
        "name": String,
        "kindergeldFuer": "ERSTES_ODER_ZWEITES_KIND" | "DRITTES_KIND" | "AB_VIERTEM_KIND",
        "unterhaltseinnahmenMonatlich": Decimal,
        "gehoertZuAntragsteller": Antragstellerzuordnung
    }

### Finanzbedarf

    {
        "fahrzeugkauf": Fahrzeugkauf,
        "finanzierungszweck": "UMSCHULDUNG" | "FAHRZEUGKAUF" | "MODERNISIEREN" | "FREIE_VERWENDUNG",
        "finanzierungswunsch": {
            "laufzeitInMonaten": Integer,
            "ratenzahlungstermin": "MONATSENDE" | "MONATSMITTE",
            "provisionswunschInProzent": Decimal,
            "kreditbetrag": Decimal,
            "rateMonatlich": Decimal
        },
        "ratenschutz": {
            "versicherteRisikenAntragsteller2": [ "ARBEITSLOSIGKEIT" | "ARBEITSUNFAEHIGKEIT" | "LEBEN" ],
            "versicherteRisikenAntragsteller1": [ "ARBEITSLOSIGKEIT" | "ARBEITSUNFAEHIGKEIT" | "LEBEN" ]
        }
    }

#### Fahrzeugkauf

    {
        "modell": String,
        "marke": String,
        "kaufpreis": Decimal,
        "erstzulassungsdatum": "YYYY-MM-DD",
        "laufleistung": Integer,
        "kw": Integer,
        "beglicheneKosten": Decimal,
        "ps": Integer,
        "anbieter": "HAENDLER" | "PRIVAT"
    }

Fahrzeugkauf wird nur ausgewertet, wenn als Finanzierungszweck "FAHRZEUGKAUF" gesetzt ist.

## Response Format

Die Angaben werden als JSON im Body der Response gesendet.

    {
        "vorgangsnummer": String,
        "messages": [ String ],
        "antragsteller1": {
            "id": String
        },
        "antragsteller2": {
            "id": String
        }
    }

In *messages* werden nicht übernommene Angaben und andere Hinweise gesendet.

Das Feld `antragsteller2` gibt es nur, wenn ein Vorgang mit zwei Antragstellern angelegt wurde.

## Nutzungsbedingungen

Die APIs werden unter folgenden [Nutzungsbedingungen](https://docs.api.europace.de/nutzungsbedingungen/) zur Verfügung gestellt.
