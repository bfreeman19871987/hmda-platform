# HMDA Platform Public API

This documenatation describes de public HMDA Platform HTTP API

## Institutions

### Search

* `/institutions?domain=<domain>`

   * `GET` - Returns a list of institutions filtered by their email domain. If none are found, an HTTP 404 error code (not found) is returned

   Example response, with HTTP code 200:

   ```json
   {
     "institutions":
     [
        {
          "id": "0",
          "name": "Bank 0",
          "domains": ["test@bank0.com"],
          "externalIds":[
            {
              "value": "1234",
              "name": "occ-charter-id"
            },
            {
              "value": "1234",
              "name": "ncua-charter-id"
            }
          ]
        }
     ]
   }
   ```

### Modified LAR

* `/institutions/<institutionId>/filings/<period>/lar`

   * `GET` - Returns the Modified LAR, in CSV format. The schema for the data is as follows:

```
   id
   respondent_id
   agency_code
   preapprovals
   action_taken_type
   purchaser_type
   rate_spread
   hoepa_status
   lien_status
   loan_type
   property_type
   purpose
   occupancy
   amount
   msa
   state
   county
   tract
   ethnicity
   co_ethnicity
   race1
   race2
   race3
   race4
   race5
   co_race1
   co_race2
   co_race3
   co_race4
   co_race5
   sex
   co_sex
   income
   denial_reason1
   denial_reason2
   denial_reason3
   period
```

For a definition of these fields, please consult the [HMDA Filing Instructions Guide](http://www.consumerfinance.gov/data-research/hmda/static/for-filers/2017/2017-HMDA-FIG.pdf).
Please note that the Modified LAR does not include the fields `Loan Application Number`, `Date Application Received` or `Date of Action` described in HMDA Filing Instructions Guide.

## Check Digit

### Check digit generation

* `/uli/checkDigit`

   * `POST` - Calculates check digit and full ULI from a loan id.

Example payload, in `JSON` format:

```json
{
  "loanId": "10Bx939c5543TqA1144M999143X"
}
```

Example response:

```json
{
    "loanId": "10Cx939c5543TqA1144M999143X",
    "checkDigit": 10,
    "uli": "10Cx939c5543TqA1144M999143X10"
}
```

A file with a list of Loan Ids can also be uploaded to this endpoint for batch check digit generation.

Example file contents:

```
10Cx939c5543TqA1144M999143X
10Bx939c5543TqA1144M999143X
```

Example response in `JSON` format:

```json
{
    "loanIds": [
        {
            "loanId": "10Bx939c5543TqA1144M999143X",
            "checkDigit": 38,
            "uli": "10Bx939c5543TqA1144M999143X38"
        },
        {
            "loanId": "10Cx939c5543TqA1144M999143X",
            "checkDigit": 10,
            "uli": "10Cx939c5543TqA1144M999143X10"
        }
    ]
}
```

* `/uli/checkDigit/csv`

   * `POST` - calculates check digits for loan ids submitted as a file

Example file contents:

```
10Cx939c5543TqA1144M999143X
10Bx939c5543TqA1144M999143X
```

Example response in `CSV` format:

```csv
loanId,checkDigit,uli
10Bx939c5543TqA1144M999143X,38,10Bx939c5543TqA1144M999143X38
10Cx939c5543TqA1144M999143X,10,10Cx939c5543TqA1144M999143X10
```

### ULI Validation

* `/uli/validate`

   * `POST` - Validates a ULI (correct check digit)

Example payload, in `JSON` format:

```json
{
	"uli": "10Bx939c5543TqA1144M999143X38"
}
```

Example response:

```json
{
    "isValid": true
}
```

A file with a list of ULIs can also be uploaded to this endpoint for batch ULI validation.

Example file contents:

```
10Cx939c5543TqA1144M999143X10
10Bx939c5543TqA1144M999143X38
10Bx939c5543TqA1144M999133X38
```

Example response in `JSON` format:

```json
{
    "ulis": [
        {
            "uli": "10Cx939c5543TqA1144M999143X10",
            "isValid": true
        },
        {
            "uli": "10Bx939c5543TqA1144M999143X38",
            "isValid": true
        },
        {
            "uli": "10Bx939c5543TqA1144M999133X38",
            "isValid": false
        }
    ]
}
```

* `/uli/validate/csv`

   * `POST` - Batch validation of ULIs

Example file contents:

```
10Cx939c5543TqA1144M999143X10
10Bx939c5543TqA1144M999143X38
10Bx939c5543TqA1144M999133X38
```

Example response in `CSV` format:

```csv
uli,isValid
10Cx939c5543TqA1144M999143X10,true
10Bx939c5543TqA1144M999143X38,true
10Bx939c5543TqA1144M999133X38,false
```

### Rate Spread Calculator

* `rateSpread`

    * `POST` - Calculate Rate Spread


Example payload, in `JSON` format:

```json
{
  "actionTakenType": 1,
  "amortizationType": 30,
  "rateType": "FixedRate",
  "apr": 6.0,
  "lockinDate": "2017-11-20",
  "reverseMortgage": 2
}
```

`RateType` can take the following values: `FixedRate` and `VariableRate`

Example Response, in `JSON` format:

```json
{
  "rateSpread": "2.01"
}
```

The response is either a number representing the Rate Spread or "NA"

* `rateSpread/csv`

    * `POST` - Batch Rate Spread calculator

Example file contents:

```
1,30,FixedRate,6.0,2017-11-20,2
1,30,VariableRate,6.0,2017-11-20,2
```

The contents of this file include the `Action Taken Type` (values 1,2,8), `Amortization Term` (1 - 50 years), `Rate Type`, `APR`, `Lockin Date` and `Reverse Mortgage` (values 1 or 2)

Example response in `CSV` format:

```csv
action_taken_type,amortization_type,rate_type,apr,lockin_date,reverse_mortgage,rate_spread
1,30,FixedRate,6.0,2017-11-20,2,2.01
1,30,VariableRate,6.0,2017-11-20,2,2.15
```

## LAR Parsing and Valiation

### Parsing
`/lar/parse`

`POST` - Returns a JSON representation of a LAR, or a list of errors if the LAR fails to parse

Example body:
```json
2|0|1|10164                    |20170224|1|1|3|1|21|3|1|20170326|45460|18|153|0501.00|2|2|5| | | | |5| | | | |1|2|31|0| | | |NA   |2|1
```

Example reponse
```
{
    "respondentId": "0",
    "applicant": {
        "coSex": 2,
        "coRace5": "",
        "coEthnicity": 2,
        "race2": "",
        "coRace2": "",
        "coRace1": 5,
        "race4": "",
        "race3": "",
        "race1": 5,
        "sex": 1,
        "coRace3": "",
        "income": "31",
        "coRace4": "",
        "ethnicity": 2,
        "race5": ""
    },
    "hoepaStatus": 2,
    "agencyCode": 1,
    "actionTakenType": 1,
    "denial": {
        "reason1": "",
        "reason2": "",
        "reason3": ""
    },
    "rateSpread": "NA",
    "loan": {
        "applicationDate": "20170224",
        "propertyType": 1,
        "amount": 21,
        "purpose": 3,
        "id": "10164",
        "occupancy": 1,
        "loanType": 1
    },
    "id": 2,
    "actionTakenDate": 20170326,
    "geography": {
        "msa": "45460",
        "state": "18",
        "county": "153",
        "tract": "0501.00"
    },
    "lienStatus": 1,
    "preapprovals": 3,
    "purchaserType": 0
}
```

Example error response
```json
{
    "lineNumber": 0,
    "errorMessages": [
        "An incorrect number of data fields were reported: 38 data fields were found, when 39 data fields were expected."
    ]
}
```

### Validation
`/lar/validate`

`POST` - Returns a list of syntactical, validity and/or quality errors.  This endpoint omits certain edits that are not relevant to a single LAR.  Edits that are omitted: macro edits, TS-only edits (e.g. Q130), and the following: Q022, S025, S270.

| Query parameter | Description |
| --------------- | ----------- |
| check | String. Valid entries are: "syntactical", "validity", "quality".  If left blank or any other text is entered, will default to all checks. |

Example body:
```json
{
    "respondentId": "0",
    "applicant": {
        "coSex": 2,
        "coRace5": "",
        "coEthnicity": 2,
        "race2": "",
        "coRace2": "",
        "coRace1": 5,
        "race4": "",
        "race3": "",
        "race1": 5,
        "sex": 1,
        "coRace3": "",
        "income": "31",
        "coRace4": "",
        "ethnicity": 2,
        "race5": ""
    },
    "hoepaStatus": 2,
    "agencyCode": 1,
    "actionTakenType": 1,
    "denial": {
        "reason1": "",
        "reason2": "",
        "reason3": ""
    },
    "rateSpread": "NA",
    "loan": {
        "applicationDate": "20170224",
        "propertyType": 1,
        "amount": 21,
        "purpose": 3,
        "id": "10164",
        "occupancy": 1,
        "loanType": 1
    },
    "id": 2,
    "actionTakenDate": 20170326,
    "geography": {
        "msa": "45460",
        "state": "18",
        "county": "153",
        "tract": "0501.00"
    },
    "lienStatus": 1,
    "preapprovals": 3,
    "purchaserType": 0
}
```

Example response:
```json
{
    "syntactical": {
        "errors": []
    },
    "validity": {
        "errors": []
    },
    "quality": {
        "errors": []
    }
}
```

### Parse and Validate

`/lar/parseAndValidate`

`POST` - Returns a list of syntactical, validity and/or quality errors. This endpoint omits certain edits that are not relevant to a single LAR.  Edits that are omitted: macro edits, TS-only edits (e.g. Q130), and the following: Q022, S025, S270.

| Query parameter | Description |
| --------------- | ----------- |
| check | String. Valid entries are: "syntactical", "validity", "quality".  If left blank or any other text is entered, will default to all checks. |

Example body:
```json
2|0|1|10164                    |20170224|1|1|3|1|21|3|1|20170326|45460|18|153|0501.00|2|2|5| | | | |5| | | | |1|2|31|0| | | |NA   |2|1
```

Example response:
```json
{
    "syntactical": {
        "errors": []
    },
    "validity": {
        "errors": []
    },
    "quality": {
        "errors": []
    }
}
```