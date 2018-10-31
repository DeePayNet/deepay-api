# DeePay API Document

The DeePay API provides a simple, powerful and secure way to accept Bitcoin, Litecoin, Ethereum and other cryptocurrencies as an online payment method. [Learn more](https://deepay.net)

### [中文版](./README-CN.md)

## Overview
1. Call Create Order API method to create an order in DeePay system.

2. DeePay checks if the request is valid.

	*  If the requst is valid, DeePay returns order data with 200 HTTP status. After receiving 200 HTTP status, you should redirect the customer to payment_url address.

	* If the request is invalid, DeePay returns 422 (or another) error HTTP status and an error message.

3. When the customer pays successfully, DeePay sends Payment Notification to your notify_url, which is defined when creating the order. DeePay also sends Payment Notification when order status is changed to expired or to any other status.




## Get Access
Merchant ID and API key is needed to make request to the system. [Sign up here](https://deepay.net) 

## Create Order
Create an order at DeePay and redirect customer to payment page.

```
POST https://deepay.net/order/create
```

#### Parameters
|Field|Type|Required|Description|
|----|---|---|---|
|merchant_id|integer|Yes|Merchant ID assigned by DeePay.|
|out_trade_id|string|Yes|Merchant's custom unique order ID.|
|price_currency|string|Yes|The currency in which you price the order. Possible values: CNY, USD, ETH, BTC, LTC.|
|price_amount|double|Yes|The price set by the merchant in price currency. Example: 1050.99|
|notify_url|string|Yes|DeePay will send a Payment Notification to this url when order status is changed. Example: http://merchant-website.com/payments/accept-notify|
|callback_url|string|Yes|Redirect to Merchant URL after payment. Example: http://merchant-website.com/account/orders.|
|title|string|Yes|Product title. Example: Apple iPhone X|
|attach|string|No|An additional field to be returned in the Payment Notification after submitting an order or in the Query API.|
|sign|string|Yes|See <a href="#signature">Signature</a>.|

#### Response

```json
{
    "payment_url": "https://deepay.net/order/20180813160720569710",
    "price_currency": "CNY",
    "price_amount": "10",
    "transaction_id": "20180813160720569710",
    "status": "new",
    "created_at": 1534147640
}
```

## Checkout
Select pay currency and fill in E-mail address. Display payment_address and pay_amount for customer or redirect to payment_url. 

```
POST https://deepay.net/order/checkout
```

#### Parameters
|Field|Type|Required|Description|
|----|---|---|---|
|merchant_id|integer|Yes|Merchant ID assigned by DeePay.|
|transaction_id|string|No|Unique order ID assigned by DeePay. Required when out_trade_id is absent.|
|out_trade_id|string|No|Merchant's custom unique order ID. Required when transaction_id is absent.|
|pay_currency|string|Yes|The currency which customer selects to pay. Possible values: ETH, BTC, LTC, BCH. You can setup this configuration in dashboard.|
|email|string|No|Customer E-mail address. An notification E-mail will be sent to this address when the payment succeeds.|
|sign|string|Yes|See signature.|


#### Response

```json
{
    "payment_url": "https://deepay.net/order/20181006170328484998",
    "price_currency": "USD",
    "price_amount": "10.0001",
    "transaction_id": "20181006170328484998",
    "pay_currency": "BTC",
    "pay_amount": "0.0016",
    "status": "pending",
    "created_at": 1538816608,
    "expire_at": 1538817815,
    "payment_address": "2Mvp6TgEzWCWT2K1hjRWpy5hq8BHnnCjY6X",
    "attach": "abc"
}
```

## Query Order
Retrieve information of a specific order. 

```
POST https://deepay.net/order/query
```

#### Parameters
|Field|Type|Required|Description|
|----|---|---|---|
|merchant_id|integer|Yes|Merchant ID assigned by DeePay.|
|transaction_id|string|No|Unique order ID assigned by DeePay. Required when out_trade_id is absent.|
|out_trade_id|string|No|Merchant's custom unique order ID. Required when transaction_id is absent.|
|sign|string|Yes|See signature.|


#### Response

```json
{
    "merchant_id": "10001",
    "out_trade_id": "E201809123",
    "pay_amount": "0.0016",
    "pay_currency": "BTC",
    "price_amount": "10.0001",
    "price_currency": "USD",
    "attach": "abc",
    "created_at": 1538796660,
    "expire_at": 1538797878,
    "transaction_id": "20181006113100525110",
    "status": "expired"
}
```

## Exchange Rate

Current exchange rate for any two currencies, fiat or crypto. This endpoint is public, authentication is not required.


```
GET https://deepay.net/rate/:from/:to
```

#### Parameters
|Field|Type|Required|Description|
|----|---|---|---|
|from|string|Yes|Currency Symbol. Example：USD, CNY, BTC, ETH, LTC|
|to|string|Yes|Currency Symbol. Example：USD, CNY, BTC, ETH, LTC|



#### Response

```json
7762.8
```


## Payment Notification
Notification will be sent to merchant's notify_url when order status is changed.

```
POST <notify_url>
```


#### Parameters
|Field|Type|Required|Description|
|----|---|---|---|
|merchant_id|integer|Yes|Merchant ID assigned by DeePay.|
|transaction_id|string|Yes|Unique order ID assigned by DeePay.|
|out_trade_id|string|Yes|Merchant's custom unique order ID.|
|price_amount|double|Yes|The price set by the merchant in price currency.|
|pay_amount|double|Yes|The amount of cryptocurrency (defined by pay_currency) paid by the customer.|
|pay_currency|string|Yes|The cryptocurrency in which the order was made.|
|attach|string|No|Additional information customized by merchant.|
|sign|string|Yes|See signature.|



## Order Status

|Status|Description|
|---|---|
|new|Order is created just now.|
|pending| Customer selected pay currency. Awaiting payment.|
|paid|Customer transferred cryptocurrency to payemnt address. Awaiting blockchain network confirmation.|
|confirmed|Payment is confirmed by the network with at least 1 confirmation. Purchased goods/services can be securely delivered to the customer.|
|complete|Payment is confirmed by the network with at least 6 confirmation and has been credited to the merchant.|
|invalid|Payment rejected by the network or did not confirm within 1 day.|
|expired|Customer did not pay within the required time (default: 20 minutes) and the order expired.|


## <a name="signature">Signature</a>
The merchant’s backend and DeePay create the same signature based on the same secret key and algorithm and use it to verify each other's identity.

#### General steps to create a signature:
1. Presume all data sent and received is the set M. Sort non-empty values in M in ascending alphabetical order (i.e. lexicographical sequence), and join them into string A via the corresponding URL key-value format (e.g. key1=value1&key2=value2...).
Notes:
    * Sort parameter names in ascending alphabetical order based on their ASCII encoded names (e.g. lexicographical sequence);
    * Empty parameter values are excluded in the signature;
    * Parameter names are case-sensitive;
    * When checking returned data or a DeePay push notification signature, the transferred sign parameter is excluded in this signature as it is compared with the created signature.

2.  Add "key= (API key value) to the end of stringA to get stringSignTemp, perform MD5 arithmetic on stringSignTemp, convert all result chars to upper case, thus get sign's value (signValue).

#### Example:  
For the following transferred parameters:

```
merchant_id: 10001 
out_trade_id: MDC001
title: iphone X
```

(1) Sort ASCII code of parameter names by lexicographical sequence based on the format of "key=value"

```
stringA="merchat_id=10001&out_trade_id=MDC001&title=iphone X";
```

(2) Join API keys

```
stringSignTemp="stringA&key=192006250b4c09247ec02edce69f6a2d" 
sign=MD5(stringSignTemp).toUpperCase()="68739BCFD4344894FA9BDDDEE915992C"
```


