# Online Programs

## **COL335**

### Description
This program is designed to accept bank card payments online.

### Notes
The program is designed to find orders that are Account Guard Eligible. It specifically looks for Account Guard Specific Conditions.

### SQL Query
The following SQL query is used to retrieve the necessary data:

```
select ath.batch_no, ath.order_seq, ord.CUSTOMER_NO, ath.DOWNPAY_OFFER_PCT, ord.BATCH_DATE, ord.ORDER_ACK_CD, scores.EXPERIAN_SCORE, scores.EQUIFAX_SCORE_04, scores.CRED_WORTHINESS, ath.TRK_RELEASE_CD, corsp.ACTION_CD
from COP_AUTH ath
join COP_ORDER ord on (ath.BATCH_NO = ord.BATCH_NO and ath.ORDER_SEQ = ord.ORDER_SEQ)
join COP_CUSTOMER cus on (ord.CUSTOMER_NO = cus.CUSTOMER_NO)
join COP_CUS_SCORES scores on (ord.CUSTOMER_NO = scores.CUSTOMER_NO)
join COP_CORRSP corsp on (ath.BATCH_NO = corsp.BATCH_NO and ath.ORDER_SEQ = corsp.ORDER_SEQ)
where ath.STATUS_CD='W' and ath.STATUS_NO = '05'
and ath.COMPANY_CD in ('Q', 'G')
and ath.DOWNPAY_OFFER_PCT > '0.00' and ath.DOWNPAY_OFFER_PCT <= '0.50'
and ord.BATCH_DATE > '20200615'    /********** make sure this is less than 20 days **********/
and cus.STATE in ('AL','AK','AZ','AR','CT','DE','DC','FL','GA','GU','HI','ID','IL','KY','MI','MN','MS','MO','MT','NV','NH','NJ','NM','ND','OK','OR','RI','SD','TN','TX','VT','VI','WV')
and SUBSTRING(lpad(cus.RANDOM_2, 16, 0), 12, 1) in (0 , 1, 2, 3, 4, 5, 6, 7)
and ord.ORDER_ACK_CD not in ('X', 'C','D')
and scores.EXPERIAN_SCORE > '0' and scores.EQUIFAX_SCORE_04 > '0' and scores.CRED_WORTHINESS > '0'
;
```

---

## **COL337**

### Description
This program is designed to accept ACH payments on Maggies. It is part of the COP - Catalog Order Processing system. The program accepts ACH bank information to allow a down payment on a credit held order (Maggie). For the initial start up of this application, only credit down payments under $200 are allowed. $200 and over are verified with the bank, and therefore excluded from this process. This program was copied from program COL335, which accepts bank card payments. ACH payments on A/R plans are also accepted. The process is similar to COL335. The ACH transaction is written to a VSAM file for batch processing via the A/R system. A 'BA' comment is written to the CORRSP file for batch process in the MOP system (COP471). COP471 applies the remittance to the order, and sets the appropriate tracking code on the AUTH record. This program does allow for same day deletes.

### Testing
This program is called from COL341. To get to this screen: GOTO CUS3, ENTER APPL ID CSR, ENTER AN AVAILABLE ORD# & ADD. This will transfer to the COL341 screen, where you type in BA for the ACTION CD & PF1. This will transfer to COL337  REPRICE COP122 COP-AUTH STATUS=W05 COP-ORDER DISC-OFFER-CD = DG

### Schedules
n/a

### Calling programs
- COL219 - C/S ACTION NAVIGATION
- COL341 - CUSTOMER SERVICE CORRESPONDENCE ENTRY
- HAL169 - THANK YOU SCREEN WITH CREDIT MESSAGES

### Copybooks
n/a

### BDE Elimination Notes
Overview of Changes: Have orders that do a down payment through COL337 send the order to RTC/Blaze to recalculate the BDE. As part of the RTC process, the CCQ (or BRC in COBOL) record will be completed, so the program doesn't have to do that anymore. We still need to call the COP484/COP485 path within the program to create the CAS audits because DM Services still uses it for their processes. COP472 makes a decision on the order and calculates the remaining order balance. We will need to save off the auth status, prev auth status, and tracking code fields to make sure the original values are reflected on the database before the Blaze call. If the Blaze call fails, we will set it to a P09 status so it can go through RTC at night through the Batch process. COP472 call updates the...

[...]