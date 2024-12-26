# 创建跟踪委托

`POST` /trade/v2/entrust/create-track

## 请求参数

| 名称               | 位置 | 类型         | 必选  | 说明                                                         |
| ------------------ | ---- | ------------ | ----- | ------------------------------------------------------------ |
| symbol             | body | string  | true  | 交易对                                                       |
| origQty            | body | number  | true  | 数量（张）                                                   |
| positionType       | body | string  | true  | 仓位模式：CROSSED 全仓、ISOLATED 逐仓                        |
| orderSide          | body | string  | true  | 订单方向：BUY;SELL                                           |
| positionSide       | body | string  | true  | 持仓方向：LONG;SHORT                                         |
| triggerPriceType   | body | string  | true  | 触发价格类型：INDEX_PRICE(指数价格); MARK_PRICE(标记价格)；LATEST_PRICE(最新价格) |
| callback           | body | string  | true  | 回调幅度配置 1 比例 2 固定                                   |
| callbackVal        | body | number  | true  | 回调幅度配置值,大于0                                         |
| activationPrice    | body | number  | false | 激活价格,非必填                                              |
| expireTime         | body | integer | false | 过期时间                                                     |
| clientOrderId      | body | string  | false | 客户端ID                                                     |
| clientMedia        | body | string  | false | 客户端媒体                                                   |
| clientMediaChannel | body | string  | false | 客户端媒体渠道                                               |

> 请求示例

```shell

```

## 响应结果

```json

```
