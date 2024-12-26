Get Futures Info

`GET` /v1/future-u/market/public/cg/contracts

## Request Parameters

Note：This method does not require a signature.

> Request Example

```shell
curl --location --request GET 'https://api.ubitex.com/v1/future-u/market/public/cg/contracts' \
--header 'Accept: */*' \
--header 'Host: api.ubitex.com' \
--header 'Connection: keep-alive'
```

## Response Result

```json
[
  {
    "id": 123,
    "ask": "1817.32",
    //Current lowest ask price
    "base_currency": "ETH",
    //Symbol/currency code of base pair, eg. BTC
    "base_volume": "13267684284",
    //24 hour trading volume
    "bid": "1817.31",
    //Current highest bid price
    "contractSize": 10,
    //Futures par value
    "end_timestamp": 253402099200000,
    //Ending of this derivative product
    "funding_rate": "-0.03",
    //Fund rate
    "high": "1828.89",
    //24-hour highest trading price
    "index_currency": "USD",
    //Underlying currency for index
    "index_name": "ETH-USD",
    //Name of the underlying index if any
    "index_price": "1816.61",
    //Underlying index price
    "last_price": "1817.31",
    //Latest price
    "low": "1778.65",
    //24-hour lowest trading price
    "next_funding_rate": "-0.03",
    //Upcoming predicted funding rate
    "next_funding_rate_timestamp": 1698681600000,
    //Next funding rate time
    "open_interest": "2419347630",
    //The open interest in the last 24 hours in contracts
    "product_type": "PERPETUAL",
    //Product type
    "start_timestamp": 1651328033000,
    //Starting of this derivative product (relevant for expirable futures or options)
    "symbol": "eth_usd",
    "target_currency": "USD",
    //Symbol/currency code of target pair, eg. ETH
    "target_volume": "73698647.51054371",
    //24 hours trading volume
    "ticker_id": "ETH-USD",
    //Identifier of a ticker with delimiter to separate base/target, eg. BTC-PERP
    "underlyingType": 1
    //Target type, Coin-M,USDT-M
  }
]
```

