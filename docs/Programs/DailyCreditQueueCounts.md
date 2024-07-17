# Daily Credit Queue Counts
 
## **Request**
Is there any way we could get a start of day credit que count?  There is an outsourcer that starts before we do.  The only way I know of to get our volumes is from the DWIP which is real time counts, so I cant get the start of day counts.

Having to beginning day volumes better helps projections and planning.

Thanks!

Dawn White
Sr. Manager, Credit Operations
DM Services, Inc.
(608) 324-6940
Please Consider the Environment Before Printing This E-mail 

## **Requirements**
1. Provide a report to DL-DMSSupervisoryStaff@sccompanies.com
2. Don't include queues 35 or 00
3. DWIP splits it by company, but don't need that for this report. Do need to...
   - Split by Colony and Outside Clients
   - Split by Unworked and CBR Back counts
4. Report should run ideally before anyone gets in the system to work (5:00am).
   - Runs at 4:30am, started on 11/18/20

## **C# Solution**
When triggered, it will read the database tables (COP_CRED_QUEUE and IMS_CRED_QEUE), format an email report, and then send the email to the respective group. The program lives in Credit Service Repo, under the creditservice project. The microservice has it's own controller it is driven off of and it's own endpoint. There is no cross between the prequal and credit queue counts process. 

## Change Notice
RFC 25040
Work Order 25040
!7449

## **Program Structure**
- Entry point into the program. You have to use the endpoint `web/creditqueuecounts` to be able call this microservice.
- The controller will format the email and call sub processes to gather data and send email.
- [Credit_Queue_Counts_Flow_Diagram.vsdx](/.attachments/Credit_Queue_Counts_Diagram-f8a1d7d9-2be6-4f4e-81a5-a302540ae55d.vsdx)


### Queries
- COP_CRED_QUEUE

```
SELECT COP_CRED_QUEUE_QUEUE, PROCESS_STATUS, COUNT(*) from COP_CRED_QUEUE WHERE
(PROCESS_STATUS = 'U' OR
PROCESS_STATUS = 'B') AND
(COP_CRED_QUEUE_QUEUE != '35' AND COP_CRED_QUEUE_QUEUE != '00') 
group by COP_CRED_QUEUE_QUEUE, PROCESS_STATUS 
order by COP_CRED_QUEUE_QUEUE, PROCESS_STATUS;
```

- IMS_CRED_QUEUE

```
SELECT (CASE WHEN IMS_CRED_QUEUE_QUEUE > '29' THEN (IMS_CRED_QUEUE_QUEUE - 8) ELSE IMS_CRED_QUEUE_QUEUE END) as 'QUEUE', PROCESS_STATUS as 'Type', COUNT(*) as 'COUNT' 
from IMS_CRED_QUEUE WHERE
(PROCESS_STATUS = 'U' OR
PROCESS_STATUS = 'B') AND
(IMS_CRED_QUEUE_QUEUE != '17' AND IMS_CRED_QUEUE_QUEUE != '00') 
group by IMS_CRED_QUEUE_QUEUE, PROCESS_STATUS 
order by IMS_CRED_QUEUE_QUEUE, PROCESS_STATUS;
```

## **New Account**
In October of 2022 Credit Queue Counts was migrated to the new AWS account. It was made multi-regional, running on both us-east-1 and us-east-2. The tfstate file is kept in S3 bucket:

- **Dev:** S3: dev-terraform-infrastructure/projects/CreditQueueCounts
- **UAT:** S3: uat-terraform-infrastructure/projects/CreditQueueCounts 
- **PRD:** S3: prd-terraform-infrastructure/projects/CreditQueueCounts  

---

## **SNS Notification**
Go to AWS SNS and subscribe to the below topics to receive email reports.

***New Account***
Topics:

- dev-EmailService-CreditQueueCounts
- uat-EmailService-CreditQueueCounts
- prd-EmailService-CreditQueueCounts

***Old Account***
Topics:

- ~~DevCreditQueueCounts~~
- ~~UATCreditQueueCounts~~
- ~~PRDCreditQueueCounts~~

---

## ARNs
***New Account***

- arn:aws:sns:us-east-1:818019740876:dev-EmailService-CreditQueueCounts

- arn:aws:lambda:us-east-1:867146357054:function:uat-CreditQueueCounts
- arn:aws:lambda:us-east-2:867146357054:function:uat-CreditQueueCounts

- arn:aws:lambda:us-east-1:931583720194:function:prd-CreditQueueCounts
- arn:aws:lambda:us-east-2:931583720194:function:prd-CreditQueueCounts

***Old Account***
ARNs

- ~~arn:aws:sns:us-east-1:194344284237:DevCreditQueueCounts~~
- ~~arn:aws:sns:us-east-1:194344284237:UATCreditQueueCounts~~
- ~~arn:aws:sns:us-east-1:194344284237:PRDCreditQueueCounts~~

[SNS Notification Documentation](https://sccompanies.visualstudio.com/CreditTeam/_wiki/wikis/CreditTeam_wiki/2529/SNS-Notifications)

---

## **API Gateway**
A new resource was added under dev-web api called creditqueuecounts

***New Account***
URL Endpoints

- https://api.dev.sccompanies.com/dev-creditqueuecounts/creditqueuecounts

- https://gxw40afvka.execute-api.us-east-1.amazonaws.com/uat/creditqueuecounts
- https://rkh0ttvvl6.execute-api.us-east-2.amazonaws.com/uat/creditqueuecounts/
- https://api.uat.sccompanies.com/uat-creditqueuecounts/creditqueuecounts

- https://1sa7xh2ob4.execute-api.us-east-1.amazonaws.com/prd/creditqueuecounts
- https://at2ef4zsv6.execute-api.us-east-2.amazonaws.com/prd/creditqueuecounts/
- https://api.prd.sccompanies.com/prd-creditqueuecounts/creditqueuecounts

***Old Account***
URL Endpoints

- ~~https://apidev.sccompanies.com/web/creditqueuecounts~~
- ~~https://apiuat.sccompanies.com/web/creditqueuecounts~~
- ~~https://api.sccompanies.com/web/creditqueuecounts~~

Each endpoint has a key associated with it that can be found in the gateway. 
[API Gateway Documentation](https://sccompanies.visualstudio.com/CreditTeam/_wiki/wikis/CreditTeam_wiki/2526/API-Gateway) 

---

## **Automic Trigger Task**
An Automic task was created that will make a http request using curl. The task was added to @TIME schedule to be run at 4:30am daily (Mon-Sun).

***New Account***
Task Name

- `JOBS.WIN.LAMBDA.CREDITQUEUECOUNTS` invokes command `curl -X POST -H "x-api-key: " &APIENV#/&RUNCLIENTLC#-creditqueuecounts/creditqueuecounts`

***Old Account***
Task Name

- `JOBS.WIN.LAMBDA.CREDITQUEUECOUNTS` invokes command `curl -X POST -H "x-api-key: key-name" &APIENV#/web/creditqueuecounts`

The script is initiated on server AECPAS-WINAUT01.

Job Name in @TIME 
LAMBDA.credit.queue.counts

Testing
- Incorrectly name SNSCredQueueCounts environment name - For Task to have "ERROR" in the message
- Use uat url on dev with dev x-api-key (this change is in Automic task) - For testing lambda connection

---

## **Error Handling**
If the email has the word 'ERROR' anywhere in it, the task will fail with a RC=1. Data Control will create a Numera Ticket and assign it to on-call (just like a batch failed job), then will call/contact the on-call person after 8:00am.

If the database reads have errors and SNS worked, then users will get the email report that tells them to contact Credit Team, and Data Control will follow above procedure. 

If failed to send SNS messages but report was created, users will _not_ receive email. Data Control will get RC=1 and follow above procedure. In this case, you can find the email in Cloudwatch logs for CreditService Lambda and manually send it to the users. 

If failed to trigger lambda, task will fail in Automic and Data Control will follow above procedure. 

On successful run, the task will receive RC=0 and users will receive email populated with report. 








