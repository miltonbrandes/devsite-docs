# Receive card payments

With MercadoPago you can collect card information in a secure way, while keeping control over the shopping experience offered to your users.

## Collect card information

The card information is collected from your buyer’s browser.**For security reasons, it is very important that these data never reach your servers.**

Mercado Pago has a Javascript library to help you do this simply and safely.

### 1. Include MercadoPago.js

To use this library, you must first enter the following code in our checkout:

```html
<script src="https://secure.mlstatic.com/sdk/javascript/v1/mercadopago.js"></script>
```

> NOTE
>
> Note
>
> The library must **always** be imported from https://secure.mlstatic.com.


### 2. Set up your public key

Your public key is what identifies you so that you can safely collect the card information. The public key must be uploaded after including _MercadoPago.js_ and before making a request.

```javascript
Mercadopago.setPublishableKey("TEST-b3d5b663-664a-4e8f-b759-de5d7c12ef8f");
```

> NOTE
>
> Note
>
> This is a public key of the test environment. To capture real cards you will need to replace them with your own [production public key](https://www.mercadopago.com/mla/account/credentials).


### 3. Collect card information

#### Form

The next step is to collect the card information. To do this it is important to have a form that uses the following `data-checkout` attributes:

```html
<form action="" method="post" id="pay" name="pay" >
    <fieldset>
        <ul>
            <li>
                <label for="email">Email</label>
                <input id="email" name="email" value="test_user_19653727@testuser.com" type="email" placeholder="your email"/>
            </li>
            <li>
                <label for="cardNumber">Credit card number:</label>
                <input type="text" id="cardNumber" data-checkout="cardNumber" placeholder="4509 9535 6623 3704" />
            </li>
            <li>
                <label for="securityCode">Security code:</label>
                <input type="text" id="securityCode" data-checkout="securityCode" placeholder="123" />
            </li>
            <li>
                <label for="cardExpirationMonth">Expiration month:</label>
                <input type="text" id="cardExpirationMonth" data-checkout="cardExpirationMonth" placeholder="12" />
            </li>
            <li>
                <label for="cardExpirationYear">Expiration year:</label>
                <input type="text" id="cardExpirationYear" data-checkout="cardExpirationYear" placeholder="2015" />
            </li>
            <li>
                <label for="cardholderName">Card holder name:</label>
                <input type="text" id="cardholderName" data-checkout="cardholderName" placeholder="APRO" />
            </li>
            <li>
                <label for="docType">Document type:</label>
                <select id="docType" data-checkout="docType"></select>
            </li>
            <li>
                <label for="docNumber">Document number:</label>
                <input type="text" id="docNumber" data-checkout="docNumber" placeholder="12345678" />
            </li>
        </ul>
        <input type="hidden" name="paymentMethodId" />
        <input type="submit" value="Pay!" />
    </fieldset>
</form>
```

> WARNING
>
> Important
>
> Fields that contain sensitive data do not have the `name` attribute, so they will never reach your servers.


#### Get the document type

The document type and number are among the required fields.

It is possible to get the list of available documents:


```javascript
Mercadopago.getIdentificationTypes();
```

#### Get the card’s payment method

It is important to get the card’s payment method so that payment can be made.

In order to get the payment method, use `MercadoPago.getPaymentMethod(jsonParam,callback)`. This method accepts two parameters: an object and a callback function.

```javascript
Mercadopago.getPaymentMethod({
    "bin": bin
}, setPaymentMethodInfo);
```

The `bin` corresponds to the first 6 card digits, which identify the payment method and the issuing bank.

The callback receives a status and a response. The function must store the response id in the `paymentMethodId` field (input hidden), for example:

```javascript
function setPaymentMethodInfo(status, response) {
    if (status == 200) {
        paymentMethod.setAttribute('name', "paymentMethodId");
        paymentMethod.setAttribute('type', "hidden");
        paymentMethod.setAttribute('value', response[0].id);

        form.appendChild(paymentMethod);
        } else {
            document.querySelector("input[name=paymentMethodId]").value = response[0].id;
        }
    }
};
```

#### Collect data

Before submitting the form, you must capture the `submit` event and use the `Mercadopago.createToken(form, sdkRespondeHandler);` method.

```javascript
doSubmit = false;
addEvent(document.querySelector('#pay'), 'submit', doPay);
function doPay(event){
    event.preventDefault();
    if(!doSubmit){
        var $form = document.querySelector('#pay');

        Mercadopago.createToken($form, sdkResponseHandler); // The function "sdkResponseHandler" is defined below

        return false;
    }
};
```

By submitting the `form` and using the `data-checkout` attributes, all fields are captured.

The `createToken` method will return a card_token, which is the secure representation of the card:

```json
{
    "id": "ff8080814cbd77a8014cc",
    "public_key": "TEST-b3d5b663-664a-4e8f-b759-de5d7c12ef8f",
    "card_id": null,
    "luhn_validation": true,
    "status": "active",
    "date_used": null,
    "card_number_length": 16,
    "date_created": "2015-04-16T13:06:25.525-04:00",
    "first_six_digits": "450995",
    "last_four_digits": "3704",
    "security_code_length": 3,
    "expiration_month": 6,
    "expiration_year": 2017,
    "date_last_updated": "2015-04-16T13:06:25.525-04:00",
    "date_due": "2015-04-24T13:06:25.531-04:00",
    "live_mode": false,
    "cardholder": {
        "identification": {
            "number": "23456789",
            "type": "type"
        },
        "name": "name"
    }
}
```

The second field of `createToken` method is the `sdkResponseHandler`, which is a callback function that will run when creating the `card_token`.

We will use this to create a hidden field (input hidden), and store the `id` value in order to send the form to your servers.

```javascript
function sdkResponseHandler(status, response) {
    if (status != 200 && status != 201) {
        alert("verify filled data");
    }else{
        var form = document.querySelector('#pay');
        var card = document.createElement('input');
        card.setAttribute('name', 'token');
        card.setAttribute('type', 'hidden');
        card.setAttribute('value', response.id);
        form.appendChild(card);
        doSubmit=true;
        form.submit();
    }
};
```

You can [download the complete example here](#).

## Receive card payments

From the parameters sent in the `POST`, you should get the `card_token` id to make a single payment.

A `card_token` **is valid for 7 days** and can be used only once.

To make the payment, simply make an API call:

[[[
```php
<?php  

  require_once ('mercadopago.php');
  MercadoPago\SDK::configure(['ACCESS_TOKEN' => 'ENV_ACCESS_TOKEN']);

  $payment = new MercadoPago\Payment();

  $payment->transaction_amount = 100;
  $payment->token = "ff8080814c11e237014c1ff593b57b4d";
  $payment->description = "Title of what you are paying for";
  $payment->installments = 1;
  $payment->payment_method_id = "visa";
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
      .setInstallments(1)
      .setPaymentMethodId("visa")
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
  installments: 1,
  payment_method_id: 'visa',
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
payment.installments = 1
payment.payment_method_id = "visa"
payment.payer = {
  email: "test_user_19653727@testuser.com"
}

payment.save()

```
]]]

> The required fields to be sent are `token`, `transaction_amount`, `payment_method_id` and `payer.email`.

Response:

```json
{
    "status": "approved",
    "status_detail": "accredited",
    "id": 3055677,
    "date_approved": "2017-02-23T00:01:10.000-04:00",
    "payer": {
        ...
    },
    "payment_method_id": "master",
    "payment_type_id": "credit_card",
    "refunds": [],
    ...
}
```

That's all, the response will indicate the payment status (`approved`, `rejected` or `in_process`).

> NOTE
>
> See more information about [response handling](#manejo-de-respuestas).

## Recibir un pago en cuotas

In order to benefit from the [promotions](https://www.mercadopago.com.ar/promociones) offered by MercadoPago, it is important to submit the `issuer_id` and `installments` field when creating a payment.

The installments field corresponds to the number of `installments` selected by the buyer. The `issuer_id` is the issuing bank of the card.

In order to get the installments available:


```javascript
Mercadopago.getInstallments({
    "bin": bin,
    "amount": amount
}, setInstallmentInfo);
```

The response includes the `issuer_id` to be sent, and the recommended message to be shown in each of the available installments specifying the amount to be paid:

```json
[
  {
    "payment_method_id": "amex",
    "payment_type_id": "credit_card",
    "issuer": {
        "id": "310",
        ...,
        {
            "installments": 3,
            "installment_rate": 18.9,
            "discount_rate": 0,
            "labels": [
            ],
            "min_allowed_amount": 2,
            "max_allowed_amount": 250000,
            "recommended_message": "3 cuotas de $ 396,33 ($ 1.189,00)",
            "installment_amount": 396.33,
            "total_amount": 1189
        }
        ...,
    ]
  }
]
```

> NOTE
>
> Note
>
> Due to [Resolution E 51/2017](https://www.boletinoficial.gob.ar/#!DetalleNormaBusquedaRapida/158269/20170125/resolucion%2051) of the Argentine Secretary of Commerce, on transparent prices, it is necessary that you comply with certain [additional requirements](/guides/localization/considerations-argentina.es.md).

To create the payment, it is important to send the data indicated above:

[[[
```php
<?php  

  require_once ('mercadopago.php');
  MercadoPago\SDK::configure(['ACCESS_TOKEN' => 'ENV_ACCESS_TOKEN']);

  $payment = new MercadoPago\Payment();

  $payment->transaction_amount = 100;
  $payment->token = "ff8080814c11e237014c1ff593b57b4d";
  $payment->description = "Title of what you are paying for";
  $payment->installments = 3;
  $payment->payment_method_id = "amex";
  $payment->issuer_id = 310;
  $payment->payer = array(
    "email" => "test_user_19653727@testuser.com"
  )

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
      .setInstallments(3)
      .setPaymentMethodId("amex")
      .setIssuerId(310)
      .setPayer(new Payer("test_user_19653727@testuser.com"));

payment.save();

```
```node

var mercadopago = require('mercadopago');
mercadopago.configurations.setAccessToken(config.access_token);

var payment = {
  transaction_amount: 100,
  token: 'ff8080814c11e237014c1ff593b57b4d'
  description: 'Title of what you are paying for',
  installments: 3,
  payment_method_id: 'amex',
  issuer_id: 310,
  payer: {
    email: 'test_user_3931694@testuser.com'
  }
};

mercadopago.payment.create(payment).then(function (data) {
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
payment.installments = 3
payment.payment_method_id = 'amex'
payment.issuer_id = 310
payment.payer = {
  email: "test_user_19653727@testuser.com"
}

payment.save()

```
]]]


## Response handling

It is **very important** to correctly report the results received when creating a payment. This will help improve conversion in cases of rejections, and avoid chargebacks in cases of approved transactions.

We recommend that you read the article [response handling](/guides/payments/api/handling-responses.es.md) and use the suggested communication in each case.

## Receive a payment notification

It is important to be aware of any updates on your payment status. For this, you must use Webhooks.

A Webhook is a notification that is sent from one server to another through an `HTTP POST` request.

You can find all the information about it in the [Webhooks](/guides/notifications/webhooks.es.md)section.

## Next steps

### Receive payments with stored cards

You can securely store your customers’ cards and make payments with a one-click-to-buy experience.

[More info.](/guides/payments/api/customers-and-cards.es.md)
