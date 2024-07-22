# OrderHook

## Overview

The OrderHook program is invoked with an order number, which it uses to query the MySQL COP_AUTH table. If the order has an 'F8' track release code, it signifies potential multi-order fraud. The program then checks for any other orders from the same customer within the last day. If such orders are found, an email alert is sent for potential cancellation. These orders' details are logged in the `env-OrderHook` DynamoDB table on the old corp AWS account.


## Important Queries
```
var query = $@"select ATH.TRK_RELEASE_CD, ORD.CUSTOMER_NO, ORD.BATCH_DATE 
from COP_AUTH ATH
join COP_ORDER ORD
on ATH.BATCH_NO = ORD.BATCH_NO and ATH.ORDER_SEQ = ORD.ORDER_SEQ
where ATH.BATCH_NO = '{ordNo.Substring(0, 6)}' and ATH.ORDER_SEQ = '{ordNo.Substring(6, 2)}'";
```


```
var query = $@"select concat(ATH.BATCH_NO, ATH.ORDER_SEQ), ORD.TOTAL_FOR_POST as ordno from COP_AUTH ATH
join COP_ORDER ORD on ATH.BATCH_NO = ORD.BATCH_NO and ATH.ORDER_SEQ = ORD.ORDER_SEQ
where ORD.CUSTOMER_NO = '{customerNo}' and (ORD.BATCH_DATE = '{batchDate}' or ORD.BATCH_DATE = '{dayBefore}')
and (ORD.COP_ORDER_STATUS = 'N' or ORD.COP_ORDER_STATUS = 'F') and ATH.TRK_RELEASE_CD <> ' F8' and ATH.STATUS_CD = 'A'";
```

## Program location

The program is old and currently sits in the product teams repo. The project is under a solution called `CallCenterOrder` and the actual C# project is called `siftOrderCall`. The lambda it run on is in the old AWS corp accout and runs on a lambda named  `CallCenterOrderPlace`.