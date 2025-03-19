![](https://ramoona.github.io/banks-db-demo//assets/logo.svg)

[![Build Status](https://img.shields.io/travis/ramoona/banks-db/master.svg?style=flat-square)](https://travis-ci.org/ramoona/banks-db)
[![Latest Stable Version](https://img.shields.io/npm/v/banks-db.svg?style=flat-square)](https://www.npmjs.com/package/banks-db)
[![NPM Downloads](https://img.shields.io/npm/dm/banks-db.svg?style=flat-square)](https://www.npmjs.com/package/banks-db)

Returns bank's name and brand color by bankcard prefix (BIN).

It is useful on billing pages to use bank’s brand color when user starts to type card number.
   
![](https://ramoona.github.io/banks-db-demo/assets/usage.gif)

It's a community driven database, so it can potentially contains mistakes. It's not a problem for UX enhancement,
but you must not use Banks DB in your billing logic.

## The Sequence of the Digits in Credit Card Numbers

The card digits and the card number order are strategically chosen and placed. They reveal  **crucial information about the card, the cardholder, and the card issuer**. Discover what they mean in the table below:

|Number|What It Is|What It Means|
|--|--|--|
|The first digit|Major Industry Identifier (MII)|Signifies the credit card network and industry|
Digits 1–8|The Issuer Identification Number (IIN) or the Bank Identification Number (BIN)|Indicates the financial institution that issued the card|
|Digits 7–15|Account Number|Identifies the card and account holder|
|The last digit|Checksum/Check Digit|Verifies the legitimacy of the other digits using the Luhn algorithm|

Note that not all credit card numbers follow this exact structure. For example, the checksum on a Visa might be digit 13 instead of the last digit.

The Bank Identification Number (BIN), also known as the Issuer Identification Number, is the  **first 4–8-digit sequence at the beginning of the card number.**  As you can see in the table above, the BIN contains the industry identifier and indicates the financial institution that issued the card.

The BIN system was established by the American National Standards Institute (ANSI) and the International Organization for Standardization (ISO). Its primary purpose is to increase the efficiency and security of payments.

The Bank Identification Number helps merchants evaluate transactions and detect fraudulent or stolen cards. When initiating a transaction, you need to enter your card number, expiration date, and CVV or CVC. The issuer then receives a request to verify the account, the availability of the funds, and the compliance with national law. The BIN gives merchants all they need to validate the information and process the transaction.

## Demo
Try your card prefix in our [demo](https://ramoona.github.io/banks-db-demo/). Note that only first 6 digits of card number are required.

## Usage

### PostCSS

With [postcss-banks-db](https://github.com/ramoona/postcss-banks-db) and
[postcss-contrast](https://github.com/stephenway/postcss-contrast) you can
generate CSS for each bank:

```css
.billing-form {
    transition: background .6s, color .6s;
    background: #eee;
}
@banks-db-template {
    .billing-form.is-%code% {
        background-color: %color%;
        color: contrast(%color%);
    }
}
```

And then switch bank’s style in JS:

```js
import banksDB from 'banks-db';

const bank = banksDB(cardNumberField.value);
if ( bank.code ) {
  billingForm.className = 'billing-form is-' + (bank.code || 'other');
  bankName.innerText = bank.country === 'ru' ? bank.localTitle : bank.engTitle;
} else {
  billingForm.className = 'billing-form';
  bankName.innerText = '';
}
```

### CSS-in-JS

```js
import contrast from 'contrast';
import banksDB  from 'banks-db';

BillingForm  = ({ cardNumber }) => {
  const title, color;
  const bank = banksDB(this.props.cardNumber);
  if ( bank.code ) {
    title = bank.country === 'ru' ? bank.localTitle : bank.engTitle;
    color = bank.color;
  } else {
    title = '';
    color = '#eee';
  }
  return (
    <div style={{
      transition: 'background .6s, color .6s',
      background: color,
      color:      contrast(color) === 'light' ? 'black' : 'white'
    }}>
      <h2>{ title }</h2>
      …
    </div>
  );
}
```

### Other Best Practices
See also best practices for billing forms:

* [fast-luhn](https://github.com/bendrucker/fast-luhn) to check bank card number for mistakes
* [Halter font](http://www.dafont.com/halter.font) to simulate bank card font
* [convert-layout](https://github.com/ai/convert-layout) to force English keyboard in holder name field

## API

There are two options to use BanksDB depends of whether you need specific countries or not.

#### If you need banks for all countries
Library exports `banksDB` function. It accepts bankcard number and returns
bank object.

```js
var banksDB = require('banks-db');
var bank    = banksDB('5275 9400 0000 0000');
console.log(bank.code) //=> 'ru-citibank'
console.log(bank.type) //=> 'mastercard'
```

If database doesn't contain some bank prefix, bank object will have only
`type` field.

```js
var unknown = banksDB('4111 1111 1111 1111');
console.log(bank.code) //=> undefined
console.log(bank.type) //=> 'visa'
```

You can also get banks database by `banksDB.data`:

```js
for ( let bank of banksDB.data ) {
    console.log(bank);
}
```

#### If you need only banks for specific countries

Instead of `banks-db` use `banks-db/core`:
```js
var banksDBCore = require('banks-db/core');
```
Then require desired countries from `banks-db/banks` by two letters code:
```js
var banksOfRussia = require('banks-db/banks/ru');
var banksOfChina = require('banks-db/banks/cn');
```
All that left is to call `banksDBCore` with your countries data to initialize. `banksDBCore` is a function that accepts one argument with banks data for countries that you've specified, and returns an instance of BanksDB object with `findBank` method and `data` property.
```js
var BanksDB = banksDBCore([banksOfRussia, banksOfChina]);
// var BanksDB = banksDBCore(banksOfRussia); no need for an array if there's only one country
```
That's it! Ready to use:
```js
var bank = BanksDB.findBank('5275 9400 0000 0000');
var data = BanksDB.data;
```

### Bank Object

* `type`: bankcard type. For example, `'visa'` or `'mastercard'`.
  Banks DB will return it even if bank is unknown.
* `code`: unique code, contains country and name. For example, you can use it to generate CSS selectors for each bank.
* `color`: bank's brand color in HEX-format.
* `localTitle`: bank's title in local language.
* `engTitle`: international bank's title.
* `name`: short bank's name (not unique). For example, `'citibank'`.
* `country`: bank's operation country. For example, you can use it
  to display `localTitle` for local banks and `engTitle` for others.
* `url`: bank's website URL.

## Contributing

In case your bankcard doesn't work, please check if your bank already in [Banks DB](https://github.com/Ramoona/banks-db/tree/master/banks):

- If your bank is already included, you can [open an issue](https://github.com/Ramoona/banks-db/issues) with your prefix (NOT full number of your card, just first 5 or 6 symbols) or send a pull request.
- Otherwise you can add a new bank (see [contributing guide](https://github.com/Ramoona/banks-db/blob/master/CONTRIBUTING.md)).

## Changelog
See [CHANGELOG.md](https://github.com/ramoona/banks-db/blob/master/CHANGELOG.md) or [release notes](https://github.com/ramoona/banks-db/releases) (with commits history).
