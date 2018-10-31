### DeePay API接口文档

DeePay API提供了一种简单易用、功能强大和安全的在线支付方法，来接受比特币、莱特币、以太坊和其他加密货币。[了解更多](https://deepay.net)

### [English Version](./README.md)


## 概览
1. 调用创建订单接口。
2. DeePay会验证请求是否合法。
	* 如果请求合法，DeePay响应`HTTP 200`状态码和返回订单数据。在接收到正确的响应后，您应该把用户重定向到支付页(payment_url)。
	* 如果请求不合法，DeePay会响应`HTTP 422`或其他错误状态码和返回错误信息。
3. 如果用户成功支付，DeePay会发送通知到`notify_url`。在订单状态改变时，DeePay同样会发出通知。


## 获取授权
你需要获取商户ID和API密钥以访问DeePay API.[马上注册](https://deepay.net) 

## 创建订单
创建订单纪录并重写向用户到支付页。

```
POST https://deepay.net/order/create
```

#### 请求参数
|字段名|类型|必填|描述|
|----|---|---|---|
|merchant_id|integer|是|DeePay分配的商户号|
|out_trade_id|string|是|商户系统内部订单号，在同一个商户号下唯一|
|price_currency|string|是|计价货币，即商品标价所用货币。可选值: CNY, USD, ETH, BTC, LTC|
|price_amount|double|是|计价金额，即商品金额。例如: 1050.99|
|notify_url|string|是|异步通知地址。当订单状态发生改变时，DeePay会发送通知到这个地址。例如: http://merchant-website.com/payments/accept-notify|
|callback_url|string|是|支付完成后，返回商户的跳转地址。例如: http://merchant-website.com/account/orders.|
|title|string|是|商品名称. 例如: Apple iPhone X|
|attach|string|是|附加数据，在查询API和支付通知中原样返回，可作为自定义参数使用。|
|sign|string|是|详见<a href="#签名算法">签名算法</a>|

#### 响应示例

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

## 检出订单
选择支付货币和填写用户邮箱地址。您可以向用户展示支付地址和金额，或者重定向到支付页面。

```
POST https://deepay.net/order/checkout
```

#### 请求参数
|字段名|类型|必填|描述|
|----|---|---|---|
|merchant_id|integer|是|DeePay分配的商户号|
|transaction_id|string|否|DeePay分配的支付编号。与out_trade_id二选一，建议优先使用|
|out_trade_id|string|是|商户系统内部订单号，在同一个商户号下唯一. 与transaction_id二选一|
|pay_currency|string|是|支付货币。可选值: ETH, BTC, LTC, BCH。您可以在控制台设置此字段的可选值。|
|email|string|否|用户邮箱地址。支付成功后，DeePay会向此邮箱发送通知。|
|sign|string|是|详见<a href="#签名算法">签名算法</a>|


#### 响应示例

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

## 查询订单
获取特定订单的信息。

```
POST https://deepay.net/order/query
```

#### 请求参数
|字段名|类型|必填|描述|
|----|---|---|---|
|merchant_id|integer|是|DeePay分配的商户号|
|transaction_id|string|否|DeePay分配的支付编号。与out_trade_id二选一，建议优先使用|
|out_trade_id|string|否|商户系统内部订单号，在同一个商户号下唯一. 与transaction_id二选一|
|sign|string|是|详见<a href="#签名算法">签名算法</a>|


#### 响应示例

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

## 换算汇率

任何两种货币的现行汇率，法币或者加密货币。此接口是公共的，不需要身份验证。


```
GET https://deepay.net/rate/:from/:to
```

#### 请求参数
|字段名|类型|必填|描述|
|----|---|---|---|
|from|string|是|货币符号。例如：USD, CNY, BTC, ETH, LTC, BCH|
|to|string|是|货币符号。例如：USD, CNY, BTC, ETH, LTC, BCH|



#### 响应示例

```json
7762.8
```


## 支付通知
当订单状态改变时，DeePay会向`notify_url`发送支付通知。

```
POST <notify_url>
```


#### 请求参数
|字段名|类型|必须|描述|
|----|---|---|---|
|merchant_id|integer|是|DeePay分配的商户号|
|out_trade_id|string|是|商户系统内部订单号，在同一个商户号下唯一|
|price_currency|string|是|计价货币，即商品标价所用货币。可选值: CNY, USD, ETH, BTC, LTC|
|price_amount|double|是|计价金额，即商品金额。例如: 1050.99|
|pay_amount|double|是|支付金额（以`pay_currency`计价）|
|pay_currency|string|是|支付货币。可选值: ETH, BTC, LTC, BCH。您可以在控制台设置此字段的可选值。|
|attach|string|是|附加数据，在查询API和支付通知中原样返回，可作为自定义参数使用。|
|sign|string|是|详见<a href="#签名算法">签名算法</a>|




## 订单状态

|状态|描述|
|---|---|
|new|新创建订单|
|pending| 用户已选择支付货币，等待支付。|
|paid|用户已转移加密货币到支付地址，等待区块链确认。|
|confirmed|支付已有至少1个区块确认。交易的货物/服务可以安全地交付给客户。|
|complete|支付已有至少6个区块确认。交易款项已划入商户账户。|
|invalid|支付被区块链网络拒绝，或者在1天内未得到至少1个确认。|
|expired|用户在支付期限内（默认20分钟）未支付，订单过期。|


## <a name="签名算法">签名算法</a>

商家的后端程序和DeePay基于相同的密钥和算法创建相同的签名，以验证彼此的身份。


#### 签名生成的通用步骤如下：

1. 设所有发送或者接收到的数据为集合M，将集合M内非空参数值的参数按照参数名ASCII码从小到大排序（字典序），使用URL键值对的格式（即key1=value1&key2=value2…）拼接成字符串stringA。

	特别注意以下重要规则：

	* 参数名ASCII码从小到大排序（字典序）；
	* 如果参数的值为空不参与签名；
	* 参数名区分大小写；
	* 验证调用返回或DeePay主动通知签名时，传送的sign参数不参与签名，将生成的签名与该sign值作校验。

2. 在stringA最后拼接上key得到stringSignTemp字符串，并对stringSignTemp进行MD5运算，再将得到的字符串所有字符转换为大写，得到sign值signValue。


#### 举例：

假设传送的参数如下：


```
merchant_id: 10001 
out_trade_id: MDC001
title: iphone X
```

第一步：对参数按照key=value的格式，并按照参数名ASCII字典序排序如下：

```
stringA="merchat_id=10001&out_trade_id=MDC001&title=iphone X";
```

第二步：拼接API密钥：

```
stringSignTemp="stringA&key=192006250b4c09247ec02edce69f6a2d" 
sign=MD5(stringSignTemp).toUpperCase()="68739BCFD4344894FA9BDDDEE915992C"
```
