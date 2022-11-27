# PrivateSDK Document for PC/Embed Web Browser platform


Beside providing solutions for mobile platforms, we also provide PC/Embed Web platform. You can use the web address and api to integrate into your PC/Embed Web Browser platform instance to enhance the user experience.


## Table of contents

1. [Integrate user system with Embed Web Browser](#integrate-user-system-with-embed-web-browser)
2. [Login with username and password](#login-with-username-and-password)




### Integrate user system with Embed Web Browser

URL: https://vngates.com/index

| Field     | Value      | Description                      |
|-----------|------------|----------------------------------|
| app_id    | `required` | The id of your game (`provided`) |
| device_id |            | Device id value, always equal "" |
| device_os | `2`        | Device OS value, always equal 2  |
| tab       | `1` or `2` | Index of tab on UI               |

Example:

https://vngates.com/index?app_id=c2eef5ceaed840535db7fd185f226fd7&device_id&device_os=2&tab=1



![Embed Web Browser](/images/15.png)

When user input all required info at Login/Register form and press Submit button `[Đăng nhập]/[Đăng ký]`, SDK server will handle and redirect to a new address with `access_token`.

Example: sample redirect URL

https://vngates.com/api/logincallback?status=1&access_token=40301707295540806b332117fb77bed8b3ee5a88c4aa4531aff1e5fe64010d21&username=demowebview


| Field        | Type     | Description                                                |
|--------------|----------|------------------------------------------------------------|
| status       | `int`    | The result code of respone:<br/>0: Error<br/>1: Successful |
| username     | `String` | Input field from user                                      |
| access_token | `String` | Access token                                               |

You need to get `access_token` field from the above `redirect URL` and send it to server game. Server game will get user info by using [Get User Info](<https://github.com/GameGateSDK/SDKDocument/tree/master/backend/README.md>) API and allow user to enter game.

### Login with username and password

After user completed the registration step, you can use this API to let them login into game.

Endpoint: https://vngates.com/api/login


Method: `HTTP/POST`

Content Type:   `application/x-www-form-urlencoded`

Parameter:

| Field      | Type     | Description                                              |
|------------|----------|----------------------------------------------------------|
| app_id     | `String` | The id of your game (`provided`)                         |
| username   | `String` | Input value from user                                    |
| password   | `String` | Input value from user                                    |
| app_secret | `String` | The app secret of your game (`provided`)                 |
| sign       | `String` | hash value of `md5(app_id+username+password app_secret)` |

Example:

```
curl --location --request POST 'https://vngates.com/api/login' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'app_id=39faa2b8dc7023c0705307371537a023' \
--data-urlencode 'username=boom1' \
--data-urlencode 'password=e10adc3949ba59abbe56e057f20f883e' \
--data-urlencode 'sign=cdee7dfd78bdfc2596f51cc8a0a614fe'
```

Respone:


```
{
  "access_token": "d50f8af4e0b57624470f031405919fd7c8adce4dfef24c6d8af80847eb2116a3",
  "status": 1
}
```


| Field        | Type     | Description                                                                |
|--------------|----------|----------------------------------------------------------------------------|
| status       | `int`    | The result code of respone:<br/>0: Error<br/>1: Successful                 |
| msg          | `String` | The respone message: error message detail, successful message,... or empty |
| access_token | `String` | Access token                                                               |



You need to get `access_token` field from the response and send it to server game. Server game will get user info by using [Get User Info](<https://github.com/GameGateSDK/SDKDocument/tree/master/backend/README.md>) API and allow user to enter game.
