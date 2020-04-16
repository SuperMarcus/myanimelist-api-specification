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
applications.

As mentioned, the specifications here (mostly) came from the analysis of
MyAnimeList's official Android app and some popular third party applications.
It uses MAL's private (or at least unpublished) APIs and is subject to change.

This document only discovers the anime-related APIs. But feel free to create
a pull request / issue if you have something related that you want to share or
change.

## Table of Contents

- [MyAnimeList Unofficial API Specification](#myanimelist-unofficial-api-specification)
  - [Introduction](#introduction)
  - [Table of Contents](#table-of-contents)
  - [Requests](#requests)
  - [Responses](#responses)
  - [Authentication and Authorization](#authentication-and-authorization)
    - [Password Grant](#password-grant)
    - [Refresh Token Grant](#refresh-token-grant)
    - [Handle Authentication Responses](#handle-authentication-responses)
  - [Requesting Resources](#requesting-resources)
    - [Paging](#paging)
    - [Fields](#fields)
  - [References](#references)
    - [Search Anime](#search-anime)
      - [Parameters](#parameters)
      - [Response](#response)
    - [Library Entries](#library-entries)
      - [Parameters](#parameters-1)
      - [Response](#response-1)
    - [Update Entries](#update-entries)
      - [Request Parameters](#request-parameters)
      - [Response](#response-2)
    - [Reference to the Current User](#reference-to-the-current-user)
  - [Response Objects](#response-objects)
    - [`AnimeObject`](#animeobject)
    - [`AlternativeTitlesObject`](#alternativetitlesobject)
    - [`BroadcastObject`](#broadcastobject)
    - [`CalendarDate`](#calendardate)
    - [`Date`](#date)
    - [`PictureObject`](#pictureobject)
    - [`SortingMethodEnum`](#sortingmethodenum)
    - [`SeasonObject`](#seasonobject)
    - [`ListStatusEnum`](#liststatusenum)
    - [`MyListStatusObject`](#myliststatusobject)

## Requests

API Endpoint: `https://api.myanimelist.net/v2`
Client Identifier (from MAL's official Android app): `6114d00ca681b7701d1e15fe11a4987e`

> Example Request
```
GET /v2/anime/search?status=not_yet_aired&limit=1&offset=0&fields=alternative_titles HTTP/1.1
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

If the request is intended for mutations (e.g. modify user library entries),
the request body is url-form encoded with content type: `application/x-www-form-urlencoded`.

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
    "next": "https:\/\/api.myanimelist.net\/v2\/anime\/search?offset=2&status=not_yet_aired&limit=2&fields=alternative_titles"
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

Send a url-form encoded `POST` request to `https://api.myanimelist.net/v2/auth/token`
with the following parameters:

| Parameter     | Value                                           |
| ------------- | ----------------------------------------------- |
| `client_id`   | **String**: `6114d00ca681b7701d1e15fe11a4987e`  |
| `grant_type`  | **String**: `password`                          |
| `password`    | **String**: The account's password              |
| `username`    | **String**: The account's username              |

> Example Request
```
POST /v2/auth/token HTTP/1.1
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
(note this url is **different** from the one used in the Password Grant).

| Parameter       | Value                                              |
| --------------- | -------------------------------------------------- |
| `client_id`     | **String**: `6114d00ca681b7701d1e15fe11a4987e`     |
| `grant_type`    | **String**: `refresh_token`                        |
| `refresh_token` | **String**: The refresh token previously received  |

> Example Request
```
POST /v1/auth/token HTTP/1.1
Host: myanimelist.net
Accept: application/json
User-Agent: NineAnimator/2 CFNetwork/976 Darwin/18.2.0
X-MAL-Client-ID: 6114d00ca681b7701d1e15fe11a4987e
Content-Type: application/x-www-form-urlencoded
Content-Length: 88

client_id=6114d00ca681b7701d1e15fe11a4987e&grant_type=refresh_token&refresh_token=xxxxxx
```

### Handle Authentication Responses

* Upon a successful authentication, the server responds `200 OK` and a JSON
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

* Upon an unsuccessful authentication, the server responds an error HTTP status code
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

Resource requests are made to the endpoint `https://api.myanimelist.net/v2`.
* Always include the `X-MAL-Client-ID` header, or you will get an unauthorized
  error. See [Making Requests](#requests)
* The old endpoint `https://api.myanimelist.net/v0.21` still works.
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
https://api.myanimelist.net/v2/anime?q=An+Anime+Title&limit=5&offset=0
```

When the response is paged, the JSON object returned also includes a `paging`
section. The `next` entry in the `paging` section includes the full URL to the
next page.

> Exampled Response with Paging
```JSON
{
  "data": [ ... ],
  "paging": {
    "next": "https:\/\/api.myanimelist.net\/v2\/anime?offset=5&q=An+Anime+Title&limit=5"
  }
}
```

### Fields

When you make a request to a specific resource, you can set the `fields`
parameter in your URL query to limit the fields responded by the server.
* Includes a `fields` parameter whenever you can. This decreases the load
  time and (perhaps) decreases the load on MAL's server as well.
* The value of the `fields` parameter is a comma seperated list
  corresponding to the JSON keys that will be returned in the response.
* If you want to includes fields in the sub-sections of the response objects,
  use `<Subsection Name>{<Subsection Entry 1>, <Subsection Entry 2>, <Subsection Entry 3>...}`.

> Example Request URL with Fields parameter
```
https://api.myanimelist.net/v2/anime?q=An+Anime+Title&fields=alternative_titles,media_type,my_list_status{start_date,finish_date}
```

## References

A list of known request paths and response objects.
* Paths are relative to the API endpoint: `https://api.myanimelist.net/v2`
* Responses are JSON encoded objects with utf8 encodings.

### Search Anime

* **Request Path**: `/anime`
* Method: `GET`, response supports paging

#### Parameters

| Parameter | Value |
| --------- | ----- |
| `q`       | **String**: Keywords to be used as search parameters |

#### Response

* Root Response Object: `Object`
  * **`data`**: `Array<AnimeObject>` - A list of [`AnimeObject`](#animeobject). The results from the search.

### Library Entries

* **Request Path**: `/users/<User Identifier>/animelist`
* Method: `GET`, response supports paging

#### Parameters

| Parameter | Value |
| --------- | ----- |
| `sort`    | **[SortingMethodEnum](#sortingmethodenum)**: The method of sorting the responsing entries. |
| `status`  | **[ListStatusEnum](#liststatusenum)**: Filter the status of the entries. |

#### Response

* Root Response Object: `Object`
  * **`data`**: `Array<AnimeObject>` - A list of [`AnimeObject`](#animeobject).

### Update Entries

* **Request Path**: `/anime/<Anime Identifier>/my_list_status`
* Method: `PUT`

#### Request Parameters

The parameters are url-form encoded in the body of the request.
* The request parameters can be one of the entries in the [`MyListStatusObject`](#myliststatusobject)

#### Response

* Root Response Object: [`MyListStatusObject`](#myliststatusobject) - The updated status object

### Reference to the Current User

You can reference to the currently authenticated user with the
`@me` placeholder in the request URL.

> Example Request URL
```
https://api.myanimelist.net/v2/users/@me/animelist
```

## Response Objects

### `AnimeObject`

An `AnimeObject` represents an anime in MAL's database.

* **`AnimeObject`**: `Object`
  * **`node`**: `Object`
    * **`alternative_titles`**: [`AlternativeTitlesObject`](#alternativetitlesobject)
    * **`average_episode_duration`**: Int - The average duration (in seconds) of the episodes.
    * **`broadcast`**: [`BroadcastObject`](#broadcastobject)
    * **`created_at`**: [`Date`](#date)
    * **`end_date`**: [`CalendarDate`](#calendardate) - The date at which the anime ended.
    * **`id`**: `Int` - The identifier of this media on MyAnimeList.
    * **`main_picture`**: [`PictureObject`](#pictureobject) - The poster artwork of the anime.
    * **`mean`**: `Double` - The mean score of this media on MyAnimeList.
    * **`media_type`**: `String` - The type of this media (e.g. `tv`).
    * **`nsfw`**: `String` - The NSFW state for this media (e.g. `white`).
    * **`num_episodes`**: `Int` - The number of episodes in this anime.
    * **`num_favorites`**: `Int` - The number of users that added this media to their favorites.
    * **`num_list_users`**: `Int` - The number of uses that added this media to their lists.
    * **`num_scoring_users`**: `Int` - (?) The number of users that voted for the scores.
    * **`popularity`**: `Int` - The popularity rankings of this anime.
    * **`rank`**: `Int` - The rankings of this anime.
    * **`start_date`**: [`CalendarDate`](#calendardate) - The date at which the anime started.
    * **`start_season`**: [`SeasonObject`](#seasonobject) - The season at which the anime started broadcasting.
    * **`status`**: `String` - An enumeration representing the broadcasting status of the anime (E.g. `finished_airing`).
    * **`synopsis`**: `String` - The synopsis of the anime.
    * **`title`**: `String` - The canonical (?) title of the anime.
    * **`updated_at`**: [`Date`](#date) - The last time that the information is updated on MyAnimeList.
    * **`my_list_status`**: [`MyListStatusObject`](#myliststatusobject)
    * **`background`**: `String` - Background story of the anime
    * **`related_anime`**: `Array<AnimeObject>` - A list of anime related to this anime

### `AlternativeTitlesObject`

The set of titles and synonyms of the anime.

* **`AlternativeTitlesObject`**: `Object`
  * **`en`**: `String` - The English title of the media
  * **`ja`**: `String` - The original (native) name of the media
  * **`synonyms`**: `Array<String>` - A list of synonyms of the media

### `BroadcastObject`

The broadcasting schedule of the anime.

* **`BroadcastObject`**: `Object`
  * **`day_of_the_week`**: `String` - A lower-cased string of the day that the media is released in a week. E.g. `thursday`
  * **`start_time`**: `String` - The time of the broadcast. E.g. `19:30`

### `CalendarDate`

A simple date formatted as `yyyy-mm-dd` (e.g. `2007-02-08`).

* **`CalendarDate`**: `String`

### `Date`

A specific time.

* **`Date`**: `String` - A [`ISO 8601`](https://en.wikipedia.org/wiki/ISO_8601) formatted time string.

### `PictureObject`

A set of pictures.

* **`PictureObject`**: `Object`
  * **`large`**: `String` - An absulute URL to the high(er) resolution picture
  * **`medium`**: `String` - An absulute URL to the medium resolution picture

### `SortingMethodEnum`

The method of sorting the entries in the response lists.

* **`SortingMethodEnum`**: `String` - One of `anime_title`, `list_score`, `list_updated_at`, `anime_start_date`

### `SeasonObject`

Representing a season in a year.

* **`SeasonObject`**: `Object`
  * **`season`**: `String` - The season in a year (E.g. `fall`).
  * **`year`**: `Int` - The four digits integer representation of a year (E.g. `2002`).

### `ListStatusEnum`

The list status of the user.

* **`ListStatusEnum`**: `String` - One of `watching`, `completed`, `on_hold`, `dropped`, `plan_to_watch`

### `MyListStatusObject`

The library entry.

* **`MyListStatusObject`**: `Object`
  * **`comments`**: `String`
  * **`is_rewatching`**: `Bool`
  * **`num_episodes_watched`**: `Int`
  * **`num_times_rewatched`**: `Int`
  * **`priority`**: `Int`
  * **`rewatch_value`**: `Int`
  * **`score`**: `Double`
  * **`status`**: **[`ListStatusEnum`](#liststatusenum)**
  * **`tags`**: ?
  * **`updated_at`**: [`Date`](#date)
