---
id: mortality
title: "Mortality"
---
## Key Questions
- How do you calculate the mortality rate in a claims dataset?
- Is it possible for a patient to die multiple times in claims data?  How often does this happen?

## How to Calculate
The basis of any mortality measure is having sound data about which patients died, when they died, and what they died from.

There are two places death information may be found in claims data.  Death information is sometimes included in eligibility and enrollment information.  In the Tuva Claims Data Model (CDM), this includes death_date and death_flag fields in the eligibility table.  However, death data from eligibility information is notoriously incomplete and inaccurate.

The second place you can find death information is in the medical claims data (i.e. medical_claim in the CDM).  Every inpatient medical claim is required to have a discharge_disposition_code.  discharge_disposition_code = 20 corresponds to a patient that died.  This information  

Unfortunately these two sources of information often do not agree.

- death_date
- death_flag
- discharge_disposition_code
- discharge_date

If you don’t already have reliable death data in your enrollment and eligibility data, you can calculate death in a straightforward way:

```sql
select distinct
    patient_id
,   discharge_date as death_date
from claims_common.medical_claim
where discharge_disposition_code = '20'
```

## Data Quality Issues
One of the most common and embarrassing data quality problems in healthcare data is that it’s not unusual for a patient to die multiple times.  You can check whether you have any patients who have died multiple times with the following query.

```sql
with all_deaths as (
select distinct
    patient_id
,   discharge_date as death_date
from claims_common.medical_claim
where discharge_disposition_code = '20'
)

select 
    patient_id
,   count(1)
from all_deaths
group by 1
having count(1) > 1
```

## How to Analyze
