---
layout: post
title: "CakePHP Stripe Payment Processing Component"
date: 2012-09-22 16:55
comments: true
categories: [cakephp, dev]
---

The component, packaged as a plugin, is [available at GitHub](https://github.com/chronon/CakePHP-StripeComponent-Plugin).

This is a simple component that interfaces a CakePHP app with Stripe's PHP API library. Pass the
component an array containing at least an amount and a Stripe token id, it will attempt the charge
and return an array of the fields you want. 

Compatibility:
--------------

Tested with CakePHP 2.2.x, though should work with any 2.x version. The required Stripe PHP API
library requires PHP 5 with cURL support.

Installation:
-------------

**Using [Composer](http://getcomposer.org/)/[Packagist](https://packagist.org):**

In your project `composer.json` file:

```
{
	"require": {
		"chronon/stripe": "*"
	},
	"config": {
        "vendor-dir": "Vendor"
    }
}
```

This will install the plugin into `Plugin/MobileDetect`, and install the Stripe library 
(from Packagist) into your `Vendor` directory.

In your app's `Config/bootstrap.php`, import composer's autoload file:

```php
<?php
App::import('Vendor', array('file' => 'autoload'));
```
**Using git:**

You will need the component (packaged as a plugin), and Stripe's PHP library (not included). The
Stripe library needs to be in this plugin's Vendor directory and must be named 'Stripe'. Using git, 
something like this:

```sh
git clone git@github.com:chronon/CakePHP-StripeComponent-Plugin.git APP/Plugin/Stripe  
git clone git://github.com/stripe/stripe-php.git APP/Plugin/Stripe/Vendor/Stripe
```

Configuration:
--------------

All configuration is in APP/Config/bootstrap.php.

**Required:** Load the plugin:

``` php
<?php	
CakePlugin::load('Stripe', array('bootstrap' => false, 'routes' => false));
```

**Required:** Set your Stripe secret API keys (both testing and live):

``` php
<?php	
Configure::write('Stripe.TestSecret', 'yourStripeTestingAPIKeyHere');
Configure::write('Stripe.LiveSecret', 'yourStripeLiveAPIKeyHere');
```

**Optional:** Set Stripe mode, either 'Live' or 'Test'. Defaults to Test if not set.

``` php
<?php
Configure::write('Stripe.mode', 'Test');
```

**Optional:** Set the currency. Defaults to 'usd'. Currently Stripe supports usd only.

``` php
<?php
Configure::write('Stripe.currency', 'usd');
```

**Optional:** fields for the component to return mapped to => Stripe charge object response fields. 
Defaults to `'stripe_id' => 'id'`. See the Stripe API docs for [Stripe\_Charge::create()](https://stripe.com/docs/api?lang=php#create_charge) for available fields. For example:
	
``` php
<?php
Configure::write('Stripe.fields', array(
	'stripe_id' => 'id',
	'stripe_last4' => array('card' => 'last4'),
	'stripe_address_zip_check' => array('card' => 'address_zip_check'),
	'stripe_cvc_check' => array('card' => 'cvc_check'),
	'stripe_amount' => 'amount'
));
```

See Usage below if `Stripe.fields` is confusing.

**Optional:** add a logging config:

``` php
<?php
CakeLog::config('stripe', array(
	'engine' => 'FileLog',
	'types' => array('info', 'error'),
	'scopes' => array('stripe'),
	'file' => 'stripe',
));
```

Usage:
------

Make a payment form however you want, see the [Stripe docs](https://stripe.com/docs/tutorials/forms)
for sample code. Add the component to your controller:

``` php
<?php
public $components = array(
	'Stripe.Stripe'
);
```

Format your form data so you can send the component an array containing at least:

``` php
<?php
$data = array(
	'amount' => '7.59',
	'stripeToken' => 'tok_0NAEASV7h0m7ny'
);
```

Optionally you can include a `description` field as well, which according to Stripe docs is:

> An arbitrary string which you can attach to a charge object. It is displayed when in the web 
> interface alongside the charge. It's often a good idea to use an email address as a description 
> for tracking later.

For example:

``` php
<?php
$data = array(
	'amount' => '7.59',
	'stripeToken' => 'tok_0NAEASV7h0m7ny',
	'description' => 'Casi Robot - casi@robot.com'
);
```

**Attempt a charge:** `$result = $this->Stripe->charge($data);`

If the charge was successful, `$result` will be an **array** as described by the configuration value 
of `Stripe.fields`. If `Stripe.fields` is not set:

``` php
<?php
$result = array(
	'stripe_id' => 'ch_0NXLLCydWzSIeE'
);
```

If `Stripe.fields` is set, using the example described above in the Configuration section would 
give you:

``` php
<?php
$result = array(
	'stripe_id' => 'ch_0NXLLCydWzSIeE',
	'stripe_last4' => '4242',
	'stripe_address_zip_check' => 'pass',
	'stripe_cvc_check' => 'pass',
	'stripe_amount' => 769
);
```

If the charge was not successful, `$result` will be a **string** containing an error message, and 
log the error.
