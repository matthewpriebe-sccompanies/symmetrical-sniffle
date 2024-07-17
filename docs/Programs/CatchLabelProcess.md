# Catch Label Process
 
Order Velocity enhancement  - Customer exceeds order velocity count (F8 tracking release code)

- Assign a new tracking code - ??
    - Decline the order due to order velocity (already done)
	- Add customer correspondence record? (do we do this already?)
	- Find all other Approved orders that caused the velocity count to be exceeded (don’t include Food)
		- Cancel orders not physically shipped  
			- Determine what’s shipped on an order by looking at shipitem database
		- Assign a new tracking code of ??  - so what about partial shipments (some items on BO?) Which tracking code to apply?
		- Credit the customer where appropriate  
		- Add customer correspondence record(s)
	- Do not generate a letter
	- Send those order(s) to HJ or team to track them down for those not physically shipped
 
 
Customer calls in with complaint about being charged for something they did not place an order for

- Cancel that order if appropriate
- Credit the customer for what they were charged
- Assign a new tracking code of ??
- Add customer correspondence record
- Do not generate a letter
- Find all other Approved orders – how would we know what was good & bad?
	- Would they walk through all these with the customer on the phone?
- Send those order(s) to HJ or team to track them down for those not physically shipped
	

Catch Label or Stop order - Action Code LS, Program COP385
Kumar, Suresh - Aug 18, 2022

**How can you find out if shipping labels can be caught and order can be stopped?**

On CSR screen of an order action LS is entered. A job/program COP385J/COP385 runs every 15 minutes to look for LS action code, generates a report about labels if they can be stopped or not. That report helps customer service to make decision or take next action on orders usually on fraud orders.
