---
sites_supported:
    - mla
    - mlb
    - mlm
---


# Creando una subscripción

Subscribe a tus clientes para recibir pagos de forma periódica y automatizada.


## 1. Crea una preferencia de subscripción

Una preferencia de subscripción contiene todo el detalle del producto o servicio que se va a pagar de forma recurrente. Por ejemplo:

1. Datos y monto de lo que se va a pagar.
2. Frecuencia de la subscripción.
3. ID de referencia de tu sistema.

Para crear una preferencia de subscripción debes [instalar el SDK de Mercado Pago](/plugins) con tus [credenciales](https://www.mercadopago.com.ar/account/credentials?type=basic).

[[[
```php
<?php
  require_once ('mercadopago.php');
  MercadoPago\SDK::configure(['ACCESS_TOKEN' => 'ENV_ACCESS_TOKEN']);
?>
```
```java
import com.mercadopago.*;
MercadoPago.SDK.configure("ENV_ACCESS_TOKEN");
```
```node
var mercadopago = require('mercadopago');
mercadopago.configure({
    access_token: 'ENV_ACCESS_TOKEN'
});
```
```ruby
require 'mercadopago'
MercadoPago::SDK.configure(ACCESS_TOKEN: ENV_ACCESS_TOKEN)
```
]]]

Luego, deberás agregar los atributos de tu preferencia:

[[[
```php
<?php

  $preapproval = new MercadoPago\Preapproval();
  $preapproval->payer_email = "my_customer@my-site.com";
  $preapproval->back_url = "http://www.my-site.com";
  $preapproval->reason = "Monthly subscription to premium package";
  $preapproval->external_reference = "OP-1234";
  $preapproval->auto_recurring = array(
		"frequency" => 1,
		"frequency_type" => "months",
		"transaction_amount" => 60,
		"currency_id" => "ARS"
  );

  $preapproval->save();

?>
```
```java

AutoRecurring autoRecurring = new AutoRecurring();
autoRecurring.setFrequency(1);
autoRecurring.setFrequencyType("Months");
autoRecurring.setTransactionAmount(60);
autoRecurring.setCurrencyId("ARS");

Preapproval preapproval = new Preapproval();
preapproval.setPayerEmail("my_customer@my-site.com");
preapproval.setBackUrl("http://www.my-site.com");
preapproval.setReason("Monthly subscription to premium package");
preapproval.setExternalReference("OP-1234");
preapproval.setAutoRecurring(autoRecurring);
preapproval.save();

```
```node

var mercadopago = require('mercadopago');
mercadopago.configure({
    access_token: 'ENV_ACCESS_TOKEN'
});

preapproval_data = {
  payer_email: "my_customer@my-site.com",
  back_url: "http://www.my-site.com",
  reason: "Monthly subscription to premium package",
  external_reference: "OP-1234",
  auto_recurring: {
    frequency: 1,
    frequency_type: "months",
    transaction_amount: 60,
    currency_id: "ARS"
  }
}

mercadopago.preapproval.create(preapproval_data).then(function (data)) {
  // Do Stuff...
}).catch(function (error) {
  // Do Stuff...
});

```
```ruby

preapproval = MercadoPago::Preapproval.new()
preapproval.payer_email = "my_customer@my-site.com"
preapproval.back_url = "http://www.my-site.com"
preapproval.reason = "Monthly subscription to premium package"
preapproval.external_reference = "OP-1234"
preapproval.auto_recurring = {
  frequency: 1,
  frequency_type: "months",
  transaction_amount: 60,
  currency_id: "ARS"
}

preapproval.save()

```
]]]


> Estos son los datos mínimos e indispensables para crear una preferencia, pero tienes más opciones que puedes encontrar en [Añade características especiales a tu subscripción](#añade-características-especiales-a-tu-subscripción).


## 2. Lleva a tu comprador al checkout

Una vez creada la preferencia utiliza la URL que encontrarás en el atributo `init_point` de la respuesta para generar un botón de pago:

```html
<!DOCTYPE html>
<html>
	<head>
		<title>Pagar</title>
	</head>
	<body>
		<a href="<?php echo $preapproval['response']['init_point']; ?>">Subscribe!</a>
	</body>
</html>
```

## 3. Recibe información de los pagos de tus subscripciones

Recibirás notificaciones en forma automática para enterarte de tus nuevos pagos y las actualizaciones de sus estados.

Mercado Pago realizará su mayor esfuerzo para lograr que tus subscripciones resulten pagas, sin requerir acción alguna de tu parte.

En caso de no conseguir una aprobación de pago para la fecha de cobro estipulada, reintentaremos hasta cuatro veces más durante diez días, antes de que el pago quede marcado como impago. Frente a este estado puedes pausar o cancelar la subscripción.

Si la subscripción se encuentra activa se intentará cobrar en el próximo período.

Cada pago rechazado te será notificado mediante [Notificaciones](../../notifications/ipn.es.md). Analiza el motivo del rechazo, y comunícate con tu usuario para que, por ejemplo, actualice los datos de su tarjeta de crédito o la cambie por otra, antes de que se realice el próximo reintento de cobro.

Visita la sección [Notificaciones](/guides/notifications/ipn.es.md) para más información.


## 4. Prueba tu integración

Puedes probar tu integración antes de salir a producción, a fin de verificar el funcionamiento y realizar los ajustes que necesites.

Para ello debes usar usuarios y tarjetas de prueba.

Visita la sección [Probando](/guides/payments/api/testing.es.md) para más información.

## Añade características especiales a tu subscripción

Revisa el [API Doc de Preapproval](#) para conocer todas las configuraciones que puedes realizar. Así podrás adecuar el cobro de subscripción a tu modelo de negocio.

A continuación te mostramos las características más relevantes que puedes especificar al momento de crear una subscripción. Ten presente que son combinables entre sí para poder sacar el máximo provecho.

### Ofrece un período gratuito de prueba

Puedes ofrecer un periodo de prueba a tus clientes por una frecuencia determinada agregando la fecha de comienzo:

```json
{
  ...
  "auto_recurring": {
    ...
    "start_date" => "2016-12-10T14:58:11.778-03:00",
    ...
  },
  ...
}
```

### Limita la cantidad de cuotas de la subscripción

Puedes indicar que las subscripciones sólo durarán un período determinado de tiempo:

```json
{
  ...
  "auto_recurring": {
    ...
    "end_date" => "2018-12-10T14:58:11.778-03:00",
    ...
  },
  ...
}
```
