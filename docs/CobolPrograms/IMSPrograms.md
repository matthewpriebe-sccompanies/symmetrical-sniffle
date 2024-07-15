# IMS Programs

## IMM110

Description: 
CREATES A REPORT OF P04, P05, P06, P07, W04,
AND W06.                               

THIS PROGRAM READS THROUGH THE IMS-AUTH DATABASE LOOKING FOR  
STATUS'S LISTED ABOVE AND CREATES A REPORT BASED ON WHAT      
COMPANY THE PROGRAM IS CURRENTLY BEING RUN FOR.               


Schedules: 
*ALL
	LTDALL 


Jobs: 
*110J
	LTD110J

Notes:

The way it is written, It will not create DR Leonards report in AMK110J because the ctl-company-cd = '04' so the sub-company-code is '0' and likewise in DRL110J the ctl-company-cd is 'A4' so the 04 records won't be on the report.

9/28/22: DRL110-01 reports are now found in AMK 110-01 reports since the merging. Used by Tasha Smith


B00 sql - Change Corporate Cd based on company being queried.
Select * from IMS_AUTH
where ((STATUS_CD = 'P' and STATUS_NO in ('04', '05', '06', '07')) or 
(STATUS_CD = 'W' and STATUS_NO in ('04','06'))) and CORPORATE_CD = '4'
order by CORPORATE_CD asc, SUB_COMP_CD asc, ADD_DATE asc;

Parm Card usage: 
CTL-TEST = Used only to change checkpoint from 5000 to 2
CTL-DISPLAY-ON = not used
CTL-UPDATE-SW = not used
CTL-COMPANY-CD = used to distinguish which records to write

---

## IMM430

Description: 
PROCESS NEEDS AUTHORIZATION

THIS PROGRAM SELECTS FROM THE AUTH DB ALL 'P06' UPDATING 
THEM TO 'W06'. IT CALLS IMM108 TO REPORT.                

Updates IMS-AUTH and IMS-CRED-QUEUE

Schedules: 
*ALL

Jobs: 
*430J

Copybooks: 

COR742C	STATE CD TABLE
IMR108C	108 ATH RECORD
COR946C	Look up table
IMR720L	Lookup copybook

Notes:

The program pulls all records with corporate cd passed in parm card. It does not split DRL from AMK when writing to report. When reports were checked, both DRL and AMK records were found in the DRL430/431 and AMK430/431 reports. 


Process Normal:
Select * from IMS_AUTH
where CORPORATE_CD = '4' AND 
((STATUS_CD = 'P' AND STATUS_NO = '06') OR
(STATUS_CD = 'P' AND STATUS_NO = '08'));


E70-UPDATE Section:
If Ath-Status = W06
	Continue
Else If Ath-Status = P06
	Set status to W06
	Update auth MOP run number to MPC MOP run number
Else 
	Set status to W05
	
If Build-control-rec upsi on and prev status is P06
	If record found in IMS-CRED-QUEUE
		Continue
	If not found
		Set zeros to 
			CCQ-MOVED-FROM-Q
			CCQ-UPDATE-DATE 
			CCQ-UPDATE-TIME 
	F20,F30 --> sets a bunch of fields for IMS-CRED-QUEUE 
	Updates or inserts to IMS-CRED-QUEUE


---

## IMM434

Jobs: 
*434J	UPSI(00000000)	Yes update, no requests
*434J2	UPSI(01000000)	Requests to magnum
*434J3 	UPSI(00010000)	Request missing rpts
*434J5	UPSI(10010000)	No update, request missing rpts


Copybooks: 


Notes:


           UPSI-0 IS UPSI-UPDATE-DB-SW                                          
              OFF IS UPSI-UPDATE-DB                                             
           UPSI-1 IS UPSI-REQUESTS-TO-MAGNUM-SW                                 
               ON IS UPSI-REQUESTS-TO-MAGNUM                                    
           UPSI-2 IS UPSI-REQUEST-MISSING-APPR-SW                               
               ON IS UPSI-REQUEST-MISSING-APPR                                  
           UPSI-3 IS UPSI-REQUEST-MISSING-RPTS-SW                               
               ON IS UPSI-REQUEST-MISSING-RPTS                                  
           UPSI-4 IS UPSI-RUN-2-MAGNUM-FILES-SW                                 
               ON IS UPSI-RUN-2-MAGNUM-FILES                                    
           UPSI-5 IS UPSI-NO-MAGNUM-FILES-SW                                    
               ON IS UPSI-NO-MAGNUM-FILES                                       
           UPSI-6 IS UPSI-RUN-TEST-BATCH-SW                                     
               ON IS UPSI-RUN-TEST-BATCH                                        
           UPSI-7 IS UPSI-TEST-RUN-SW                                           
               ON IS UPSI-TEST-RUN.  

---

## IMM444

Description: 
THIS PROGRAM MERGES YTD TRACKING WITH THE 
DAILY TRACKING FILE.                      


Schedules: 
*ALL

Jobs: 
*444J

Copybooks: 
 CREDIT-TRACKING-RECORD      
COPY IMR444C.                
                             
 R490-MOP-TOTALS             
COPY IMR490C9.               
                             
 491 CALL AREA               
COPY COR491C.                
                             
 TRACKING-CODE-TABLE         
*COPY COR723C.                
COPY IMR723C.                
                             

Notes:


Doesn't look like LTD/LSC specifically needs to be handled, as the program is file driven. 

For Figis and MKC, special coding is in there to print discount details to a PRINT-FILE-3 (SYS008) and PRINT-FILE-4 (SYS009), which are not used in DRL or AMK jobs. 
