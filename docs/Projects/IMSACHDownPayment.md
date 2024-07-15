# IMS ACH DownPayment

## API Overview

**REPO:**  
ImsAchDownpayment

**Swagger link:**  
[https://api.dev.sccompanies.com/dev-imsachdownpayment/swagger/index.html](https://api.dev.sccompanies.com/dev-imsachdownpayment/swagger/index.html)

**IMS ACH DP API URL:**  
[https://api.dev.sccompanies.com/dev-imsachdownpayment/api/ImsAchDownpayment](https://api.dev.sccompanies.com/dev-imsachdownpayment/api/ImsAchDownpayment)


**S3 bucket:**  
dev-imsachdownpayment-east1  
dev-imsachdownpayment-east2 - nothing is written in it.

**Firehose Delivery streams:** (delivery destination is always dev-imsachdownpayment-east1)  
dev-imsAchDpApiRequest  
dev-imsAchDpApiResponse  
dev-imsAchDpBlazeRequest  
dev-imsAchDpBlazeResponse

**Timestream database:**  
dev-ImsAchDownpayment

**Timestream tables:**  
dev-ImsAchDpApiRequest  
dev-ImsAchDpApiResponse  
dev-ImsAchDpBlazeRequest  
dev-ImsAchDpBlazeResponse

**Lambda:**  
dev-ImsAchDownpayment

**SNS Topic:**  
dev-IMSACHDownpaymentSNS

**Query for test data:**  

```
select concat(ath.BATCH_NO, ath.ORDER_SEQ, ath.ORDER_FILL) as ORDER_NO, ath.SUB_COMP_CD, ath.CORPORATE_CD, ord.CUSTOMER_NO, concat(ath.STATUS_CD, ath.STATUS_NO) as STATUS_CD, ath.DOWNPAY_OFFER_PCT, ath.DOWNPAY_AMT, ord.AMOUNT_DUE, ord.BATCH_DATE, ath.LTR_DISC_CD, ath.TRK_SUSPEND_CD, ath.TRK_RELEASE_CD
from IMS_AUTH as ath
inner join IMS_ORDER as ord on (ath.BATCH_NO = ord.BATCH_NO and ath.ORDER_SEQ = ord.ORDER_SEQ and ath.ORDER_FILL = ord.ORDER_FILL)
inner join IMS_CUSTOMER as cus on (ord.CUSTOMER_NO = cus.CUSTOMER_NO)
where ath.STATUS_CD = 'W' 
and ath.STATUS_NO = '05'
and ath.DOWNPAY_OFFER_PCT > '0.00' and ath.DOWNPAY_OFFER_PCT < '0.50'
and ath.ADD_DATE > 20230901
limit 100;
```

## Down Payment Rules

IMS ACH Down Payment Rules:

Customer with '0' credit limit and order in held status are not allowed to make ACH payment.
	
```
        select concat(ath.BATCH_NO, ath.ORDER_SEQ) as orderNbr, 
	    cus.CUSTOMER_NO as customerNbr, 
	    cus.CREDIT_LIMIT as customerLimit,
	    concat(ath.STATUS_CD, ath.STATUS_NO) as authStatus, 
	    ath.TRK_RELEASE_CD as trkReleaseCd, 
	    ath.DOWNPAY_AMT as downPaymentAmt, 
	    ath.DOWNPAY_OFFER_PCT as downPaymentOfferPct,
	    ord.REMITTANCE as remittanceAmt,
	    ord.BALANCE_DUE as balanceDue
	    from IMS_AUTH as ath 
	    inner join IMS_ORDER as ord on ath.BATCH_NO = ord.BATCH_NO and ath.ORDER_SEQ = ord.ORDER_SEQ
	    inner join IMS_CUSTOMER as cus on cus.CUSTOMER_NO = ord.CUSTOMER_NO
	    where (ath.STATUS_CD = 'P' or 'W') and cus.CREDIT_LIMIT = 0
	    order by ath.SEQUENCE_ID_KEY desc
	    limit 5;
```
	
If BLOCK_ACH_CD = 'Y' for customer on IR_CUSTOMER, ACH transaction not allowed.
	
```
    select CUSTOMER_NO, BLOCK_ACH_CD from IR_CUSTOMER where BLOCK_ACH_CD = 'Y' order by SEQUENCE_ID_KEY desc limit 1;

    select CUSTOMER_NO, BLOCK_ACH_CD from IR_CUSTOMER where CUSTOMER_NO = '71022645608';
```

	
If customer not found in on IR_CUSTOMER, ACH transaction not allowed.
	
If customer not found on IMS_CUS_SCORES table or if found customer, but Experian score and Equifax-score-10 is zero then ACH payment not allowed.

```
    select CUSTOMER_NO, COMPANY_CD, EXPERIAN_SCORE, EQUIFAX_SCORE_10 
	from IMS_CUS_SCORES 
	where EXPERIAN_SCORE = 0 and EQUIFAX_SCORE_10 = 0 
	order by SEQUENCE_ID_KEY desc
	limit 10;
    
    select CUSTOMER_NO, COMPANY_CD, EXPERIAN_SCORE, EQUIFAX_SCORE_10 
	from IMS_CUS_SCORES 
	where CUSTOMER_NO = '2099669596' and COMPANY_CD = '02';
```	
	
Only two ACH payment allowed to customer per company per season
```
    select CUSTOMER_NO, concat(SUB_COMP_CD, CORPORATE_CD) as companyCd, ACH_COUNT from IMS_CUSTOMER 
	where ACH_COUNT > 2 
	order by SEQUENCE_ID_KEY desc 
	limit 2;
	
	select CUSTOMER_NO, ACH_COUNT from IMS_CUSTOMER
	where CUSTOMER_NO = '2099669359' and SUB_COMP_CD = '0' and CORPORATE_CD = '2' ;
```	
	
Can not accept ACH payment > $200

ACH not allowed if customer marked fraud in IR_CUSTOMER

```
	select CUSTOMER_NO, concat(SUB_COMP_CD, CORPORATE_CD) as companyCd, BAD_DEBT_CD from IMS_CUSTOMER 
	where BAD_DEBT_CD = 'F' 
	order by SEQUENCE_ID_KEY desc 
	limit 2;
	select CUSTOMER_NO, concat(SUB_COMP_CD, CORPORATE_CD) as companyCd, BAD_DEBT_CD from IMS_CUSTOMER
	where CUSTOMER_NO = '2099669359' and SUB_COMP_CD = '0' and CORPORATE_CD = '2' ;
```

ACH payment allowed only on held order, so if STATUS_CD != 'P' or 'W' or 'H', ACH payments not allowed. STATUS_CD in IMS_AUTH

If down payment amount is less than minimum down payment required, then ACH payment is not allowed.
```
        COMPUTE WS-MINIMUM-PAYMENT
	               = (ORD-BALANCE-DUE * WS-REQUESTED-PERCENT)
```
	Note - 
	From IMS_AUTH, we can get DOWNPAY_OFFER_PCT, and DOWNPAY_AMT
	From IMS_ORDER , we can get BALANCE_DUE
Down payment amount cannot be more than order BALANCE_DUE. (We can get BALANCE_DUE from COP_ORDER)
ACH is allowed with checking or saving account type
*Check for ACH transaction in AR transaction file. If there is pending transaction, payments not allowed. We might need to review this scenario.

---

Following are my are additional findings on ACH down payment rules for outside clients for specific message type on screen(highlighted):
 
'SPEED PAY OPTION NOT AVAILABLE' results on following screens, when order in Process/pend or Waiting status and customer credit limit is zero.
 
DSO screen (Program – IOL324)
CSR screen (Program – IOL341)
AUTH screen (Program – IOL347)
 
Example:


 
'ACH PAYMENT OPTION NOT AVAILABLE' results on CSR screen (Program -IOL341) under various conditions which are checked in program IMM239and they are as follows:
	- If AR blocked ACH payment on AR_CUSTOMER table
	- If customer not found on COP_CUS_SCORES table or if found customer, but Experian score and Equifax-score-10 is zero.
	- There is another rule for corporate code ‘4’, only two ACH payment allowed, I think this not relevant anymore.
	- At some point for outside clients ACH payment over 200 wasn’t accepted thru screen, they were asked to mail the check, not sure if that is still applicable (Found in program IOL337).
