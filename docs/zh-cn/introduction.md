# REST API

> `域名`:  `https://api.ubitex.com`

----



# 接口基本信息

鉴于延迟高和稳定性差等原因，不建议通过代理的方式访问API。

GET请求参数放入query Params中，POST请求参数放入request body中

请求头信息请设置为：`Content-Type=application/json`

对于/public以外开头的请求，需要对请求报文进行签名

----

# 限频规则

部分接口会有限流控制(对应接口下会有限流说明)，限流主要分为网关限流和WAF限流。

若接口请求触发了网关限流则会返回`429`，表示警告访问频次超限，即将被封IP或者apiKey。

网关限流分为针对IP和apiKey限流。

IP限流示例说明：`100/s/ip`，表示每个IP每秒该接口请求次数限制。

apiKey限流示例说明：`50/s/apiKey`，表示每个apiKey每秒该接口请求次数限制。

----

# 签名说明

由于UbitEx需要为第三方平台提供一些开放性的接口，所以需要接口的数据安全问题，比如数据是否被篡改，数据是否已过时，数据是否可以重复提交，接口在某个时间内访问频率等问题。其中数据是否被篡改是最重要的。

1. 先通过用户中心申请appkey和secretkey，针对不同的调用，提供不同的appkey和secretkey。

2. 加入`timestamp`(时间戳)，其值应当是请求发送时刻的unix时间戳(毫秒)，数据的有郊时间根据此值来计算。

3. 加入`signature`(数据签名)，所有数据的签名信息。

4. 加入`recvwindow`(自定义请求有效时间)，有效时间目前相对简单统一固定为某个值。

   > 服务器收到请求时会判断请求中的时间戳，最长60秒，最小为2秒，如果是5000毫秒之前发出的，则请求会被认为无效。  
   这个时间窗口值可以通过发送可选参数recvWindow来设置。 另外，如果服务器计算得出客户端时间戳在服务器时间的‘未来’一秒以上，也会拒绝请求。
   关于交易时效性
   互联网状况并不100%可靠，不可完全依赖,因此你的程序本地到UBIT服务器的时延会有抖动。这是我们设置recvwindow的目的所在，如果你从事高频交易，对交易时效性有较高的要求，可以灵活设置recvwindow以达到你的要求。
   不推荐使用5秒以上的recvwindow

5. 加入algorithms (签名方法/算法)，用户计算签名是基于哈希的协议，推荐使用HmacSHA256。具体支持那些协议，请参见下面列出:
   `HmacMD5、HmacSHA1、HmacSHA224、HmacSHA256(推荐)、HmacSHA384、HmacSHA512`

## 签名生成

以 https://api.ubitex.com/v1/spot 为例  
以下是在linux bash环境下使用 echo openssl 和curl工具实现的一个调用接口下单的示例

appkey、secret仅供示范:

- `appKey`: uasdfk-76d0-4f6e-a6b2-asdfdas
- `secretKey`: bc6630d0231fda5cd98794f52c4998659beda290

Header部分数据：

- `validate-algorithms`: HmacSHA256
- `validate-appkey`: uasdfk-76d0-4f6e-a6b2-asdfdas
- `validate-recvwindow`: 5000
- `validate-timestamp`: 1717234493000
- `validate-signature`: 1231312318f13dc27dbbd02c2cc51ff7059765ed12313131

## 请求数据

```json
{"symbol":"BTC_USDT","clientOrderId":"16559590087220001","side":"BUY","type":"LIMIT","timeInForce":"FOK","bizType":"SPOT","price":40000,"quantity":2}
```

### 1. 数据部分

- `method`: 大写的请求方法，例如：GET、POST、DELETE、PUT

- `path`: 按照path中顺序将所有value进行拼接。形如/test/{var1}/{var2}/的restful路径将按填入的实际参数后路径拼接，示例：/sign/test/bb/aa

- `query`: 按照key的字典序排序，将所有key=value进行拼接。示例：userName=dfdfdf&password=ggg

- `body`: 直接按JSON字符串不做转换或排序操作。

- `x-www-form-urlencoded`: 按照key的字典序排序，将所有key=value进行拼接，示例:userName=dfdfdf&password=gggform-data：此格式暂不支持。

如果存在多种数据形式，则按照path、query、body的顺序进行再拼接，得到所有数据的拼接值。

方法method示例： POST

路径path示例: /v1/spot/order

上述拼接值记作为 path

参数通过query示例: symbol=btc_usdt

上述值拼接记作 query

参数通过body示例：  
`x-www-form-urlencoded`:

```
symbol=btc_usdt&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=69000
```
    
上述值拼接记作body

`json`:

```json
{"symbol":"BTC_USDT","clientOrderId":"16559590087220001","side":"BUY","type":"LIMIT","timeInForce":"FOK","bizType":"SPOT","price":40000,"quantity":2,"media":"btok","mediaChannel":"12345"}
```

上述值拼接记作body

- 混合使用query与body(分为表单与json两种格式)

`query`:

```
symbol=btc_usdt&side=BUY&type=LIMIT
```

上述拼接值记作query

`body`:

```json
{"symbol":"BTC_USDT","clientOrderId":"16559590087220001","side":"BUY","type":"LIMIT","timeInForce":"FOK","bizType":"SPOT","price":40000,"quantity":2,"media":"btok","mediaChannel":"12345"}
```

上述拼接值记作body

整个数据最且拼接值由##符号分别与method、path、query、body进行拼接成##method、##path、##query、##body，
最终拼接值记作为Y=##method##path##query##body。 注意:

query有数据，body无数据：Y=##method##path##query

query无数据，body有数据：Y=##method##path##body

query有数据，body有数据：Y=##method##path##query##body

### 2. 请求头部分

将key按照字母自然升序后，使用&方式拼接在一起，作为X。如：

```
validate-algorithms=HmacSHA256&validate-appkey=2fa91add-388c-44f2-8365-f4b72886c135&validate-recvwindow=6000&validate-timestamp=1725455266041
```

### 3. 生成签名

最终把需要进行加密的字符串，记作为original=XY

最后将最终拼接值按照如下方法进行加密得到签名。

```java
String signature=org.apache.commons.codec.digest.HmacUtils.hmacSha256Hex(secretkey, original);
```

将生成的签名singature放到请求头中，以validate-signature为Key，以singature为值。

### 4. 样例

- 签名原始报文样例：  

```
validate-algorithms=HmacSHA256&validate-appkey=2fa91add-388c-44f2-8365-f4b72886c135&validate-recvwindow=6000&validate-timestamp=1725455266041#POST#/v1/spot/order#{"symbol":"BTC_USDT","clientOrderId":"16559590087220001","side":"BUY","type":"LIMIT","timeInForce":"FOK","bizType":"SPOT","price":40000,"quantity":2,"media":"btok","mediaChannel":"12345"}
```

- 请求报文样例：

```shell
curl --location --request POST 'https://api.ubitex.com/v1/spot/order' \
--header 'Content-Type: application/json' \
--header 'validate-algorithms: HmacSHA256' \
--header 'validate-appkey: 2fa91add-388c-44f2-8365-f4b72886c135' \
--header 'validate-recvwindow: 6000' \
--header 'validate-timestamp: 1725455266041' \
--header 'validate-signature: ce246607785e168d4677afff5af3746eb8513133d11ca3c5e3913eeea5aca63c' \
--header 'Accept: */*' \
--header 'Host: api.ubitex.com' \
--header 'Connection: keep-alive' \
--data-raw '{"symbol":"BTC_USDT","clientOrderId":"16559590087220001","side":"BUY","type":"LIMIT","timeInForce":"FOK","bizType":"SPOT","price":40000,"quantity":2,"media":"btok","mediaChannel":"12345"}'
```
- 注意事项：

  注意检查 Content-Type、签名原始报文中的参数格式、请求报文中的参数格式  
  Java sdk: http://git.ubit.site/backend/sdk-for-java.git

# 响应格式

所有的接口返回都是JSON格式。

```json
{
  "code": 0,
  "data": {},
  "msg": "SUCCESS",
  "msgInfo": []
}
```
----

# 响应代码

## HTTP Status

| httpStatus | 描述                    |
|:-----------|:----------------------|
| 200        | 请求成功，请进一步查看rc、mc部分    |
| 404        | 接口不存在                 |
| 429        | 请求过于频繁，请按照限速要求，控制请求速率 |
| 500        | 服务异常                  |
| 502        | 网关异常                  |
| 503        | 服务不可用，请稍后重试           |

## Result Code

| code | return Code |
|------|-------------|
| 0    | 业务成功        |
| 1    | 业务失败        |

## Message Code

| msg          | message code                                |
|--------------|---------------------------------------------|
| SUCCESS      | 成功                                          |
| FAILURE      | 失败                                          |
| AUTH_001     | 缺少请求头 validate-appkey                       |
| AUTH_002     | 缺少请求头 validate-timestamp                    |
| AUTH_003     | 缺少请求头 validate-recvwindow                   |
| AUTH_004     | 错误的请求头 validate-recvwindow                  |
| AUTH_005     | 缺少请求头 validate-algorithms                   |
| AUTH_006     | 错误的请求头 validate-algorithms                  |
| AUTH_007     | 缺少请求头 validate-signature                    |
| AUTH_101     | ApiKey不存在容                                  |
| AUTH_102     | ApiKey未激活容                                  |
| AUTH_103     | 签名错误                                        |
| AUTH_104     | 非绑定IP请求                                     |
| AUTH_105     | 报文过时                                        |
| AUTH_106     | 超出apikey权限                                  |
| SYMBOL_001   | 交易对不存在                                      |
| SYMBOL_002   | 交易对未开盘                                      |
| SYMBOL_003   | 交易对暂停交易                                     |
| SYMBOL_004   | 此交易对不支持您所在的国家                               |
| SYMBOL_005   | 该市场不支持通过API进行交易                             |
| ORDER_001    | 平台拒单                                        |
| ORDER_002    | 资金不足                                        |
| ORDER_003    | 交易对暂停交易                                     |
| ORDER_004    | 禁止交易                                        |
| ORDER_005    | 订单不存在                                       |
| ORDER_006    | 过多的未完成订单                                    |
| ORDER_007    | 子账户暂无交易权限                                   |
| ORDER_008    | 当前下单价格或数量精度异常                               |
| ORDER_F0101  | 触发价格过滤器-最小值                                 |
| ORDER_F0102  | 触发价格过滤器-最大值                                 |
| ORDER_F0103  | 触发价格过滤器-步进值                                 |
| ORDER_F0201  | 触发数量过滤器-最小值                                 |
| ORDER_F0202  | 触发数量过滤器-最大值                                 |
| ORDER_F0203  | 触发数量过滤器-步进值                                 |
| ORDER_F0301  | 触发金额过滤器-最小值                                 |
| ORDER_F0401  | 触发开盘保护滤器                                    |
| ORDER_F0501  | 触发限价保护滤器-买单最大偏离度                            |
| ORDER_F0502  | 触发限价保护滤器-卖单最大偏离度                            |
| ORDER_F0601  | 触发市价保护滤器                                    |
| COMMON_001   | 用户不存在                                       |
| COMMON_002   | 系统繁忙，请稍后再试                                  |
| COMMON_003   | 操作失败，请稍后再试                                  |
| CURRENCY_001 | 币种信息异常                                      |
| DEPOSIT_001  | 充值暂未开放                                      |
| DEPOSIT_002  | 当前账号安全等级较低，请绑定手机/邮箱/谷歌身份验证器中的任意两种安全验证后再进行充值 |
| DEPOSIT_003  | 地址格式不正确，请重新输入                               |
| DEPOSIT_004  | 地址已存在，请重新输入                                 |
| DEPOSIT_005  | 冷钱包地址未找到                                    |
| DEPOSIT_006  | 暂无充值地址，请稍后再试                                |
| DEPOSIT_007  | 地址生成中，请稍后再试                                 |
| DEPOSIT_008  | 不支持充值                                       |
| WITHDRAW_001 | 提现暂未开放                                      |
| WITHDRAW_002 | 提币地址不合法                                     |
| WITHDRAW_003 | 当前账号安全等级较低，请绑定手机/邮箱/谷歌身份验证器中的任意两种安全验证后再进行提现 |
| WITHDRAW_004 | 未添加提币地址                                     |
| WITHDRAW_005 | 提币地址不能为空                                    |
| WITHDRAW_006 | Memo不能为空                                    |
| WITHDRAW_008 | 触发风控，暂不支持该币提现                               |
| WITHDRAW_009 | 提现失败，本次提现中部分资产受T+1提币限制                      |
| WITHDRAW_010 | 提币精度不合法                                     |
| WITHDRAW_011 | 可用余额不足                                      |
| WITHDRAW_012 | 提现失败，您今日剩余提现额度不足                            |
| WITHDRAW_013 | 提现失败，您今日剩余提现额度不足，可通过完成更高等级的实名认证提高额度         |
| WITHDRAW_014 | 该笔提现地址不能使用内部转账功能，请取消内部转账功能后再提交              |
| WITHDRAW_015 | 提现金额不足以抵扣手续费                                |
| WITHDRAW_016 | 提币地址已经存在                                    |
| WITHDRAW_017 | 本次提币已处理，无法取消                                |
| WITHDRAW_018 | Memo必须为数字                                   |
| WITHDRAW_019 | Memo不正确，请重新输入                               |
| WITHDRAW_020 | 您今日提现额度已达上限，请明天再试                           |
| WITHDRAW_021 | 您今日提现额度已达上限，本次最多只能提现{0}                     |
| WITHDRAW_022 | 提现金额必须大于{0}                                 |
| WITHDRAW_023 | 提现金额必须小于{0}                                 |
| WITHDRAW_024 | 不支持提现                                       |
| WITHDRAW_025 | 请前往充值页面创建FIO地址                              |
| FUND_001     | 请求重复（一个bizId请求多次接口）                         |
| FUND_002     | 余额不足                                        |
| FUND_003     | 划转操作不支持 （比如子账户不支持理财划入划出）                    |
| FUND_004     | 解冻失败                                        |
| FUND_005     | 划转禁止                                        |
| FUND_014     | 划入账户id和划出账户id不可以一样                          |
| FUND_015     | from和to 业务类型不可相同（用户不可以操作自己现货划转到现货）          |
| FUND_016     | 杠杆交易对不可为空                                   |
| FUND_017     | 参数错误                                        |
| FUND_018     | 冻结记录无效                                      |
| FUND_019     | 解冻用户不相等                                     |
| FUND_020     | 解冻币种不相等                                     |
| FUND_021     | 操作不支持                                       |
| FUND_022     | 冻结记录不存在金额最大长度为113 不可超过限制                    |
| SYMBOL_001   | 交易对不存在                                      |
| TRANSFER_001 | 请求重复（一个bizId请求多次接口）                         |
| TRANSFER_002 | 余额不足                                        |
| TRANSFER_003 | 用户未注册                                       |
| TRANSFER_004 | 币种不允许划转                                     |
| TRANSFER_005 | 用户币种不允许划转                                   |
| TRANSFER_006 | 划转禁止                                        |
| TRANSFER_007 | 请求超时                                        |
| TRANSFER_008 | 杠杆划入异常                                      |
| TRANSFER_009 | 杠杆划出异常                                      |
| TRANSFER_010 | 杠杆清零 划出禁止                                   |
| TRANSFER_011 | 杠杆有借贷 划出禁止                                  |
| TRANSFER_012 | 币种划转禁止                                      |
| GATEWAY_0001 | 触发风控                                        |
| GATEWAY_0002 | 触发风控                                        |
| GATEWAY_0003 | 触发风控                                        |
| GATEWAY_0004 | 触发风控                                        |

----

# 公共模块

## 订单状态码

| State            | 说明                       |
|:-----------------|:-------------------------|
| NEW              | 新建                       |
| PARTIALLY_FILLED | 部分成交                     |
| FILLED           | 全部成交                     |
| CANCELED         | 用户撤单                     |
| REJECTED         | 下单失败                     |
| 内EXPIRED容        | 过期(time_in_force撤单或溢价撤单) |

## 订单类型

| Type   | 说明  |
|--------|-----|
| LIMIT  | 限价单 |
| MARKET | 市价单 |

## 交易对状态

| State    | 说明  |
|----------|-----|
| ONLINE   | 上线的 |
| OFFLINE  | 下线的 |
| DELISTED | 退市的 |

## 有效方式

这里定义了订单多久能够失效

| TimeInForces | 说明               |
|--------------|------------------|
| GTC          | 成交为止,一直有效        |
| IOC          | 无法立即成交(吃单)的部分就撤销 |
| FOK          | 无法全部立即成交就撤销      |
| GTX          | 无法成为挂单方就撤销       |

## 充值/提现记录状态码

| Status        | 说明               |
|:--------------|:-----------------|
| SUBMIT        | 提现: 未冻结          |
| REVIEW        | 提现: 已冻结,待审核      |
| AUDITED       | 提现: 已审核,发送钱包,待上链 |
| AUDITED_AGAIN | 复审中              |
| PENDING       | 充值/提现: 已上链       |
| SUCCESS       | 完成               |
| FAIL          | 失败               |
| CANCEL        | 已取消              |

## BizType

| Status    | Description |
|:----------|:------------|
| SPOT      | 现货          |
| FINANCE   | 理财          |
| FUTURES_U | 合约u本位       |
| UB_CARD   | UB卡账户       |

## 买卖方向

| Status | Description |
|:-------|:------------|
| BUY    | 买           |
| SELL   | 卖           |

----

# FAQ

1. **为什么在多次成功请求后，后续请求返回500个错误码?**

    这种情况可能是由于服务器不稳定造成的。我们建议您稍后再发送请求。如果问题仍然存在，请联系技术支持。

2. **为什么我遇到“无效签名”错误?**

    此错误通常由不正确的签名引起。确保连接顺序和散列方法是准确的。如需进一步帮助，请参阅文档了解详细信息。

3. **为什么我得到“缺少请求头验证-appkey”错误?**

    这意味着您的请求缺少' validate-appkey '标头。请确保您的请求中包含所有必需的标题。

4. **为什么429状态码请求失败?**

    429状态码表示已超过请求速率限制。请减少你的请求频率。

5. **如何避免利率限制?**

    遵循API文档中指定的速率限制。如果需要扩展请求量，请考虑使用多个apikey。

6. **可以修改签名算法吗?**

    目前支持的算法有:HmacMD5、HmacSHA1、HmacSHA224、HmacSHA256、HmacSHA384和HmacSHA512。建议使用HmacSHA256协议。

7. **为什么请求失败，提示“服务不可用”错误?**

    此错误表明该服务当前不可用。请稍后再试。

8. **如果我遇到“服务错误”的回应，我应该怎么做?**

    这通常表明服务器端存在问题。如果错误仍然存在，请联系支持以获得进一步帮助。

9. **AUTH_105：报文过时**

    服务器在校验请求头参数validate-timestamp(validTimeStamp)、validate-recvwindow（recvwindow）时，需要符合以下规则：dealTimeStamp（请求被处理时服务器时间,单位毫秒）- validTimeStamp < recvwindow ，否则就会返回AUTH_105，为了避免此错误，建议validate-timestamp 设置为请求发出的时间，以毫秒为单位，validate-recvwindow设置的大一点

----

# 技术支持

在我们的电报组中获取支持:  [UBX API Support Group](https://t.me/UbitEx_UB)
如有疑问请咨询在线客服