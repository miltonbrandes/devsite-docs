# Other payment methods

In addition to credit or debit cards, there are other payment methods available in each country you can use to receive payments. Most of them are what we call “offline” or “cash” payment methods.

The payment methods available are:

*	ticket.
*	atm.
*	bank_transfer.
*	prepaid_card.


## Get the payment methods available

You can get the list of payment methods available by making an `HTTP GET` request:

[[[
```php
<?php

  require_once ('mercadopago.php');
  MercadoPago\SDK::configure(['ACCESS_TOKEN' => 'ENV_ACCESS_TOKEN']);

  $payment_methods = MercadoPago::get("/v1/payment_methods");

?>
```
```java
import com.mercadopago.*;
MercadoPago.SDK.configure("ENV_ACCESS_TOKEN");

payment_methods = MercadoPago.SDK.get("/v1/payment_methods");
```
```node
var mercadopago = require('mercadopago');
mercadopago.configurations.setAccessToken(config.access_token);

payment_methods = mercadopago.get("/v1/payment_methods");
```
```ruby
require 'mercadopago'
MercadoPago::SDK.configure(ACCESS_TOKEN: ENV_ACCESS_TOKEN)

payment_methods = MercadoPago::SDK.get("/v1/payment_methods")
```
]]]

The result will be an array with the payment methods and their properties:

```json
[
    {
        "id": "rapipago",
        "name": "Rapipago",
        "payment_type_id": "ticket",
        "status": "active",
        "secure_thumbnail": "https://www.mercadopago.com/org-img/MP3/API/logos/2002.gif",
        "thumbnail": "http://img.mlstatic.com/org-img/MP3/API/logos/2002.gif",
        "deferred_capture": "does_not_apply",
        "settings": [],
        "additional_info_needed": []
    },
    {
        "id": "redlink",
        "name": "RedLink",
        "payment_type_id": "atm",
        "status": "active",
        "secure_thumbnail": "https://www.mercadopago.com/org-img/MP3/API/logos/2003.gif",
        "thumbnail": "http://img.mlstatic.com/org-img/MP3/API/logos/2003.gif",
        "deferred_capture": "does_not_apply",
        "settings": [],
        "additional_info_needed": []
    },
    {
        "...": "..."
    }
]
```

## Receive a payment with a cash payment method

To receive payments in cash, you just need to get the `email`. Then, you need to make an `HTTP POST` request by sending `transaction_amount`, `payment_method_id` and the buyer's `email`:

[[[
```php
<?php  

  require_once ('mercadopago.php');
  MercadoPago\SDK::configure(['ACCESS_TOKEN' => 'ENV_ACCESS_TOKEN']);

  $payment = new MercadoPago\Payment();
  $payment->transaction_amount = 100;
  $payment->token = "ff8080814c11e237014c1ff593b57b4d";
  $payment->description = "Title of what you are paying for";
  $payment->payment_method_id = "rapipago";
  $payment->payer = array(
    "email" => "test_user_19653727@testuser.com"
  );

  $payment->save();

?>
```
```java

import com.mercadopago.*;
MercadoPago.SDK.configure("ENV_ACCESS_TOKEN");

Payment payment = new Payment();

payment.setTransactionAmount(100)
      .setToken('ff8080814c11e237014c1ff593b57b4d')
      .setDescription('Title of what you are paying for')
      .setPaymentMethodId("rapipago")
      .setPayer(new Payer("test_user_19653727@testuser.com"));

payment.save();

```
```node

var mercadopago = require('mercadopago');
mercadopago.configurations.setAccessToken(config.access_token);

var payment_data = {
  transaction_amount: 100,
  token: 'ff8080814c11e237014c1ff593b57b4d'
  description: 'Title of what you are paying for',
  payment_method_id: 'rapipago',
  payer: {
    email: 'test_user_3931694@testuser.com'
  }
};

mercadopago.payment.create(payment_data).then(function (data) {
  // Do Stuff...
}).catch(function (error) {
  // Do Stuff...
});

```
```ruby

require 'mercadopago'
MercadoPago::SDK.configure(ACCESS_TOKEN: ENV_ACCESS_TOKEN)

payment = MercadoPago::Payment.new()
payment.transaction_amount = 100
payment.token = 'ff8080814c11e237014c1ff593b57b4d'
payment.description = 'Title of what you are paying for'
payment.payment_method_id = "rapipago"
payment.payer = {
  email: "test_user_19653727@testuser.com"
}

payment.save()

```
]]]

Response:

```json
{
    ...,
    "status": "pending",
    "status_detail": "pending_waiting_payment",
    ...,
    "transaction_details": {
        "net_received_amount": 0,
        "total_paid_amount": 100,
        "overpaid_amount": 0,
        "external_resource_url": "http://www.mercadopago.com/mla/payments/ticket/helper?payment_id=123456789&payment_method_reference_id= 123456789&caller_id=123456",
        "installment_amount": 0,
        "financial_institution": null,
        "payment_method_reference_id": "1234567890"
    }
}
```

You will receive a response with a **pending** `status` until the buyer makes the payment.

In the `external_resource_url` there is a URL which contains the instructions for your buyer to make the payment. You can redirect your buyer to it or send the link for access.


> NOTE
>
> Note
>
> The buyer has from **3** to **5** days to make the payment depending on the payment method. After that, you **must** cancel it.

## Cancel a payment

You can only cancel payments whose status `pending` or `in_process`.

The payment methods available for payments in cash should be paid from 3 to 7 days depending on each one.

They do not expire automatically, so you’re required to [cancel the payment](#) after expiration.

You can view the [complete list of expiration periods](#).

## Payment confirmation period

Each payment method has their own confirmation period, in some cases it is done immediately, but it may also take up to 3 business days.

We recommend you check the [confirmation period by payment method](#).

## Refunds

If you need to make a refund to your buyer, you can do it with the Refunds API. All refunds of cash payments are made to your buyer’s Mercado Pago account.

If your buyer doesn’t have one, you will receive an email at the address sent with the payment with instructions on how to withdraw the refund.

For more information, go to [Refunds](#).

## Integrate Webpay (Chile)

Webpay is one of the payment methods available in Chile. In order to process payments with Webpay, you must send the **RUT**, **entity type**, **IP address** of the buyer, and the **financial institution** that will process the payment.

> NOTE
>
> Note
>
> Consulta todas las instituciones financieras (_financial\_institutions_) que tienes disponibles a través del recurso [payment_methods](#obten-los-medios-de-pago-disponibles):


```json
{
  "id": "webpay",
  "name": "RedCompra (Webpay)",
  "payment_type_id": "bank_transfer",
  ...
  "financial_institutions": [
    {
      "id": "1234",
      "description": "Transbank"
    }
  ]
}
```

To generate the payment using Webpay you must send the ´payment_method_id´ **webpay**, the ´identification number´ and the ´financial_institution´:

```php
<?php

require_once ('mercadopago.php');
MercadoPago\SDK::configure(['ACCESS_TOKEN' => 'ENV_ACCESS_TOKEN']);

$payment = new MercadoPago\Payment();
$payment->transaction_amount = 10000;
$payment->description = "Title of what you are paying for";
$payment->payer = array (
		"email" => "test_user_19653727@testuser.com",
		"identification" => array(
			"type" => "RUT",
			"number" => "76262349"
		),
		"entity_type" => "individual"
	);
$payment->transaction_details = array(
		"financial_institution" => 1234
	);
$payment->additional_info = array(
		"ip_address" => "127.0.0.1"
	);
$payment->callback_url = "http://www.your-site.com";
$payment->payment_method_id = "webpay";

$payment->save();

?>
```
```java
import com.mercadopago.*;
MercadoPago.SDK.configure("ENV_ACCESS_TOKEN");

Payer payer = new Payer();
payer.setEmail("test_user_19653727@testuser.com");
payer.setIdentification(new Identification("RUT", 76262349));
payer.setEntityType("individual");

TransactionDetails transactionDetails = new TransactionDetails();
transactionDetails.financialInstitution = 1234;

AdditionalInfo additionalInfo = new AdditionalInfo();
additionalInfo.ipAddress = "127.0.0.1";

Payment payment = new Payment();
payment.setTransactionAmount(10000)
      .setDescription('Title of what you are paying for')
      .setPayer(payer)
      .setTransactionDetails(transactionDetails)
      .additionalInfo(additionalInfo)
      .callbackUrl("http://www.your-site.com")
      .setPaymentMethodId("webpay");

payment.save();

```
```node
var mercadopago = require('mercadopago');
mercadopago.configurations.setAccessToken(ENV_ACCESS_TOKEN);

var payment_data = {
  ransaction_amount: 10000,
  description: 'Title of what you are paying for',
  payer: {
    email: 'test_user_3931694@testuser.com',
    identification: {
      type: "URL",
      number: "76262349"
    },
    entity_type: "individual"
  },
  transaction_details: {
    financial_institution: 1234
  },
  additional_info: {
    ip_address: "127.0.0.1"
  },
  callback_url: "http://www.your-site.com",
  payment_method_id: "webpay"
}

mercadopago.payment.create(payment_data).then(function (data) {
  // Do Stuff...
}).catch(function (error) {
  // Do Stuff...
});

```
```ruby
require 'mercadopago'
MercadoPago::SDK.configure(ACCESS_TOKEN: ENV_ACCESS_TOKEN)

payment = MercadoPago::Payment.new()
payment.transaction_amount = 10000
payment.description = 'Title of what you are paying for'
payment.payer = {
  email: 'test_user_3931694@testuser.com',
  identification: {
    type: "URL",
    number: "76262349"
  },
  entity_type: "individual"
}
payment.transaction_details = {
  financial_institution: 1234
}
payment.additional_info = {
  ip_address: "127.0.0.1"
}
payment.callback_url = "http://www.your-site.com"
payment.payment_method_id = "webpay"

payment.save();
```

> NOTE
>
> Note
>The expected `entity_type` are `individual` (Individuals) or `association` (Companies).

The response you will receive is:

```json
{
	"id": 3692089,
	"date_created": "2017-04-27T16:53:03.000-04:00",
	"date_approved": null,
	"date_last_updated": "2017-04-27T16:53:03.000-04:00",
	"money_release_date": null,
	"operation_type": "regular_payment",
	"issuer_id": null,
	"payment_method_id": "webpay",
	"payment_type_id": "bank_transfer",
	"status": "pending",
	"status_detail": "pending_waiting_transfer",
	...
	"transaction_details": {
		...
		"external_resource_url": "https://www.mercadopago.com/mlc/payments/bank_transfer/sandbox/helper/commerce?id=3692089&caller_id=251027719",
		"installment_amount": 0,
		"financial_institution": "1234",
		"payment_method_reference_id": null
	}
}
```
Direct your customer to the URL that you will find in the `external_resource_url` attribute in the `transaction_details` of the response. Upon completion of the payment, you will be redirected to the `callback_url` that you indicate, and you will receive the payment result via [Webhooks](/guides/notifications/webhooks.en.md).
