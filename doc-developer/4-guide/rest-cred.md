---
sidebar_label: How to set a REST credential through dashboard
sidebar_position: 6
slug: rest-cred
---
# How to set a REST credential through dashboard

## Introductions

REST type credential is a way for Galxe to pull data from your RESTful HTTPs backend, (or any HTTP RESTful endpoint that you are legally allowed to use).
It takes a single wallet address as input and outputs 0(false)/1(true) to indicate whether the wallet address is eligible. 
The workflow is

```
galxe.com                          galxe backend                                 your backend
---------------------------------------------------------------------------------------------
user_address          ---->   HTTP request based on config              ---->        Endpoint
crendential for user  <----   Expression evaluation on response.body    <---- 
```
We support GET and POST method:

* **GET**:  Endpoint, Headers and Expression.
* **POST**: Endpoint, Headers, Post Body and Expression

## HTTP response format requirement

The response of the request, for both GET and POST, **must** a JSON where values are stored under `data` field. For example,

```
// valid response 🦸
{
    "data": {
        "is_ok": true,
    }
}

// invalid response example 1 ⛔
{
    "response": {
        "is_ok": true,
    }
}

// invalid response example 2 ⛔
{
    "is_ok": true,
}
```

The data field will then be extracted and passed to the Expression for evaluation.

## GET

### Endpoint

The RESTFul style URL to which we will sent the HTTP GET request. The `$address` is the user address placeholder. You can put it on query params (`info?address=$address`) or as path variable (`/info/$address`).

The response of the request **must** a JSON string where values are stored under `data` field.

NOTE: [galxe.com](http://galxe.com) isn’t allowed.

### Headers

Optional; HTTP request header

NOTE: Cookie header isn’t allowed.

### Expression

The expression is a JavaScript (ES6) function of type signature: `(object) => int`. The object that will be passed as the parameter to the function is
the `data` object of the response.
The function should return either number 1 or 0, representing if the address is eligible for this subgraph credential. 
Behind the scenes, first, we send the request with the user's address to the endpoint, and then we will apply the function against the 'data' field of the response. 
If the returned value is 1, then user can own this credential, otherwise not.
The function must be anonymous, which means that the first line of the expression should be like `function(data) {}`, instead of `let expression = (data) => {}`.

### Example

**Credential**

Polygon OAT Holder

**Endpoint**

`https://api.covalenthq.com/v1/matic-mainnet/address/$address/collection/0x5D666F215a85B87Cb042D59662A7ecd2C8Cc44e6/`

**Headers**

`Authorization`: `Bearer YOU_API_KEY`

**Query Output**

```
{
    "data": {
        "updated_at": "2023-06-26T05:12:35.553904397Z",
        "address": "0x123",
        "collection": "0x5d666f215a85b87cb042d59662a7ecd2c8cc44e6",
        "is_spam": false,
        "items": []
    },
    "error": false,
    "error_message": null,
    "error_code": null
}
```

**Expression**

```javascript
function(data) {
  if (data.items != null && data.items.length > 0) {
    return 1
  }
  return 0
}
```

## POST

### Endpoint

The RESTFul style URL to which we will sent the HTTP POST request. Unlike the above HTTP-GET-sourced type, no placeholder of the address is allowed in the URL.

The response of the request **must** a JSON string where values are stored under `data` field.

NOTE: [galxe.com](http://galxe.com) isn’t allowed.

### Headers

Optional; HTTP request header

NOTE: Cookie header isn’t allowed.

### Post Body

As part of a `POST` request, a data payload can be sent to the server in the body of the request. We only supported JSON format post body, and `$address` must be included.

### Expression

The expression is a JavaScript (ES6) function of type signature: `(object) => int`. The object that will be passed as the parameter to the function is
the `data` object of the response.
The function should return either number 1 or 0, representing if the address is eligible for this subgraph credential. 
Behind the scenes, first, we send the request with the user's address to the endpoint, and then we will apply the function against the 'data' field of the response. 
If the returned value is 1, then user can own this credential, otherwise not.
The function must be anonymous, which means that the first line of the expression should be like `function(data) {}`, instead of `let expression = (data) => {}`.

### Example

**Credential**

Ethereum Balancer ($ETH Balance > 0)

**Endpoint**

[`https://mainnet.infura.io/v3/YOUR-API-KEY`](https://mainnet.infura.io/v3/YOUR-API-KEY)

**Headers: No header**

**Post Body**

```json
{
    "jsonrpc": "2.0",
    "method": "eth_getBalance",
    "params": [
        "$address",
        "latest"
    ],
    "id": 1
}
```

**Query Output**

```json
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": "0x7c2562030800"
}
```

**Expression**

```javascript
function(data) {
  if (data.result >= "0x0") {
    return 1
  }
  return 0
}
```