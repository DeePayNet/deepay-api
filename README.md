### DeePay.net API Document

The DeePay API provides a simple, powerful and secure way to accept Bitcoin, Litecoin, Ethereum and other cryptocurrencies as an online payment method. [Learn more](https://deepay.net)

### Getting Access
Merchant ID and API key is needed to make request to the system. [Sign up here](https://deepay.net) 

### Create Payment
|post|https://deepay.net/payment/create|
|----|---|

#### Parameters
|name|type|required|description|
|----|---|---|---|
|merchant_id|integer|Yes||
|out_trade_id|string|Yes|Merchant's custom unique order ID.|
|price_currency|string|Yes|The currency in which you price the order. Possible values: CNY, ETH, BTC, LTC.|
|price_amount|double|Yes|The price set by the merchant in price currency. Example: 1050.99.|
|receive_currency|string|Yes|The currency in which the customer will pay for the order.|
|notify_url|string|Yes|DeePay will send a notification to this url when order status is changed. Example: http://merchant-website.com/payments/accept-notify|
|callback_url|string|Yes|Redirect to Merchant URL after payment. Example: http://merchant-website.com/account/orders.|
|sign|string|Yes|See signature.|
|title|string|Yes|Product title. Example: Apple iPhone X|

#### Response

```json
{
    "payment_url": "http://deepay.net/payment/20180813160720569710",
    "price_currency": "CNY",
    "price_amount": "10",
    "transaction_id": "20180813160720569710",
    "receive_currency": "BTC",
    "receive_amount": "0.0003",
    "status": "pending",
    "created_at": 1534147640
}
```

### Signature
The merchant’s backend and DeePay create the same signature based on the same secret key and algorithm and use it to verify each other's identity.

#### General steps to create a signature:
1. Presume all data sent and received is the set M. Sort non-empty values in M in ascending alphabetical order (i.e. lexicographical sequence), and join them into string A via the corresponding URL key-value format (e.g. key1=value1&key2=value2...).
Notes:
    * Sort parameter names in ascending alphabetical order based on their ASCII encoded names (e.g. lexicographical sequence);
    * Empty parameter values are excluded in the signature;
    * Parameter names are case-sensitive;
    * When checking returned data or a DeePay push notification signature, the transferred sign parameter is excluded in this signature as it is compared with the created signature.

2.  Add "key= (API key value) to the end of stringA to get stringSignTemp, perform MD5 arithmetic on stringSignTemp, convert all result chars to upper case, thus get sign's value (signValue).

