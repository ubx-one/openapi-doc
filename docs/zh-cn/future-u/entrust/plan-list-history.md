# 查询历史计划委托

`GET` /v1/future-u/trade/entrust/plan-list-history

## 请求参数

| 名称      | 位置  | 类型    | 必选 | 说明                                                         |
| --------- | ----- | ------- |----| ------------------------------------------------------------ |
| symbol    | query | string  | 否  | 交易对                                                       |
| direction | query | string  | 否  | 方向（PREV:上一页；NEXT:下一页）                             |
| id        | query | integer | 否  | none                                                         |
| limit     | query | integer | 否  | none                                                         |
| startTime | query | integer | 否  | 起始时间                                                     |
| endTime   | query | integer | 否  | 结束时间                                                     |
| state     | query | string  | 否  | 订单状态 NOT_ACTIVATION: 未激活；NOT_TRIGGERED：新建委托（未触发）；TRIGGERING：触发中；TRIGGERED：已触发；USER_REVOCATION：用户撤销；PLATFORM_REVOCATION：平台撤销（拒绝）；EXPIRED：已过期;DELEGATION_FAILED: 委托失败 |

> 请求示例

```shell
curl --location --request GET 'https://api.ubitex.com/v1/future-u/trade/entrust/plan-list-history?symbol=btc_usdt&id&direction=NEXT&limit&startTime&endTime&state' \
--header 'validate-algorithms: HmacSHA256' \
--header 'validate-appkey: 2fa91add-388c-44f2-8365-f4b72886c135' \
--header 'validate-recvwindow: 6000' \
--header 'validate-timestamp: 1725513706792' \
--header 'validate-signature: 9583e560d58bd48b0ebf6147b45e02f5a5124dddba2ef96c738d05613cdbc6c1' \
--header 'Accept: */*' \
--header 'Host: api.ubitex.com' \
--header 'Connection: keep-alive'
```

## 响应结果

```json
{
    "code": 0,
    "msg": "success",
    "msgInfo": null,
    "data": {
        "hasPrev": false,
        "hasNext": false,
        "items": [
            {
                "entrustId": "401862973001741632",
                "symbol": "btc_usdt",
                "entrustType": "TAKE_PROFIT",
                "orderSide": "BUY",
                "positionSide": "LONG",
                "timeInForce": "GTC",
                "closePosition": null,
                "price": "57115.65",
                "origQty": "1936",
                "stopPrice": "50000",
                "triggerPriceType": "LATEST_PRICE",
                "executedQty": "0",
                "avgPrice": null,
                "state": "USER_REVOCATION",
                "marketOrderLevel": null,
                "createdTime": 1725513599041,
                "updatedTime": 1725513702188,
                "ordinary": false
            }
        ]
    },
    "ts": 1725513706974
}
```
