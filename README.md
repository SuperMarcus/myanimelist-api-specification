MyAnimeList Unofficial API Specification
========================================

An **unofficial** specification for [MyAnimeList.net](//myanimelist.net)'s long
undocumented APIs. GPLv3 Licensed.

## Introduction

For starters, I've only been a MyAnimeList user for a short period of time.
Nevertheless, as far as I know, MyAnimeList was and still is the largest anime
information database website, which also happens to be infamous to the developers
for its unstable APIs.

This document is intended to create a up-to-date speficiation for the private
APIs that MyAnimeList is using in its official mobile apps. The goal is to
encourage the developments of third party and (particuarly) **open source**
applications. Feel free to create a pull request / issue if you have something
related that you want to share or change.

As mentioned, the specifications here (mostly) came from the analysis of
MyAnimeList's official Android app and some popular third party applications.
It uses MAL's private (or at least unpublished) APIs and is subject to change.

## Table of Contents

0. [Introduction](#introduction)
1. [Table of Contents](#table-of-contents)
2. [Requests](#requests)
3. [Responses](#responses)
4. [Authentication and Authorization](#authentication-and-authorization)
   * [Password Grant](#password-grant)
   * [Refresh Token Grant](#refresh-token-grant)
   * [Handle Authentication Responses](#handle-authentication-responses)
5. [Requesting Resources](#requesting-resources)
   * [Paging](#paging)
   * [Fields](#fields)

## Requests

API Endpoint: `https://api.myanimelist.net/v0.21`
Client Identifier (from MAL's official Android app): `6114d00ca681b7701d1e15fe11a4987e`

> Example Request
```
GET /v0.21/anime/search?status=not_yet_aired&limit=1&offset=0&fields=alternative_titles HTTP/1.1
Host: api.myanimelist.net
Accept: application/json
User-Agent: NineAnimator/2 CFNetwork/976 Darwin/18.2.0
Authorization: Bearer <OAuth2 Token>
X-MAL-Client-ID: 6114d00ca681b7701d1e15fe11a4987e
```

>
> **Note**: Always include the `X-MAL-Client-ID` header. Currently the only
> known client id is that of the MAL's official Android app.
>

## Responses

The responses are formatted in json with content type
`application/json; charset=UTF-8`.

If the request **succeeded**, the server returns `200` with a `data` object at the
root of its response JSON object.

> Example Response
```JSON
HTTP/1.1 200 OK

{
  "data": [
    {
      "node": {
        "id": 34134,
        "title": "One Punch Man Season 2",
        "main_picture": {
          "medium": "https:\/\/myanimelist.cdn-dena.com\/images\/anime\/1797\/93459.jpg",
          "large": "https:\/\/myanimelist.cdn-dena.com\/images\/anime\/1797\/93459l.jpg"
        },
        "alternative_titles": {
          "synonyms": [ "One Punch-Man 2", "One-Punch Man 2","OPM 2" ],
          "en": "",
          "ja": "\u30ef\u30f3\u30d1\u30f3\u30de\u30f3 2"
        }
      }
    }
  ],
  "paging": {
    "next": "https:\/\/api.myanimelist.net\/v0.8\/anime\/search?offset=2&status=not_yet_aired&limit=2&fields=alternative_titles"
  }
}
```

If the request **failed**, the server respond with an HTTP error status
and returns an error message.

> Example Response
```JSON
HTTP/1.1 400 Bad Request
Content-Type: application/json; charset=UTF-8

{ "message": "invalid q", "error": "bad_request" }
```

## Authentication and Authorization

OAuth2 is used in MAL's authentication scheme. Two tokens will be returned
upon a successful authentication: the `access_token` and the `refresh_token`.

Once authenticated, all of your requests should include the header
`Authentication` with value `Bearer <access_token>`.

### Password Grant

* Authenticate with the account's username and password.

Send a url-form encoded `POST` request to `https://api.myanimelist.net/v0.21/auth/token`
with the following parameters:

| Parameter     | Value                                           |
| ------------- | ----------------------------------------------- |
| `client_id`   | **String**: `6114d00ca681b7701d1e15fe11a4987e`  |
| `grant_type`  | **String**: `password`                          |
| `password`    | **String**: The account's password              |
| `username`    | **String**: The account's username              |

> Example Request
```
POST /v0.21/auth/token HTTP/1.1
Host: api.myanimelist.net
Accept: application/json
User-Agent: NineAnimator/2 CFNetwork/976 Darwin/18.2.0
X-MAL-Client-ID: 6114d00ca681b7701d1e15fe11a4987e
Content-Type: application/x-www-form-urlencoded
Content-Length: 112

client_id=6114d00ca681b7701d1e15fe11a4987e&grant_type=password&password=xxxxxx-yyyyyy-zzzzzz&username=abcdefghij
```

### Refresh Token Grant

* Authenticate (or I guess re-authenticate) with a previously returned
  `refresh_token`

Send a url-form encoded `POST` request to `https://myanimelist.net/v1/oauth2/token`
(note this url is different from the one used in the Password Grant).

| Parameter       | Value                                              |
| --------------- | -------------------------------------------------- |
| `client_id`     | **String**: `6114d00ca681b7701d1e15fe11a4987e`     |
| `grant_type`    | **String**: `refresh_token`                        |
| `refresh_token` | **String**: The refresh token previously received  |

> Example Request
```
POST /v0.21/auth/token HTTP/1.1
Host: myanimelist.net
Accept: application/json
User-Agent: NineAnimator/2 CFNetwork/976 Darwin/18.2.0
X-MAL-Client-ID: 6114d00ca681b7701d1e15fe11a4987e
Content-Type: application/x-www-form-urlencoded
Content-Length: 88

client_id=6114d00ca681b7701d1e15fe11a4987e&grant_type=refresh_token&refresh_token=xxxxxx
```

### Handle Authentication Responses

* On a successful authentication, the server responds `200 OK` and a JSON
  object with the following entries:

| Entry              | Value                                                  |
| ------------------ | ------------------------------------------------------ |
| `token_type`       | **String**: `Bearer`                                   |
| `expires_in`       | **Integer**: The lifespan of the responded access token in seconds, after which the client needs to be reauthenticated. |
| `access_token`     | **String**: The access token that should be included in further requests from this client. |
| `refresh_token`    | **String**: The refresh token that can be used to re-authenticate the client with `Refresh Token Grant` |

You should persist the `refresh_token` in a secure location. When the `access_token`
expires after `expires_in` seconds, use the `refresh_token` to re-authenticate the
session. See [Refresh Token Grant](#refresh-token-grant).

> Example Response
```JSON
HTTP/1.1 200 OK
Content-Type: application/json; charset=UTF-8

{
  "token_type": "Bearer",
  "expires_in": 3600,
  "access_token": "xxxx",
  "refresh_token": "xxxx"
}
```

Further requests should be made with the `Authorization` header. See [Making Requests](#requests).

* On an unsuccessful authentication, the server responds an error HTTP status code
  and a JSON object with the following entries:

| Entry               | Value                                                         |
| ------------------- | ------------------------------------------------------------- |
| `error`             | **String**: The type of the error                             |
| `message`           | **String**: A human-readable description of the error.        |
| *(Optional)* `hint` | **String**: An optional message about how the error occurred. |

> Example Response
```JSON
HTTP/2 401
Content-Type: application/json; charset=UTF-8

{
  "error": "invalid_request",
  "message": "The refresh token is invalid.",
  "hint": "Cannot decrypt the refresh token"
}
```

## Requesting Resources

Resource requests are made to the endpoint `https://api.myanimelist.net/v0.21`.
* Always include the `X-MAL-Client-ID` header, or you will get an unauthorized
  error. See [Making Requests](#requests)
* There happens to be a different endpoint `https://api.myanimelist.net/v0.8`. But
  when you make a mutational request to this endpoint, the server will responds
  an outdated app error.

### Paging

Set the following GET parameters in your request url to limit the number of
resources included in the response:

| Parameter | Value |
| --------- | ----- |
| `limit`   | The maximum number of elements to be contained in the data section. |
| `offset`  | The page number that the client is requesting. |

> Example Request URL with Paging
```
https://api.myanimelist.net/v0.21/anime?q=An+Anime+Title&limit=5&offset=0
```

When the response is paged, the JSON object returned also includes a `paging`
section. The `next` entry in the `paging` section includes the full URL to the
next page.

> Exampled Response with Paging
```JSON
{
  "data": [ ... ],
  "paging": {
    "next": "https:\/\/api.myanimelist.net\/v0.21\/anime?offset=5&q=An+Anime+Title&limit=5"
  }
}
```

### Fields

When you make a request to a specific resource, you can set the `fields`
parameter in your URL query to limit the fields responded by the server.
* Includes a `fields` parameter whenever you can. This increases the load
  speed and (perhaps) decreases the load on MAL's server as well.
* The value of the `fields` parameter is a comma seperated list
  corresponding to the JSON keys that will be returned in the response.

> Example Request URL with Fields parameter
```
https://api.myanimelist.net/v0.21/anime?q=An+Anime+Title&fields=alternative_titles,media_type,my_list_status{start_date,finish_date}
```

