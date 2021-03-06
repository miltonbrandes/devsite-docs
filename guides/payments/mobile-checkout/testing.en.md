# Test the integration

Before going into production, it is very important to test the payments flow, checking whether the configurations you made at the preference level are effectively reflected in the checkout.

You should check if:

+ The information about the product or service to be paid is correct.
+ If you recognize the customer’s account to send the email.
+ It offers the payment methods you wish.
+ The payment experience is adequate and the payment result is reported.

To test the integration, follow the steps below:

1. Configure the [sandbox public key](https://www.mercadopago.com/mla/account/credentials?type=basic) in your application.
2. Create the preference on your server with the access token.
3. Complete the form, entering the digits of a test card. On the expiration date you must enter any date after the current date and a 3-or 4-digit security code, depending on the card.
4. In the cardholder field you must enter the prefix corresponding to what you want to test:
        * **APRO**: Payment approved.  
        * **CONT**: Pending payment.  
        * **CALL**: Payment declined, call to authorize.  
        * **FUND**: Payment declined due to insufficient funds.  
        * **SECU**: Payment declined by security code.  
        * **EXPI**: Payment declined by expiration date.  
        * **FORM**: Payment declined due to error in form.  
        * **OTHE**: General decline.  

### Test cards to test our checkout

Use these test cards to test the different payment results.

| Country 		| Visa 				 | Mastercard        | American Express |
| ---- 		| ---- 				 | ----------        | ---------------- |
| Argentina  	| 4509 9535 6623 3704|5031 7557 3453 0604|3711 803032 57522 |
| Brazil  	| 4235 6477 2802 5682|5031 4332 1540 6351|3753 651535 56885 |
| Chile   	| 4168 8188 4444 7115|5416 7526 0258 2580|3757 781744 61804 |
| Colombia  	| 4013 5406 8274 6260|5254 1336 7440 3564|3743 781877 55283 |
| Mexico  	| 4075 5957 1648 3764|5474 9254 3267 0366|unavailable     |
| Peru    	| 4009 1753 3280 6176|unavailable      |unavailable     |
| Uruguay  	| 4014 6823 8753 2428|5808 8877 7464 1586|unavailable     |
| Venezuela  	| 4966 3823 3110 9310|5177 0761 6430 0010|unavailable     |
