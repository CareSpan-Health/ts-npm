---
sidebar_position: 1
---

<!-- https://dev.bandwidth.com/apis/voice/#tag/Calls/operation/listCalls -->

# Value Object

repo: `cscore/npm-package/cs-package`

Value Object (VO) is used to work with

* CareSpan Record Data - When pulling data from the database
* CareSpan Save Data - When adding record to database
* FHIR Date - When returning data as a resource

The VO also has business logic. For example, it can check if two users are in the same group.

:::info
When naming the VO, the VO should match the database structure of CareSpan if possible
:::

:::warning
VO should not rely on any models or modules

VO should rely on data that gets passed in. This is how we can keep VOs test-able using unit testing
:::

## Instanciating

### Constructor

When creating new VO, we will using the new command to create an instance

```typescript
const vital : VitalRecord = new VitalRecord(properties, "carespan" | "fhir");
```

This is how you create a new VO instance

### `init` and `normalizeProperties`

The constructor will save the properties

```typescript
    constructor(properties: Record<string, any>, structureType: 'carespan' | 'fhir' = 'carespan') {
        this.init(properties, structureType);
        this.normalizeProperties();
    }
```

You may use override `init` and `normalizeProperties` functions if you need to do something specific

* `init` is for any intialization
* `normalizeProperties` is to adjust any properties after initialization (e.g. adjusting time format and set it in `extension` property)

## VO Class Variables

### `properties`

This is for saving the raw data coming out of the database

### `extensions`

This is for saving the adjusted data (within `normalizeProperties` function). For example, assessment/evaluation (cat `17`) will need extra computation based on the properties to see if the assessment/evaluation has been submitted.

Once we have that figured out, we can save it in `extensions`

### `resourceType`

This is the CareSpan resource type that is unique per `VO` class. This is how we can look at a `JSON` and fit the resource in the right `VO`

## VO Class Functions

All `VOs` have the following

| name             | description                   |
| ---------------- | ----------------------------- |
| getProperty      | Get a value from `property`   |
| setProperty      | Set a value for `property`    |
| appendProperties | Merge an object to `property` |
| getExtension     | Get a value from `extension`  |
| setExtension     | Set a value for `extension`   |

### VO.getProperty

#### VO.getProperty Parmeters

| name         | description                         |
| ------------ | ----------------------------------- |
| key : string | The key for looking up a `property` |

#### VO.getProperty Return

| description                 |
| --------------------------- |
| the value of the `property` |

# HealthRecord Value Object

repo: `cscore/serverless/ehr/carespan-ehr`

HealthRecord `VO` extends from `VO`

This is a `VO` that is used to manage the data stored in `ehr`.`records` table. So it has all the variables and functions of VO and more.

## HealthRecord Class Variables

### `recordId`

This is the record id of a record from the `ehr`.`records` table

### `categoryId`

This is what the category id is for this record.

[https://carespan-health.github.io/dataset/docs/dataset/patient-health-records](https://carespan-health.github.io/dataset/docs/dataset/patient-health-records)

## HealthRecord Class Functions

All `VOs` have the following

| name           | description                                                                                                                                   |
| -------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| getField       | Get a value from `propertie.fields`                                                                                                           |
| setField       | Set a value for `propertie.fields`                                                                                                            |
| toCs           | get the data in a structure can be used to save to the database                                                                               |
| toFhir         | get the data in FHIR resource                                                                                                                 |
| toFhirList     | get a list of FHIR resources<br/>Sometimes, what we store on CareSpan is not a 1 to 1 match with FHIR                                         |
| init (o)       | Need to override init to parse FHIR if FHIR resource is passed in to the constructor                                                          |
| getEventDetail | When working with EHR, we must track all the events (CRUD)<br/>Hence, we wil populate this function to store the information we want to track |

:::note
o for overriding
:::

### HealthRecord.getField

#### HealthRecord.getField Parameters

| name         | description                      |
| ------------ | -------------------------------- |
| key : string | The key for looking up a `field` |

#### HealthRecord.getField Return

| description              |
| ------------------------ |
| the value of the `field` |

:::note
Field is tricky

When returning from the database, the structure looks like this:

```typescript
// this is the interface
interface CsField {
    label?: string;
    ukey?: string | number | undefined;
    units?: any | undefined;
    recordid?: string[] | number[];
    data: any[]
}

// this is what the object looks like
{
    label: "string"
    ...
    data:  [
        <value>
    ]
}
```

The return value will parse through all that and return

```typescript
field?.data?.[0];
```

:::

:::warning
There could be a scenario when you need to grabe `field?.data` as an array

Please write a `getList` function in `HealthRecord` to do this
:::

### HealthRecord.setField

#### HealthRecord.setField Parameters

| name         | description                      |
| ------------ | -------------------------------- |
| key : string | The key for looking up a `field` |
| value : any  | Save thes value in `field` |

:::note
Saving to field is tricky also. This function will wrap the value so it can be stored in the right structure


```typescript

    setField(key: string, value: any) {
        const field: CsField = {
            data: [value],
        }
        // extrat error checking ...
        this.properties.fields[key] = field;
    }
```

See [CsField](#healthrecordgetfield-return)
:::

### HealthRecord.toFhir

#### HealthRecord.toFhir Return

| description    |
| -------------- |
| the VO in FHIR |

As of this writing, we will be using `fhir/r5` from `@types/fhir` package. For example

```typescript
import { Annotation, CarePlan, Goal, Reference } from "fhir/r5";
```

Here is an example of what toFhir would look like:

```typescript
toFhir(version: string = 'r5') {

    const fhir:QuestionnaireResponse = {
        resourceType: 'QuestionnaireResponse',
        status: 'in-progress',
        questionnaire: `Questionnaire/${this.getField('assessmentid')}`,
        subject: {
            reference: `Patient/${this.getProperty('patientGuid')}`,
        },
        authored: this.getReceivedAt(),
    }
    return fhir;
}
```

### HealthRecord.init

The init function is triggered automatically by the constructor to populate `properties` class variable

We want to make sure we can popualte `properties` from both `carespan` and `fhir` objects

Here is an example

```typescript
init(properties: Record<string, any>, structureType: "carespan" | "fhir" = "carespan") {
    if (structureType == "fhir" && (properties as CarePlan)?.resourceType == "CarePlan") {

        const fhir: CarePlan = properties as CarePlan;
        const prop = {'fields': {}};

        const concern = {
            '0': 'patient',
            'concern': fhir?.title
        };

        this.setField('concern', concern)

        // there are more fields that got removed to keep example simple
        // ...

        // populate the `properties` using `prop`
        super.init(prop, 'carespan');
    }

    super.init(properties, structureType);
}

```
