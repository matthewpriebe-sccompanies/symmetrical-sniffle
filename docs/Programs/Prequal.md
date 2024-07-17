# Prequal

## New Endpoints

### Mason
- [https://api.dev.sccompanies.com/dev-mason-prequal/mason-prequal](https://api.dev.sccompanies.com/dev-mason-prequal/mason-prequal)
- [https://api.uat.sccompanies.com/uat-mason-prequal/mason-prequal](https://api.uat.sccompanies.com/uat-mason-prequal/mason-prequal)
- [https://api.prd.sccompanies.com/prd-mason-prequal/mason-prequal](https://api.prd.sccompanies.com/prd-mason-prequal/mason-prequal)

### Colony
- [https://api.dev.sccompanies.com/dev-colony-prequal/prequal/](https://api.dev.sccompanies.com/dev-colony-prequal/prequal/)
- [https://api.uat.sccompanies.com/uat-colony-prequal/prequal](https://api.uat.sccompanies.com/uat-colony-prequal/prequal)
- [https://api.prd.sccompanies.com/prd-colony-prequal/prequal](https://api.prd.sccompanies.com/prd-colony-prequal/prequal)

## Response Codes
- 00 - Good prequal offer for brand new person (updates promo tables)
- 10 - Denied prequal from Blaze rules
- 11 - No Hit from Bureau Indicators
- 12 - Denied from Bureau Indicators
- *13 - Denied Bad Debt
- *14 - Denied Slow Pay
- 20 - Service deactivated
- 30 - Existing prequal offer
    - IMS - found on IMS_PROMO
    - Colony - valid offer with prequal flag = "Y"
- *31 - Existing Buyer with valid limit for the company
- *32 - Existing Buyer, new prequal offer
- *33 - Existing Promo with valid limit for the company
- *34 - Existing Promo, new prequal offer
- *40 - Denied Vermont
- 90 - Internal Error
- 91 - Invalid Company Code Error
- 92 - Database Error (reading or connection issues)
- 93 - Bureau Error (executing or parsing returned info)
- 94 - Blaze Error (executing or parsing returned info)
- 95 - Insert Database Error (writing to a table)
- 96 - Spectrum Error (anything from calling to storing returned values)
- *97 - Matchcode Error
- *Colony Specific Code

## Expiration Date Logic
### IMS
- If they get a prequal limit November through April, we will expire those July 31th.
- If they get a prequal limit May through October, we will expire those January 31st.

### Colony
- If they get a prequal limit November through April, we will expire those June 30th.
- If they get a prequal limit May through October, we will expire those December 31st.

## Prequal Audit RedShift
Detailed list - PreQual Redshift table Audits

### Production Table
crd_s.prequal

### Dev Table
crd_dev_s.prequal
Credit team – you’ll need to expand the EXTERNAL TABLE folder and then you will see the prequal table

## Tips and Tricks:
1. Add year, month, day when querying to make it execute faster.
2. Colony Key = Cookie Id. This may not be unique
3. Mason Key = Finder Number

## SampleQueries

```
select count(pres_offeramount1) 
from crd_s.prequal 
where pres_offeramount1 > 0 and responsecode = '34';

select pres_requestid, customerno, preq_firstname, preq_lastname, preq_address1, company, date, responsecode, 
pres_offeramount1, pres_offercode1, pres_offermessage1, pres_offerexpirationdate1  
from crd_s.prequal 
where company = 'colony' and pres_clientid = 'Q' and date > '01/22/2020'
limit 25;

Select month, day, key, responsecode, blzq_riskscore, blzs_riskrank, blzs_rules_idfield, blzs_rules_valfield, burs_nohitindicator, burs_scoretext, burs_reasoncode1, burs_reasoncode2, burs_reasoncode3, burs_reasoncode4, burs_retcode  
from crd_s.prequal 
where preq_clientid = 'G2' and year = '2020' order by key;

Select * from crd_s.prequal where year = '2020' and month = '08' and day = '01' limit 1;

Select key, companyid, responsecode, pres_score, blzq_findernbr,blzs_riskrank, burs_irregularreportdeceasedindicator, burs_consumerstatementindicator, burs_abnormalreportindicator, burs_addressmismatchindicator, burs_nohitindicator, pres_offeramount1, year, month, day 
from crd_s.prequal 
where company = 'colony' and year = '2020' and month = '05' 
order by year, month, day asc;

Select * from crd_dev_s.prequal where KEY = '10207122550615862045249' AND
year = '2020' and month = '04' and day = '06'  order by date;

select companyid, pres_responsecode, count(*)
from crd_s.prequal 
where year = '2023' and month = '01' and day in ('04', '05') 
group by companyid, pres_responsecode;

```

## Ad Codes

Advertisement codes (Ad codes or Adv codes) are unique codes that identify a marketing season, year and catalog. These codes are used for sales reporting. The LST team owns and maintains the table, along with Ami Barker and Kevin Geib's team. For prequal, Ami Barker assigns the ad code using a screen on for production. LST team (Deb or Heidi) assign the same codes in development. In the past, with permission giving from LST team, I have copied the new production codes into dev to test. 

Ad codes are stored on the MRK_AD_CODE table in MySQL. There is no easy key that we can use to find the prequal codes, so this was the fastest way I could grab only the records I needed. 

Select * from MRK_AD_CODE where DESC_1 like '%PREQUAL%';
02/15/22 - Code enhancement to look up Adv Code based on Company_CD + Season_Year + %PREQUAL%, which removed the need to hardcode each ad code.
Season End Year End Prequal update every 6 months 
When business requests a new season cut over, CRD team will need to update the SeasonYear field in the environment variable of the lambda. Ex: S22 or F22.
Note: The format of DESC_1 in the database should follow this format example MW S22 SI PREQUAL
Not doing so will result in default 9999[suffix] adv codes and warning emails to CRD team.

Typically, the ad codes are active in December/January and June/July timeframes

Ad code load new, current goes to prev
Already a valid offer > write to hold file and reapply after end of season exp. 
	Try to reapply any limits that were in the hold file 
Plan code - yes update for both 
	Buyers - happens when ad code is added.
	
Read MRK ad code table and pull plan codes from that too when applying. 

Installment then open plan > July 1st, only limits are updated, not ad code or plan codes. 
	Equal limit or lower, then update limit
	
Amy Barker - promo updates
Credit - customer updates

Anyone who received prior plan O or plan I will be forced into the same plan in this mailing. 
	Promo - plan will not change on the customers.
	If promo has higher expiration date, then only will the record be added/updated. 
	Always loading offer ad code and plan, but may not upload limit. 

Ad code and plan code don't expire. Only limits will expire. 

## Prequal Audit Locations

Production Table: **crd_s.prequal**

Dev Table: **crd_dev_s.prequal**



Cloudwatch Log groups
API Gateway Logs
These are good to look at to see if user is hitting the api gateway. Either they have a permissions issue, api key issue, or possibly not hitting it at all. 

API-Gateway-Execution-Logs_lalzi7idnl/prd (colony east-1)
API-Gateway-Execution-Logs_eehsrcydhi/prd (colony east-2)

API-Gateway-Execution-Logs_e9es7y2415/prd (mason east-1)
API-Gateway-Execution-Logs_fppbw90w77/prd (mason east-2)

## ZipCode Test Cases

The test cases have been expanded to cover more scenarios and now rely on the last two digits of the zip code. Below are more details around the changes:

- 22 test cases
- Use the last 2 digits of the zip code to trigger each test scenario
- Each test case is associated with a rank. See the "Rank-Score Mapping" tab to know what you can expect for test cases between 6 and 19.
    - Example: zip ending in ***10 = rank 10, which maps to score 890
- I have a note on some of the test cases indicating there will be different results due to the volume split test in our rules engine. I gave an example of each scenario between the different companies for that row. 

See the "Volume Split Test" tab for expectations.
