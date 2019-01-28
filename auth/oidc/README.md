# OpenID Connect

OpenID Connect 通常簡稱 *OIDC*。它有幾份[規格](https://openid.net/developers/specs/)（specifications）需要先讀如下：

| 全名 | 用途 |
| --- | --- |
| [OpenID Connect Core](https://openid.net/specs/openid-connect-core-1_0.html) | 定義 OIDC 的核心功能 |
| [OpenID Connect Discovery](https://openid.net/specs/openid-connect-discovery-1_0.html) | 定義 Client 如何動態的發現 OP 的資訊 |
| [OpenID Connect Dynamic Registration](https://openid.net/specs/openid-connect-registration-1_0.html) | 定義 Client 如何動態跟 OP 註冊 |
| [OAuth 2.0 Multiple Response Types](https://openid.net/specs/oauth-v2-multiple-response-types-1_0.html) | 為 OAuth2 定義幾個新的 response types |
| [OAuth 2.0 Form Post Response Mode](https://openid.net/specs/oauth-v2-form-post-response-mode-1_0.html) | 定義如何使用 HTTP Post 自動提交 HTML Form，並拿到 OAuth2 授權的 response parameters |
| [OpenID 2.0 to OpenID Connect Migration 1.0](https://openid.net/specs/openid-connect-migration-1_0.html) | 如何從 OpenID 2.0 遷移成 OpenID Connect |

## Flow

### Authorization Code Flow

流程文字描述如下：

1.  Client 準備帶有必要參數的驗證請求（Authentication Request）
2.  Client 將驗證請求發送給 Authorization Server
3.  Authorization Server 驗證 End-User 身份
4.  Authorization Server 取得 End-User 授權
5.  Authorization Server 將授權碼（Authorization Code）透過 End-User 送回給 Client
6.  Client 使用授權碼向 Authorization Server 的 token endpoint 發送請求
7.  Client 收到代表 End-User 身份的 ID Token 跟 Access Token
8.  Client 驗證 ID Token 並取出使用者資訊

### Implicit Flow

流程文字描述如下：

1.  Client 準備帶有必要參數的驗證請求（Authentication Request）
2.  Client 將驗證請求發送給 Authorization Server
3.  Authorization Server 驗證 End-User 身份
4.  Authorization Server 取得 End-User 授權
5.  Authorization Server 將 ID Token 透過 End-User 送回給 Client，如果有額外要求的話，會再附加 Access Token
6.  Client 驗證 ID Token 並取出使用者資訊

### Hybrid Flow

流程文字描述如下：

1.  Client 準備帶有必要參數的驗證請求（Authentication Request）
2.  Client 將驗證請求發送給 Authorization Server
3.  Authorization Server 驗證 End-User 身份
4.  Authorization Server 取得 End-User 授權
5.  Authorization Server 將授權碼（Authorization Code）透過 End-User 送回給 Client，並且依據 response type 不同，會有數個額外的參數
6.  Client 使用授權碼向 Authorization Server 的 token endpoint 發送請求
7.  Client 收到代表 End-User 身份的 ID Token 跟 Access Token
8.  Client 驗證 ID Token 並取出使用者資訊

## References

* https://medium.com/@darutk/diagrams-of-all-the-openid-connect-flows-6968e3990660
