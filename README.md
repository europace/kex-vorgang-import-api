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

The property *kundenbetreuer.partnerId* is mandatory.\
Concerning *leadquelle*:

If you are using the selfservice via api (RaaS) please provide the correct value for “leadquelle” for your selfservice-use case. That is important in order to generate appropriate and valuable reporting.
Possible values for “leadquelle” regarding your selfservice usecase are:
* For the usecases “Marktplatz in der Bank”, “Marktplatz beim Vertrieb” and “Vermittler:innen Frontend” the format for “leadquelle” is USECASE_PARTNERID_CHANNEL
  * Possible values for USECASE: MARKTPLATZBANK, MARKTPLATZVERTRIEB, VERMITTLERFRONTEND
  * PARTNERID is your Europace partner id
  * For CHANNEL you can define the value on your own suitable for your special project.
  * example: “MARKTPLATZBANK_ABC12_KAMPAGNE”, MARKTPLATZVERTRIEB_ABC12_WEBSITE”, “VERMITTLERFRONTEND_ABC12_CRMSYSTEM”
* For the usecase “LeadTools” the format is `TOOL_PARTNERID`
  * Possible values for TOOL: KREDITLEAD, KREDITTIPP, KREDITLEADSELFSERVICE
  * PARTNERID is your Europace partner id
  * example: KREDITLEAD__ABC12

If you are not sure which values ​​you should use for USECASE,  PARTNERID or CHANNEL please get in touch with your Europace representative.

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
        "arbeitserlaubnisVorhanden": true | false,
        "arbeitserlaubnisBefristetBis": "YYYY-MM-DD",
        "aufenthaltstitel": "VISUM" | "AUFENTHALTSERLAUBNIS" | "NIEDERLASSUNGSERLAUBNIS" | "ERLAUBNIS_ZUM_DAUERAUFENTHALT_EU",
        "aufenthaltBefristetBis": "YYYY-MM-DD",
        "inDeutschlandSeit": "YYYY-MM-DD",
        "staatsangehoerigkeit": "ALPHA-2 Isocode",
        "steuerId": String
    }

### Personendaten

    {
        "anrede": "FRAU" | "HERR",
        "email": String,
        "familienstand": "LEDIG" | "VERHEIRATET" | "GESCHIEDEN" | "VERWITWET" | "GETRENNT_LEBEND" | "EHEAEHNLICHE_LEBENSGEMEINSCHAFT" | "EINGETRAGENE_LEBENSPARTNERSCHAFT",
        "geburtsdatum": "YYYY-MM-DD",
        "geburtsname": String,
        "geburtsort": String,
        "geburtsland": "ALPHA-2 Isocode",
        "nachname": String,
        "telefonGeschaeftlich": String,
        "telefonPrivat": String,
        "titel": [ "DOKTOR" | "PROFESSOR" ],
        "vorname": String,
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
        "anzahlPersonenImHaushalt": Integer,
        "anzahlPkw": Integer,
        "gemeinsamerHaushalt": true | false,
        "wohnart": "ZUR_MIETE" | "ZUR_UNTERMIETE" | "IM_EIGENEN_HAUS" | "BEI_DEN_ELTERN",
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
        "angestellter": Angestellter,
        "arbeiter": Arbeiter,              
        "arbeitsloser": Arbeitsloser,
        "beamter": Beamter,
        "freiberufler": Freiberufler,
        "hausfrau": Hausfrau,
        "rentner": Rentner,
        "selbststaendiger": Selbstständiger
    }

The `beschaeftigungsart` determines which data is used. For example the `beschaeftigungsart=ARBEITER` means that all data of field `arbeiter` is used, for `beschaeftigungsart=BEAMTER` the data of field `beamter` is used. All other fields will be ignored.
If there is no value for `beschaeftigungsart` or the corresponding field to a `beschaeftigungsart` is empty, all data is ignored.

#### Angestellter

    {
        "beschaeftigungsverhaeltnis": {
            "arbeitgeber": Firma,   
            "berufsbezeichnung": String,
            "befristung": "BEFRISTET" | "UNBEFRISTET",
            "befristetBis": "YYYY-MM-DD",
            "beschaeftigtSeit": "YYYY-MM-DD",
            "inProbezeit": true | false,
            "nettoeinkommenMonatlich": BigDecimal
        },
        "vorherigesBeschaeftigungsverhaeltnis": {
            "arbeitgeber": Firma,
            "beschaeftigtSeit": "YYYY-MM-DD",
            "beschaeftigtBis": "YYYY-MM-DD"
        }
    }
    
#### Arbeiter

    {
        "beschaeftigungsverhaeltnis": {
            "arbeitgeber": Firma,   
            "berufsbezeichnung": String,
            "befristung": "BEFRISTET" | "UNBEFRISTET",
            "befristetBis": "YYYY-MM-DD",
            "beschaeftigtSeit": "YYYY-MM-DD",
            "inProbezeit": true | false,
            "nettoeinkommenMonatlich": BigDecimal
        },
        "vorherigesBeschaeftigungsverhaeltnis": {
            "arbeitgeber": Firma,
            "beschaeftigtSeit": "YYYY-MM-DD",
            "beschaeftigtBis": "YYYY-MM-DD"
        }
    }

#### Arbeitsloser

    {
        "sonstigesEinkommenMonatlich": BigDecimal
    }

#### Beamter

    {
        "beschaeftigungsverhaeltnis": {
            "arbeitgeber": Firma,
            "berufsbezeichnung": String,
            "beschaeftigtSeit": "YYYY-MM-DD",
            "inProbezeit": true | false,
            "nettoeinkommenMonatlich": BigDecimal,
            "verbeamtetSeit": "YYYY-MM-DD"
        },
        "vorherigesBeschaeftigungsverhaeltnis": {
            "arbeitgeber": Firma,
            "beschaeftigtSeit": "YYYY-MM-DD",
            "beschaeftigtBis": "YYYY-MM-DD"
        }
    }
    
#### Freiberufler

    {
        "berufsbezeichnung": String,
        "firma": Firma,
        "selbststaendigSeit": "YYYY-MM-DD", 
        "nettoeinkommenJaehrlich": BigDecimal,
        "bruttoEinkommenLaufendesJahr": BigDecimal,
        "einkommenssteuerLaufendesJahr": BigDecimal,
        "abschreibungenLaufendesJahr": BigDecimal,
        "bruttoEinkommenLetztesJahr": BigDecimal,
        "einkommenssteuerLetztesJahr": BigDecimal,
        "abschreibungenLetztesJahr": BigDecimal,
        "einkommenssteuerVor2Jahren": BigDecimal,
        "bruttoEinkommenVor2Jahren": BigDecimal,
        "abschreibungenVor2Jahren": BigDecimal,
        "bruttoEinkommenVor3Jahren": BigDecimal,
        "einkommenssteuerVor3Jahren": BigDecimal,
        "abschreibungenVor3Jahren": BigDecimal
    }
    
#### Hausfrau

    {
        "sonstigesEinkommenMonatlich": BigDecimal
    }

#### Rentner

    {
        "rentenversicherung": {
            "name": String,
            "anschrift": Anschrift
        },
        "rentnerSeit": "YYYY-MM-DD",
        "staatlicheRenteMonatlich": BigDecimal
    }

#### Selbstständiger

    {
        "berufsbezeichnung": String,
        "firma": Firma,
        "selbststaendigSeit": "YYYY-MM-DD",
        "nettoeinkommenJaehrlich": BigDecimal,
        "bruttoEinkommenLaufendesJahr": BigDecimal,
        "einkommenssteuerLaufendesJahr": BigDecimal,
        "abschreibungenLaufendesJahr": BigDecimal,
        "bruttoEinkommenLetztesJahr": BigDecimal,
        "einkommenssteuerLetztesJahr": BigDecimal,
        "abschreibungenLetztesJahr": BigDecimal,
        "einkommenssteuerVor2Jahren": BigDecimal,
        "bruttoEinkommenVor2Jahren": BigDecimal,
        "abschreibungenVor2Jahren": BigDecimal,
        "bruttoEinkommenVor3Jahren": BigDecimal,
        "einkommenssteuerVor3Jahren": BigDecimal,
        "abschreibungenVor3Jahren": BigDecimal
    }

#### Firma

    {
        "anschrift": Anschrift,
        "branche": "BAUGEWERBE" | "DIENSTLEISTUNGEN" | "ENERGIE_WASSERVERSORGUNG_BERGBAU" | "ERZIEHUNG_UNTERRICHT" | "GEBIETSKOERPERSCHAFTEN" | "GEMEINNUETZIGE_ORGANISATION" | "GESUNDHEIT_SOZIALWESEN" | "HANDEL" | "HOTEL_GASTRONOMIE" | "INFORMATION_KOMMUNIKATION" | "KREDITINSTITUTE_VERSICHERUNGEN" | "KULTUR_SPORT_UNTERHALTUNG" | "LANDWIRTSCHAFT_FORSTWIRTSCHAFT_FISCHEREI" | "OEFFENTLICHER_DIENST" | "PRIVATE_HAUSHALTE" | "VERARBEITENDES_GEWERBE" | "VERKEHR_LOGISTIK",
        "name": String
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
        "ausgaben": {
            "mietausgaben": [ Mietausgabe ],
            "privateKrankenversicherungen": [ Private Krankenversicherung ],
            "unterhaltsverpflichtungen": [ Unterhaltsverpflichtung ],
            "sonstigeAusgaben": [ Sonstige Ausgabe ]
        },
         "einnahmen": {
            "ehegattenunterhalt": [ Ehegattenunterhalt ],
            "einkuenfteAusNebentaetigkeit": [ Einkunft aus Nebentätigkeit ],
            "sonstigeEinnahmen": [ Sonstige Einnahme ],
            "unbefristeteZusatzrenten": [ Unbefristete Zusatzrente ]
        },
        "verbindlichkeiten": {
            "dispositionskredite": [ Dispositionskredit ],
            "kreditkarten": [ Kreditkarte ],
            "leasings": [ Leasing ],
            "ratenkredite": [ Ratenkredit ],
            "sonstigeVerbindlichkeiten": [ Sonstige Verbindlichkeit ]
        },
        "vermoegen": {
            "bausparvertraege": [ Bausparvertrag ],
            "lebensversicherungen": [ Lebensversicherung ]
        },
        "immobilien": [ Immobilie ],
        "kinder": [ kind ],
        "kontoverbindung": {
            "bic": String,
            "gehoertZuAntragsteller": Antragstellerzuordnung,
            "iban": String,
            "kreditinstitut": String
        }
    }

#### Antragstellerzuordnung

    "ANTRAGSTELLER_1" | "ANTRAGSTELLER_2" | "BEIDE"
    
#### Private Krankenversicherung und Unterhaltsverpflichtung

    {
        "betragMonatlich": BigDecimal,
        "gehoertZuAntragsteller": "ANTRAGSTELLER_1" | "ANTRAGSTELLER_2"
    }

#### Sonstige Ausgabe und Mietausgabe

    {
        "betragMonatlich": BigDecimal,
        "gehoertZuAntragsteller": Antragstellerzuordnung
    }
    
#### Ehegattenunterhalt, Sonstige Einnahme und Unbefristete Zusatzrente

    {
        "betragMonatlich": BigDecimal,
        "gehoertZuAntragsteller": "ANTRAGSTELLER_1" | "ANTRAGSTELLER_2"
    }
    
#### Einkunft aus Nebentätigkeit

    {
        "betragMonatlich": BigDecimal,
        "beginnDerTaetigkeit": "YYYY-MM-DD",
        "gehoertZuAntragsteller": "ANTRAGSTELLER_1" | "ANTRAGSTELLER_2"
    }
    
#### Dispositionskredit

    {
        "abloesen": true | false,
        "beanspruchterBetrag": BigDecimal,
        "bic": String,
        "gehoertZuAntragsteller": Antragstellerzuordnung,
        "glaeubiger": String,
        "iban": String,
        "kreditinstitut": String,
        "verfuegungsrahmen": BigDecimal,
        "zinssatz": BigDecimal
    }

#### Kreditkarte

    {
        "abloesen": true | false,
        "beanspruchterBetrag": BigDecimal,
        "bic": String,
        "gehoertZuAntragsteller": Antragstellerzuordnung,
        "glaeubiger": String,
        "iban": String,
        "kreditinstitut": String,
        "rateMonatlich": BigDecimal,
        "verfuegungsrahmen": BigDecimal,
        "zinssatz": BigDecimal  
    }

#### Leasing

    {
        "datumLetzteRate": "YYYY-MM-DD",
        "gehoertZuAntragsteller": Antragstellerzuordnung,
        "glaeubiger": String,
        "rateMonatlich": BigDecimal,
        "schlussrate": BigDecimal
    }

#### Ratenkredit und Sonstige Verbindlichkeit

    {
        "abloesen": true | false,
        "bic": String,
        "datumErsteZahlung": "YYYY-MM-DD",
        "datumLetzteRate": "YYYY-MM-DD",
        "glaeubiger": String,
        "gehoertZuAntragsteller": Antragstellerzuordnung,
        "iban": String,
        "kreditinstitut": String,
        "rateMonatlich": BigDecimal,
        "restschuld": BigDecimal,
        "schlussrate": BigDecimal,
        "urspruenglicherKreditbetrag": BigDecimal
    }

#### Bausparvertrag

    {
        "gehoertZuAntragsteller": Antragstellerzuordnung,
        "sparbeitragMonatlich": BigDecimal
    }
    
#### Lebensversicherung

    {
        "gehoertZuAntragsteller": Antragstellerzuordnung,
        "praemieMonatlich": BigDecimal
    }

#### Immobilie

    {
        "bezeichnung": String,
        "darlehen": [
            {
                "restschuld": BigDecimal,
                "zinsbindungBis": "YYYY-MM-DD",
                "rateMonatlich": BigDecimal
            }
        ],
        "gehoertZuAntragsteller": Antragstellerzuordnung,
        "immobilienart": "EIGENTUMSWOHNUNG" | "EINFAMILIENHAUS" | "MEHRFAMILIENHAUS" | "BUEROGEBAEUDE",
        "mieteinnahmenKaltMonatlich": BigDecimal,
        "mieteinnahmenWarmMonatlich": BigDecimal,
        "nebenkostenMonatlich": BigDecimal,
        "nutzungsart": "EIGENGENUTZT" | "VERMIETET" | "EIGENGENUTZT_UND_VERMIETET",
        "vermieteteWohnflaeche": Integer,
        "wert": BigDecimal,
        "wohnflaeche": Integer
    }

#### Kind

    {
        "gehoertZuAntragsteller": Antragstellerzuordnung,
        "kindergeldFuer": "ERSTES_ODER_ZWEITES_KIND" | "DRITTES_KIND" | "AB_VIERTEM_KIND",
        "name": String,
        "unterhaltseinnahmenMonatlich": BigDecimal
    }

### Finanzbedarf

    {
        "fahrzeugkauf": Fahrzeugkauf,
        "finanzierungszweck": "UMSCHULDUNG" | "FAHRZEUGKAUF" | "MODERNISIEREN" | "FREIE_VERWENDUNG",
        "finanzierungswunsch": {
            "kreditbetrag": BigDecimal,
            "laufzeitInMonaten": Integer,
            "provisionswunschInProzent": BigDecimal,
            "rateMonatlich": BigDecimal,
            "ratenzahlungstermin": "MONATSENDE" | "MONATSMITTE"
        },
        "ratenschutzAntragsteller1": FinanzbedarfRatenschutz,
        "ratenschutzAntragsteller2": FinanzbedarfRatenschutz
    }
 
`fahrzeugkauf` is only processed if the `finanzierungszweck` is set to `FAHRZEUGKAUF`.

`ratenschutzAntragsteller` is only processed if no `versicherteRisikenAntragsteller` is set.

#### Fahrzeugkauf

    {
        "anbieter": "HAENDLER" | "PRIVAT",
        "beglicheneKosten": BigDecimal,
        "erstzulassungsdatum": "YYYY-MM-DD",
        "kaufpreis": BigDecimal,
        "kw": Integer,
        "laufleistung": Integer,
        "marke": String,
        "modell": String,
        "ps": Integer
    }

#### FinanzbedarfRatenschutz
    
    {
      arbeitslosigkeitAbsicherung: RatenschutzAbsicherung,
      arbeitsunfaehigkeitAbsicherung: RatenschutzAbsicherung,
      todesfallAbsicherung: RatenschutzAbsicherung
    }
    
##### RatenschutzAbsicherung

    {
      gewuenscht: Boolean,
      kommentar: String,
      vorhanden: Boolean,
      wichtig: Boolean
    }

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
