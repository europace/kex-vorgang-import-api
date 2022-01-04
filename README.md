# KEX-Vorgang-Import-API

> ⚠️ You'll find German domain-specific terms in the documentation, for translations and further explanations please refer to our [glossary](https://docs.api.europace.de/common/glossary/)

This API enables the user to create new Vorgänge in **Kredit**Smart.

The corresponding JSON-schema, which is useful for code-generation, can be found [here](https://github.com/europace/kex-vorgang-api-schema).

> ⚠️ This API is continuously developed. Therefore we expect
> all users to align with the "[Tolerant Reader Pattern](https://martinfowler.com/bliki/TolerantReader.html)", which requires clients to be
> tolerant towards compatible API changes when reading and processing the data. This means:
>
> 1. unknown properties must not result in errors
>
> 2. Strings with a restricted set of values (Enums) must support new unknown values
>
> 3. sensible usage of HTTP status codes, even if they are not explicitly documented
>

<!-- https://opensource.zalando.com/restful-api-guidelines/#108 -->

## Creation of a new Vorgang

New Vorgänge can be created via **HTTP POST**.

The URL for the creation of Echtgeschäftsvorgängen is:

    https://kex-vorgang-import.ratenkredit.api.europace.de/vorgang?environment=PRODUCTION

The URL for the creation of Testvorgängen is:

    https://kex-vorgang-import.ratenkredit.api.europace.de/vorgang

The data is sent in the request body as JSON.

A successful call results in a response with the HTTP Statuscode **201 CREATED**.

## Authentication

This API is secured by the OAuth client credentials flow using the [Authorization-API](https://docs.api.europace.de/privatkredit/authentifizierung/).
To use this API your OAuth2-Client needs the following Scopes:

| Scope                          | Label in Partnermanagement             | Description                               |
|--------------------------------|----------------------------------------|-------------------------------------------|
| privatkredit:vorgang:schreiben | KreditSmart-Vorgänge anlegen/verändern | Scope for creating and updating a Vorgang |

## Content-Type

This API accepts data with the Content-Type "**application/json**".

Therefore the Content-Type header must be set appropriately in the request. In addition the encoding needs to be specified if it is not UTF-8.

| Request Header Name | Header Value           |
|---------------------|------------------------|
| Content-Type        | application/json       |

## Example

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

## Error Codes

If a request cannot be proccessed successfully the API is returning an error code which the client can react to.

Attention: In case of an error no data is imported into **Kredit**Smart.

| Error code | Message              | Description                                        |
|------------|----------------------|----------------------------------------------------|
| 401        | Unauthorized         | Authentication failed                              |
| 403        | Forbidden            | the client does not have the necessary permissions |
| 422        | Unprocessable Entity | A valid Kundenbetreuer-Partner-ID is missing       |

Further error codes and their meaning can be found on Wikipedia: [HTTP-Statuscode](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)

## Request Format

The data needs to be sent as JSON inside the request body.

For better readability, the overall format is broken down into *types* that are defined separately but should be used at the corresponding positions. The attributes within a block can be specified in any order.

Currently we have only one mandatory field in our dataset (see [Vorgang](#vorgang)).

In general, all data is imported as a new Vorgang in **Kredit**Smart, except:

* values, which are not processable due to a wrong format (e.g. "1" instead of "true", "01.01.2016" instead of "2016-01-01")
* values, which are unneccessary or inconsistent due to the data constellation (e.g. a value inside the 1. Antragsteller for `gemeinsamerHaushalt`)

At different positions inside the data you need to specify a country or a citizenship. These values are using the format as described in [ISO-3166/ALPHA-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements)

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

The property *kundenbetreuer.partnerId* is mandatory.

### Partner

    {
        "partnerId": String
    }

The Europace 2 Partner-ID has 5-characters and has the format ABC12.

### Benachrichtigung

    {
       "aktiv": true | false
    }

If the Benachrichtigung is set to `aktiv`, the Kundenbetreuer receives a confirmation email.

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

The value of `gemeinsamerHaushalt` is relevant for the second Antragsteller only. The value of `voranschrift` is only relevant if `wohnhaftSeit` is more recent than 3 years ago.

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

The `beschaeftigungsart` determines which data is used. For example the `beschaeftigungsart=ARBEITER` means that all data of field `arbeiter` is used, for `beschaeftigungsart=BEAMTER` the data of field `beamter` is used. All other fields will be ignored.
If there is no value for `beschaeftigungsart` or the corresponding field to a `beschaeftigungsart` is empty, all data is ignored.

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

This Type uses the format [ISO-3166/ALPHA-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements)

In addition there is the value "SONSTIGE" ("other")

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

`Fahrzeugkauf` is just processed if the `Finanzierungszweck` is set to "FAHRZEUGKAUF".

## Response Format

The data is sent as JSON in the response body.

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

In `messages` you can find hints and/or adaptions we made, for example about values we could not process.

The property `antragsteller2` is only returned if the Vorgang has 2 Antragsteller.

## Terms of use
The APIs are made available under the following [Terms of Use](https://docs.api.europace.de/terms/).
