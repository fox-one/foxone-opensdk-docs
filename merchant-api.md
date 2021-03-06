# Merchant API

商户是由 Fox.ONE 来创建的，请联系客服人员。

会分配 `key/secret` 给到商户开发者，然后通过加密生成 JWT 和 API 通讯。

<br>

## ▎HOST

* 测试环境：https://dev-gateway.fox.one
* 生产环境：https://openapi.fox.one


<br>

## ▎请求签名 & HTTP Authorization


### > 请求签名

出于安全的考虑，每一个请求都要附带一个签名（sign），来防止请求被篡改，同时为防重放攻击，所有请求中均需包含 _ts, _nonce 参数，其中 _ts 为当前时间，与服务器时间不得相差半个小时；_nonce 可以使用随机的 uuid，一个小时内不得重复。_ts, _nonce 参数可在 query 里，也可在 json body 里。


#### - 签名示例 golang

请求的签名为 base64 (sha256(http_method+uri+payload))


```go
func SignAuthenticationToken(method, uri, body string, key, secret string) (string, error) {
    expire := time.Now().Add(time.Hour * 24 * 30 * 3)
    sum := sha256.Sum256([]byte(method + uri + body))
    claims := jwt.MapClaims{
        "exp":  expire.Unix(),
        "sign": base64.StdEncoding.EncodeToString((sum[:])),
        "key":  key,
    }

    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString([]byte(secret))
}
```

### > HTTP Authorization

鉴权使用 JWT，HTTP 请求时，在 Header 中添加 "Authorization: Bearer token"

#### - JWT 使用说明

* JWT 的签名算法为 `HS256 (HMAC with SHA-256)`
* 签名密钥为 Merchant secret（由 Fox.ONE  签发）

#### - JWT Payload

```javascript
{
    "exp": 11111,  // 过期时间，unix timestamp
    "key": "xxx",  // memrchant key
    "sign": "xxx", // 请求签名
    "mem": "xxx"   // optional, member id, 只在指定用户的请求时需要。如查询某用户余额
}
```


### > PIN 签名的 RSA的公钥

公钥是固定的，但在不同的环境中需要设置相对应环境变量所使用的值，变量  key 为：`FOX_GATEWAY_PUBLIC_KEY`


```
os.SetEnv("FOX_GATEWAY_PUBLIC_KEY","pk")
```


- **prod:**

```text
-----BEGIN RSA PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA0/SMrN1Ki50xAD0mjIjA
NroYtZ+dtFh9i2gT8ANy9ObQplKJQedM5VDviEqnNiyNgQj6byso3EnykgG7JbpQ
qwt7XgAwO+uE01EdGi46G59DzvobBfwchmV9q9caHE0od95XukCq7vQzlpL/IS2+
BWaG6RjYeqcE7mxdmcVIzQ6ifcY4tfcAnEXqVz5kAcKM+GbLVDOhdeb3LPpkydNT
Li+q8vY1PrnnWDlGnJORosBuRS5IXab7QxojKFx1lrq4EvnKGeyB6m3+h14Ixlcv
/QO5p7RR4lI9hs11Ecatritck25xQQ+YO4n0gYAvScxV0t0nQGBjmsN11Nm4Hl1x
kwIDAQAB
-----END RSA PUBLIC KEY-----
```

- **dev:**

```text
-----BEGIN RSA PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA5PbIXYPU+leiueNmwzbk
u05eXRJ79gQCPyS3mwoUIySAMmPaU2/RN47lhhJO3YHJQ/J/u2jqgTntQKdmmjei
0L0odUyL3JaSrqirLi6JVWKaF3o9YkX88xwyLLWNdWywFPL9CPWcqLvTbymrS2zN
l0U8zGVXft+aDlRYdCaBtQWF1a2tmNpKXfOlPOaZotXO/iN4Diqpl+vTqUqRxb0q
FkPcAH1XKpWTDP7XSAVLyh2CIf2GEZFzt55nMudfUXMEwv0aAUIgsxK/Yk2cyTbY
qrYnTJbX/WysMtg+vhVy7DJznwx5sPl1huPO5ytfwTagKgQArF34WfLEB7OIZuZL
+QIDAQAB
-----END RSA PUBLIC KEY-----
```



<br>

## ▎API 使用说明

### > Validate Token

> 验证 Token 有效性

- URL: /merchant/validate
- Method: POST
- Params:

```json
{
  "uri": "/api/xxx",
  "method": "POST",
  "body": "xxx",
  "token": "xxxx"
}
```

#### - Return:

```json
{
  "merchant_id": "xxx",
  "member_id": "xxx"
}
```

### > Merchant Services

> 商户开启的业务

- URL: /merchant/services
- Method: GET


#### - Return:

```javascript
[{
    "merchant_id": "xxx",
    "service": "exchange",
    "wallet_id": "xxx",
    "status": 1, // 1 OK; 2 Frozen
    "created_at": 1234,
}]
```

### > 创建用户

> 商户创建用户

- URL: /merchant/member/new
- Method: POST

#### - Return:

```javascript
{  
   "member":{  
      "id":"987aeabe84654087bb3cb5ca602c7b1f",
      "created_at":1562125853,
      "is_pin_set":false
   },
   "session":{  
      "key":"874869407833634c678ff766dbc4b472-987aeabe84654087bb3cb5ca602c7b1f",
      "secret":"79ce901ee75bc9afc48856b0c3109370",
      "created_at":1562125853,
      "expired_at":1562212253
   },
   "wallets":[  
      {  
         "label":"",
         "member_id":"987aeabe84654087bb3cb5ca602c7b1f",
         "service":"payment",
         "wallet_id":"874ac4c9-0042-3099-8e2e-28111928510a",
         // the session_id and session_key are only available when you create a member
         // the wallet_id 、 session_id and session_key can be used to access Mixin Api
         "session_id":"75bd97b5-724b-4f54-bcde-a412fd776794",
         "session_key":"-----BEGIN RSA PRIVATE KEY-----\nMIICXAIBAAKBgQDZ2RYv1EM4sqXB7YURO7YgMPCih3OO+G6ERZpxYcPItI34fXa/\nUo9aLvi1doiyONM1F40lE9fr1Ahx6rmmzJPPNFcKH6lhDlenSWKVrg4KKFcK/EMD\nxHgGZxzfJB3sPOWLnCIgNwk1rffQGVcumWF3E8aND00Nl7D1IVlLk2N3OwIDAQAB\nAoGATp0adpQg1fsR+hOeq4Niy+cdT2mV+AgKyczcWQIwxuLxQLT1/0Dp3l+I/OMT\nnU0IWuZu1ux8ROw1R/aunFTDGZ5OMUE9kr3w5xDdMV7cdTxJNwweUJW6Hz6raoe3\nTheR8J7PHkDp2tObU69PrrZrUScj3v8Eybpy7R6v4/jDg5ECQQDe8/nNjJkuUUqC\nVuCjyEylteulvSiC7hGF85tHUgWZ9HZJ5tM2Uww39qONYJzv3CzIZGS2pomtz06b\n7GJn9UPNAkEA+iNmDp+VogacfkJCn9NuqAi4Z2qBkETSFLjZqWG8btqQ82lhTyqi\n+57wNDRriYBb3lkFLbkEYyb/PkEK72GvJwJABWAEabw2BTPYhAPsLoapsmUMZVaG\nH4H10jDpUXLcx7VpFKcH+ItQBBliIApwPigkvEAPXYfuUc5pqsCsLq1vEQJBANwA\nUF3iPDAiknd1/bUmt/ewm8fRZB0oeoFhR4dzf9EcCUsdT0na3ThjxS6VQFPSgnqg\nXy6kwNgYT3xIpr5+cxcCQA8f2VcFt/3QT/uOpmQtA4AbmH/A7fbycZ1RG7I13FtH\nROhqSZrBCS2IYdYJ+RrNJhhW6Nmu21mEvu7fwFNVu+g=\n-----END RSA PRIVATE KEY-----\n"
      }
   ]
}
```


### > 登陆用户

- URL: /merchant/member/login
- Method: POST
- Params:

```javascript
{
    "id": "member_id",
    "expire": 1111 // active duration in seconds
}
```

#### - Return:

```javascript
{
    "member": {
        "id": "xxx",
        "created_at": 123,
        "is_pin_set": false,
    },
    "session": {
        "key": "xxx",
        "secret": "xxx",
        "created_at": 123,
        "expired_at": 123,
    }
}
```

### > 登出用户

- URL: /merchant/member/logout
- Method: POST

#### - Params:

```javascript
{
    "id": "member_id",
    "session_key": "xxx" // optional
}
```

#### - Return:

```json
{}
```


### > 获取用户信息

- URL: /merchant/member/info
- Method: GET


#### - Return:

```javascript
{  
   "id":"e82e7a6ed335451b83fe33e7477b03ac",
   "created_at":1562142129,
   "is_pin_set":false
}
```

### > 为用户设置 PIN 

- PUT /merchant/member/pin

### -jwt payload
``` javascript
{
  "exp": 1562211730,
  "key": "944dd94adc135c94be3dc9f6c1c6ed43",
  "mem": "bfbebc70de9f40f7a35ea247bdf7b5ce",
  "pin": "GffHqNSvfHb58s23NhprcAuPsT5Zjyjc7Un5WJa3obxbbXx0Y6YWHthMK2ptwlSbgJsPrsJv1WxbteEhYx7rQDOWzq6cMWrDIhklMuZc1h8fRj0mWmeGDbqiZoiixypfqdrEevLCW3avg1QjmMyecT2tvGBrkpA8CSD1SGLu7B3OXJV6Brjpo1Dur2+8H/61V/9CSTaFBZ3PxoMJE9K4GD9jgDohvL0S4+unjkS49g7yYMIpA8T14wpLOYHetRIBcuqk2kuAUmr/VhuWcLKe3sql1ascf8EaKChOP9rvbw+ybydZT+Jd7Dpek+T0KiuH891GU+XSybFQnVZs3B9g1g==",  // if required
  "sign": "QO6wsapg5iiBk6tRcFlhg/7D+1r4vmHCcmvn7oDrW+s="
}
```

### Note
pin为使用RSA OAEP算法对pin加密后的结果，加密时需包含如下信息
```javascript
{
    "t": 1562199964,                // current unix timestamp
    "n": "somethin unique",         // nonce, usually the uuid
    "p": "123456"                   // your raw pin
}
```

> If `is_pin_set` is false, do not put pin in the authorization token.
Namely if the member is newly created, use this api to set pin without decrypted pin in token.
Otherwise put decrypted pin in token to modify pin.

#### - Params:

```javascript
{
    "pin" : "mAvjFxe8Ozkr8pO7xImEL0H83Ja6JCkVS0mT029V+nC3pM9xjHZFX50882+iY+GNNhqVQC2cWHkiYJBoum9BkW3NhL0js4AFULRKFXoKnkMOoXlgAKcYZmE/cihJnwR6L6KeN7c8ycTYPUNBnSxvkvT2PcD59xBqDEyXcB4LnaL5AncpevtTAiejFE25u2CsLl/4EB0s42+yxbj5CTob73RUhgVndJB9b38MEQn7cDKmod/Uej7/T+/K47DMEvOmndt+YGNlK8iBfViO6T/QOZC0JKs5yWkaZ8wyfRyW9bT1oqDriIzVvFxHCQXPWVPoK/h25EEfHOVfdzBXLgGM2g=="
}
```

### >  Verify PIN

- POST /merchant/member/pin


#### - Return:

``` javascript
{}
```



### > 查询用户业务钱包列表

- URL: /merchant/member/services?member_id=xxx&service=xxx
- Method: GET

#### - Return:

```javascript
[{
    "member_id": "xxx",
    "wallet_id": "xxx",
    "service":   "exchange",
    "label":     "xxx",
}]
```

> service为可选参数。若不传则查询所有


### > 查询用户转账记录

- URL: /merchant/member/snapshots
- Method: GET

#### - Param: 

?service=exchange&asset_id=xxx&cursor=xxx&order=ASC|DESC&limit=50

#### - Return:

```javascript
{
    "snapshots" : [{
      "snapshot_id": "56048f80-ff17-44a2-81ea-bd96840cdec5",
      "trace_id": "1e6c2635-11de-34df-bc12-f10803895e06",
      "wallet_id": "8256c737-a11a-3b0d-bd25-9e18aac7db96",
      "asset_id": "c94ac88f-4671-3976-b60a-09064f1811e8",
      "opponent_id": "91e067e6-4bfd-309f-9d46-5cf06a778a6c",
      "source": "MATCH",
      "amount": "0.00004884",
      "memo": "haFBsJZeXG5DTD+pt4DFD0PNlVyhQ6NBU0uhT7DHgfWrBz5LJ5KFdTU4evAboVCpMC4wMDAwMDE0oVOlTUFUQ0g=",
      "member_id": "137b4c927fc1441a8524345e7c532a18",
      "service": "exchange",
      "created_at": 1551376859938,
      "asset": {
        "asset_id": "c94ac88f-4671-3976-b60a-09064f1811e8",
        "asset_key": "0xa974c709cfb4566686553a20790685a47aceaa33",
        "chain_id": "43d61dcd-e413-450d-80b8-101d5e903357",
        "name": "Mixin",
        "symbol": "XIN",
        "icon_url": "https://www.fox.one/assets/coins/xin.png",
        "price": "956.79711445",
        "price_usd": "143.179558669",
        "change": "0.00588117"
      }
    }],
    "pagination" : {
        "next_cursor" : "MTU1MTM1NTU0MDA4ODQ0NjAwMDsx",
        "has_next" : true
    }
}
```

> service 必选，可选 exchange,otc,payment...
