## API Signature

### Getting Your API Key and Secret Key

> For Testnet
<a href="https://testnet.bybit.com/user/api-management">https://testnet.bybit.com/user/api-management</a>

> For Mainnet
<a href="https://www.bybit.com/app/user/api-management">https://www.bybit.com/app/user/api-management</a>

### HTTP Requests
* `GET` requests should be in the `application/x-www-form-urlencoded` format
* `POST` requests should be in the `application/json` format

### Limits

#### API Request Rate Limits
> The rate limit is accurate to milliseconds
##### Viewing Your Request Rate Limit
On each request to the API, these fields are return：
```
"rate_limit_status":119,
"rate_limit_reset_ms":1572114055663815,
"rate_limit":120
```
* rate_limit you current limit; 
* rate_limit_status remaining requests; 
* rate_limit_reset_ms the timestamp reset you request limit, If you have not exceeded your limit, this value is always the current timestamp，the timestamp unit is millisecond.

##### endpoints rate limit details

  <escape>
    <table>
      <tr>
        <th>limit</th>
        <th>path</th>
      </tr>
      <tr>
        <td rowspan="6">100/min</td>
        <td>open-api/order/create </td>
      </tr>
      <tr><td>open-api/order/cancel       </td></tr>
      <tr><td>open-api/stop-order/create  </td></tr>
      <tr><td>open-api/stop-order/cancel  </td></tr>
      <tr><td>open-api/order/replace      </td></tr>
      <tr><td>open-api/stop-order/replace </td></tr>
      <tr>
        <td rowspan="3">600/min</td>
        <td>open-api/order/list </td>
      </tr>
	    <tr><td>open-api/stop-order/list </td></tr>
      <tr><td>v2/private/order </td></tr>
      <tr>
        <td>120/min</td>
        <td>v2/private/execution/list</td>
      </tr>
      <tr>
        <td rowspan="3">75/min</td>
        <td>leverage/save  </td>
      </tr>
      <tr><td>position/change-position-margin </td></tr>
      <tr><td>position/trading-stop           </td></tr>
      <tr>
        <td rowspan="2">120/min</td>
        <td>position/list  </td>
      </tr>
      <tr><td>user/leverage</td></tr>
      <tr>
        <td rowspan="3">120/min</td>
        <td>open-api/funding/prev-funding-rate  </td>
      </tr>
      <tr><td>open-api/funding/prev-funding      </td></tr>
      <tr><td>open-api/funding/predicted-funding </td></tr>
      <tr>
        <td rowspan="2">120/min</td>
        <td>open-api/wallet/fund/records  </td>
      </tr>
	  <tr><td>open-api/wallet/withdraw/list </td></tr>
    <tr>
        <td rowspan="1">600/min</td>
        <td>open-api/api-key  </td>
      </tr>
    </table>
  </escape>

#### Order Count Limits
    * Each account can hold up to 200 active orders yet to be filled entirely simultaneously.
    * Each account can hold up to 5 conditional orders in the same side simultaneously.

#### How To Raised API Limit Threshold
    * Please send application email to api@bybit.com, We will reply in 3-5 working days.

### Authentication

When calling the API, you need to provide your API key as identification for every request. In addition, a signature of the request you are making is required, which should be signed using the corresponding API secret.

#### Public Parameters
Name | Description | Type | Required | Default value| Comments
:- | :- | :- | :- | :- | :-
api_key | API_KEY requested from platform | string | yes | -- | For identification.
timestamp | timestamp when initiating request, unit: millisecond | int64 | yes | -- | UTC timestamp, server will verify this parameter when it receives the request. Rule of verification: timestamp < server_time + 1000.
recv_window| valid request timespan, unit: millisecond| int | no | 5000 | An http request will be invalid after this time: timestamp+recv_window. This is to prevent replay attacks.
sign | signature message |  string | yes | no | The signature message which is generated from a certain algorithm.

#### How to Sign
1. Concatenate all the public parameters except 'sign' in the query string format. The parameters must be ordered in **ascending** order. Here is an example of adjusting the leverage of an account (using the 'symbol' and 'leverage' parameters):

``` js
var param_str = 'api_key=B2Rou0PLPpGqcU0Vu2&leverage=100&symbol=BTCUSD&timestamp=1542434791000';
```

2. Sign the parameters string.
```js
var secret = 't7T0YlFnYXk0Fx3JswQsDrViLg1Gh3DUU5Mr';
var sign = hex(HMAC_SHA256($secret, $param_str));
// sign = 670e3e4aa32b243f2dedf1dafcec2fd17a440e71b05681550416507de591d908
```

3. Append the signature at the end of parameters string, and send the HTTP request.
Please note that the format for messages is different depending on whether you are sending a GET or POST request:

GET requests:

```http
GET /user/leverage/save HTTP/1.1
Host: api-testnet.bybit.com
Content-Type: application/x-www-form-urlencoded

api_key=B2Rou0PLPpGqcU0Vu2&leverage=100&symbol=BTCUSD&timestamp=1542434791000&sign=670e3e4aa32b243f2dedf1dafcec2fd17a440e71b05681550416507de591d908

```

POST requests:

```http
POST /user/leverage/save HTTP/1.1
Host: api-testnet.bybit.com
content-type: application/json

{
    "api_key":"B2Rou0PLPpGqcU0Vu2",
    "leverage":100,
    "symbol":"BTCUSD",
    "timestamp":1542434791000,
    "sign":"670e3e4aa32b243f2dedf1dafcec2fd17a440e71b05681550416507de591d908"
}
```

### Response Messages

Name | Description | Sample value|
:-: | :-: | :-:
ret_code | return code (0：success, otherwise failure) | 0
ret_msg | return message | ok
ext_code | external code error| null
result | refer to each API|
rate_limit_status | Number of remaining calls in current period (1 minute)

### Example of signature algorithm

* [C#](/en/example/Encryption.cs)
* [Python](/en/example/Encryption.py)
* [C++](/en/example/Encryption.cpp)
* [Go](/en/example/Encryption.go)

### Errors

`10004:error sign`

This means the signature you signed is not the same as the server signed.
You may need to visit the [How to sign](#how-to-sign) section above.

<hr>

`10002:invalid request`

```
if (timestamp < (server_time + 1000) && (server_time - timestamp) <= recv_window) {
  // process request
} else {
  // invalid request
  // You may need to check timestamp and recv_window
  // Server time can be get from the HTTP response.
}
```

<hr>

`10007:Login failed`

This means the server didn't find a parameter named api_key in your HTTP request.
