[![Go Report Card](https://goreportcard.com/badge/plutov/paypal)](https://goreportcard.com/report/plutov/paypal)
[![Build Status](https://travis-ci.org/plutov/paypal.svg?branch=master)](https://travis-ci.org/plutov/paypal)
[![Godoc](http://img.shields.io/badge/godoc-reference-blue.svg?style=flat)](https://godoc.org/github.com/plutov/paypal)

### Go client for PayPal REST API

当前只支持 **v2** 版本, 如果你想要使用 **v1** 版本API, 请使用版本 **v1.1.4** git tag.

### 总览

 * POST /v1/oauth2/token
 * POST /v1/identity/openidconnect/tokenservice
 * GET /v1/identity/openidconnect/userinfo/?schema=**SCHEMA**
 * POST /v1/payments/payouts
 * GET /v1/payments/payouts/**ID**
 * GET /v1/payments/payouts-item/**ID**
 * POST /v1/payments/payouts-item/**ID**/cancel
 * GET /v1/payment-experience/web-profiles
 * POST /v1/payment-experience/web-profiles
 * GET /v1/payment-experience/web-profiles/**ID**
 * PUT /v1/payment-experience/web-profiles/**ID**
 * DELETE /v1/payment-experience/web-profiles/**ID**
 * POST /v1/vault/credit-cards
 * DELETE /v1/vault/credit-cards/**ID**
 * PATCH /v1/vault/credit-cards/**ID**
 * GET /v1/vault/credit-cards/**ID**
 * GET /v1/vault/credit-cards
 * GET /v2/payments/authorizations/**ID**
 * POST /v2/payments/authorizations/**ID**/capture
 * POST /v2/payments/authorizations/**ID**/void
 * POST /v2/payments/authorizations/**ID**/reauthorize
 * GET /v2/payments/sale/**ID**
 * POST /v2/payments/sale/**ID**/refund
 * GET /v2/payments/refund/**ID**
 * POST /v2/checkout/orders
 * GET /v2/checkout/orders/**ID**
 * PATCH /v2/checkout/orders/**ID**
 * POST /v2/checkout/orders/**ID**/authorize
 * POST /v2/checkout/orders/**ID**/capture
 * GET /v2/payments/billing-plans
 * POST /v2/payments/billing-plans
 * PATCH /v2/payments/billing-plans/***ID***
 * POST /v2/payments/billing-agreements
 * POST /v2/payments/billing-agreements/***TOKEN***/agreement-execute
 * POST /v1/notifications/verify-webhook-signature

### Missing endpoints
在此SDK中有可能会出现错误 "missing endpoints", 你可以使用内建函数处理此类问题: **NewClient -> NewRequest -> SendWithAuth**
### 创建客户端

```go
import "github.com/plutov/paypal"

// 如果使用go mod
// import "github.com/plutov/paypal/v3" 

// 创建一个客户端实例
c, err := paypal.NewClient("clientID", "secretID", paypal.APIBaseSandBox)
c.SetLog(os.Stdout) // Set log to terminal stdout

accessToken, err := c.GetAccessToken()
```

### 通过ID获取认证信息

```go
auth, err := c.GetAuthorization("2DC87612EK520411B")
```

### 获取授权

```go
capture, err := c.CaptureAuthorization(authID, &paypal.Amount{Total: "7.00", Currency: "USD"}, true)
```

### 废止授权

```go
auth, err := c.VoidAuthorization(authID)
```

### 重新获取授权

```go
auth, err := c.ReauthorizeAuthorization(authID, &paypal.Amount{Total: "7.00", Currency: "USD"})
```

### 根据ID获取销售单

```go
sale, err := c.GetSale("36C38912MN9658832")
```

### 根据ID对销售单退款

```go
// 全部退款
refund, err := c.RefundSale(saleID, nil)
// 部分退款
refund, err := c.RefundSale(saleID, &paypal.Amount{Total: "7.00", Currency: "USD"})
```

### 根据ID获取退款

```go
refund, err := c.GetRefund("O-4J082351X3132253H")
```

### 根据ID获取订单

```go
order, err := c.GetOrder("O-4J082351X3132253H")
```

### 创建订单

```go
order, err := c.CreateOrder(paypal.OrderIntentCapture, []paypal.PurchaseUnitRequest{paypal.PurchaseUnitRequest{ReferenceID: "ref-id", Amount: paypal.Amount{Total: "7.00", Currency: "USD"}}})
```

### 更新订单

```go
order, err := c.UpdateOrder("O-4J082351X3132253H", []paypal.PurchaseUnitRequest{})
```

### 授权订单

```go
auth, err := c.AuthorizeOrder(orderID, paypal.AuthorizeOrderRequest{})
```

### 获取订单

```go
capture, err := c.CaptureOrder(orderID, paypal.CaptureOrderRequest{})
```

### 认证

```go
token, err := c.GrantNewAccessTokenFromAuthCode("<Authorization-Code>", "http://example.com/myapp/return.php")
// ... or by refresh token
token, err := c.GrantNewAccessTokenFromRefreshToken("<Refresh-Token>")
```

### 根据open id获取用户信息

```go
userInfo, err := c.GetUserInfo("openid")
```

### 创建一次性付款到邮箱中

```go
payout := paypal.Payout{
    SenderBatchHeader: &paypal.SenderBatchHeader{
        EmailSubject: "Subject will be displayed on PayPal",
    },
    Items: []paypal.PayoutItem{
        paypal.PayoutItem{
            RecipientType: "EMAIL",
            Receiver:      "single-email-payout@mail.com",
            Amount: &paypal.AmountPayout{
                Value:    "15.11",
                Currency: "USD",
            },
            Note:         "Optional note",
            SenderItemID: "Optional Item ID",
        },
    },
}

payoutResp, err := c.CreateSinglePayout(payout)
```

### 根据ID获取支付单

```go
payout, err := c.GetPayout("PayoutBatchID")
```

### 根据ID获取支付项目

```go
payoutItem, err := c.GetPayoutItem("PayoutItemID")
```

### 根据ID取消一个未使用的支付单

```go
payoutItem, err := c.CancelPayoutItem("PayoutItemID")
```

### 创建一个网络购物清单

```go
webprofile := WebProfile{
    Name: "YeowZa! T-Shirt Shop",
    Presentation: Presentation{
        BrandName:  "YeowZa! Paypal",
        LogoImage:  "http://www.yeowza.com",
        LocaleCode: "US",
    },

    InputFields: InputFields{
        AllowNote:       true,
        NoShipping:      NoShippingDisplay,
        AddressOverride: AddrOverrideFromCall,
    },

    FlowConfig: FlowConfig{
        LandingPageType:   LandingPageTypeBilling,
        BankTXNPendingURL: "http://www.yeowza.com",
    },
}

result, err := c.CreateWebProfile(webprofile)
```

### 获取网络购物单

```go
webprofile, err := c.GetWebProfile("XP-CP6S-W9DY-96H8-MVN2")
```

### 列出购物单

```go
webprofiles, err := c.GetWebProfiles()
```

### 更新购物单

```go

webprofile := WebProfile{
    ID: "XP-CP6S-W9DY-96H8-MVN2",
    Name: "Shop YeowZa! YeowZa! ",
}
err := c.SetWebProfile(webprofile)
```

### Delete web experience profile

```go
err := c.DeleteWebProfile("XP-CP6S-W9DY-96H8-MVN2")
```

### 信用卡存储

```go
// Store CC
c.StoreCreditCard(paypal.CreditCard{
    Number:      "4417119669820331",
    Type:        "visa",
    ExpireMonth: "11",
    ExpireYear:  "2020",
    CVV2:        "874",
    FirstName:   "Foo",
    LastName:    "Bar",
})

// 删除信用卡存储
c.DeleteCreditCard("CARD-ID-123")

// 编辑信用卡存储
c.PatchCreditCard("CARD-ID-123", []paypal.CreditCardField{
    paypal.CreditCardField{
        Operation: "replace",
        Path:      "/billing_address/line1",
        Value:     "New value",
    },
})

// 获取存储的信用卡
c.GetCreditCard("CARD-ID-123")

// 列出所有保存的信用卡
c.GetCreditCards(nil)
```

### 如何贡献

* Fork 仓库
* 新增/修改 一些东西
* 检查并测试通过
* 创建PR

当前贡献者:

- [Roopak Venkatakrishnan](https://github.com/roopakv)
- [Alex Pliutau](https://github.com/plutov)

### 测试

* 单元测试: `go test -v ./...`
* 集成测试: `go test -tags=integration`
