# PrivateSDK Document for Backend

This document is used for backend to receive notification info of logged-in user and payment.

## Table of contents

[Before you start](#before-you-start)
1. [Getting Data from SDK Server](#getting-data-from-sdk-server)
   1. [Get user info](#get-user-info)
   2. [Receive payment info](#receive-payment-info)
2. [Returning data for SDK Server](#returning-data-for-sdk-server)
   1. [Get character list of user](#get-character-list-of-user)

### Before you start

You need to have the following:

- `app_id`: This is ID of your game on our system (be provided in `document`).
- The `payment callback url` should be set on our system

## Getting Data from SDK Server
This is set of APIs that you can use for getting data from SDK server.
>Note: All API must be implemented fully.
### Get user info

Endpoint: https://vngates.com/api/userinfo

Method: `HTTP/POST`

Content Type:   `application/x-www-form-urlencoded`

Parameter:

| Field        | Type     | Description                              |
|--------------|----------|------------------------------------------|
| app_id       | `String` | The id of your game (`provided`)         |
| access_token | `String` | access token of logged-in user           |
| sign         | `String` | hash value of `md5(app_id+access_token)` |

##### Example

```
curl --location --request POST 'https://vngates.com/api/userinfo' \
> --header 'Content-Type: application/x-www-form-urlencoded' \
> --data-urlencode 'sign=83bb790e425b0eaa251e94ede00da20f' \
> --data-urlencode 'app_id=39faa2b8dc7023c0705307371537a023' \
> --data-urlencode 'access_token=b8a4c967ea135a4dcfca38162111fef814a8ba21c2f04a189526eb7c7494501b'
```

If this request succeeds, SDK backend returns the user info, which is the HTTP 200 code and a JSON response body that
shows captured info.

```
{
  "access_token": "b8a4c967ea135a4dcfca38162111fef814a8ba21c2f04a189526eb7c7494501b",
  "refresh_token": "1dfad604a0de526872fdf3ecd7834df6",
  "refresh_expires": 1668698125455,
  "data": {
    "user_id": "229",
    "username": "gamegate36i"
  },
  "access_expires": 1668104125455,
  "status": 1
}
```

| Field           | Type          | Description                                                                |
|-----------------|---------------|----------------------------------------------------------------------------|
| status          | `int`         | The result code of respone:<br/>0: Error<br/>1: Successful                 |
| msg             | `String`      | The respone message: error message detail, successful message,... or empty |
| access_token    | `String`      | Access token                                                               |
| refresh_token   | `String`      | Refresh token                                                              |
| access_expires  | `long`        | Expire time  of Access token                                               |
| refresh_expires | `long`        | Expire time  of Refresh token                                              |
| data            | `Json Object` | User info detail                                                           |
| user_id         | `String`      | User id                                                                    |
| username        | `String`      | User name                                                                  |

### Receive payment info

Whenever a payment process is completed, we notify you of the event and whether it was successful or not. The result will be sent to the `callback url`.


Endpoint(`callback url`): https://your-server-domain/addgold 

Method: `HTTP/GET`

##### Example

```
{
  "transaction_id": "45179546928220",
  "amount": 1000.00,
  "user_id": "229",
  "product_id": "com.team.demo.goivang1",
  "sign": "e5f813c5067e46dff79127bfdc2c6701",
  "char_id": "43",
  "response_time": 1665067951903,
  "server_id": "1",
  "status": 1
}
```

| Field          | Type     | Description                                                                |
|----------------|----------|----------------------------------------------------------------------------|
| status         | `int`    | The result code of respone:<br/>0: Error<br/>1: Successful                 |
| transaction_id | `String` | Transaction id                                                             |
| amount         | `float`  | amount                                                                     |
| product_id     | `String` | Product id                                                                 |
| sign           | `String` | Expire time  of Refresh token                                              |
| char_id        | `String` | Char id                                                                    |
| server_id      | `String` | Server id                                                                  |
| response_time  | `long`   | The time at response is sent                                               |

## Returning data for SDK Server
This is set of APIs that you must implement for returning data for SDK server.

### Get character list of user  

Endpoint: https://your-server-domain/rolelist

Method: `HTTP/GET`


| Field   | Type     | Description                  |
|---------|----------|------------------------------|
| user_id | `String` | User id                      |
| sign    | `String` | hash value of `md5(user_id)` |

If this request succeeds, your backend returns the character list of requested user id.

```
{
  "msg": "success",
  "data": [
    {
      "role_name": "Boombu",
      "server_name": "TestBom",
      "role_id": "105",
      "login_time": 1667812169000,
      "server_id": "100"
    }
  ],
  "status": 1
}
```


| Field       | Type          | Description                                                                |
|-------------|---------------|----------------------------------------------------------------------------|
| status      | `int`         | The result code of respone:<br/>0: Error<br/>1: Successful                 |
| msg         | `String`      | The respone message: error message detail, successful message,... or empty |
| data        | `Json Object` | Character list info detail                                                 |
| role_id     | `String`      | Character id                                                               |
| role_name   | `String`      | Character name                                                             |
| server_id   | `String`      | Server id                                                                  |
| server_name | `String`      | Server name                                                                |
| login_time  | `long`        | Login time                                                                 |
