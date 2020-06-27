MyAnimeList Unofficial API Specification
========================================

An **unofficial** specification for [MyAnimeList.net](//myanimelist.net)'s long
undocumented APIs. GPLv3 Licensed.

## Introduction

For starters, I've only been a MyAnimeList user for a short period of time. Nevertheless, as far as I know, MyAnimeList was and still is the largest anime information database website, which also happens to be infamous to the developers for its unstable APIs.

This document intends to create an up-to-date specification for the private (open-beta) APIs that MyAnimeList is using in its official mobile apps. The goal is to encourage the developments of third-party and (particularly) **open source** applications.

As mentioned, the specifications here (mostly) came from the analysis of MyAnimeList's official Android app and some popular third-party applications. Note because this document uses MAL's private (or at least unpublished) APIs, everything is subject to change.

This document only discovers the anime-related APIs. But feel free to create a pull request/issue if you have something related that you want to share or change.

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
    - [User Information](#user-information)
      - [Parameters](#parameters)
      - [Response](#response)
      - [Example Flow](#example-flow)
    - [Anime Information](#anime-information)
      - [Parameters](#parameters-5)
      - [Response](#response-5)
    - [Search Anime](#search-anime)
      - [Parameters](#parameters-1)
      - [Response](#response-1)
    - [Library Entries](#library-entries)
      - [Parameters](#parameters-2)
      - [Response](#response-2)
    - [Update Entries](#update-entries)
      - [Parameters](#parameters-3)
      - [Response](#response-3)
      - [Example Flow](#example-flow-1)
    - [Remove Entries](#remove-entries)
      - [Parameters](#parameters-4)
      - [Response](#response-4)
      - [Example Flow](#example-flow-2)
    - [Referring the Current User](#referring-the-current-user)
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
    - [`UserObject`](#userobject)

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

### User Information

* **Request Path**: `/users/<user id>`
* Method: `GET`

#### Parameters

This operation doesn't require any parameters. Note `@me` can be used in place of `<user id>`. See [Referring the Current User](#referring-the-current-user).

#### Response

An [`UserObject`](#userobject).

#### Example Flow

> Example Request

```
GET https://api.myanimelist.net/v2/users/@me HTTP/2.0
authorization: Bearer <bearer token>
```

> Example Response

```
HTTP/2.0 200
content-type: application/json; charset=UTF-8

{"id":1234567,"name":"username","location":"some location","joined_at":"2010-01-01T01:11:11+00:00"}
```

---

### Anime Information

* **Request Path**: `/anime/<anime id>`
* Method: `GET`

#### Parameters

This operation supports the use of the [`fields`](#fields) parameter.

#### Response

An [`AnimeObject`](#animeobject).

---

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

---

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

---

### Update Entries

* **Request Path**: `/anime/<Anime Identifier>/my_list_status`
* Method: Either `PATCH` or `PUT`

#### Parameters

The parameters are url-form encoded in the body of the request. The parameter keys resembles those found in [`MyListStatusObject`](#myliststatusobject).

| Parameter | Value |
| --------- | ----- |
| `num_watched_episodes` | An integer denoting the number of episodes watched. Note this is different from the key in [`MyListStatusObject`](#myliststatusobject) (#1). |
| `status` | [`ListStatusEnum`](#liststatusenum) |
| `score` | An integer representing the user's rating. Value ranging from 1 to 10. Set this value to 0 to remove the user's rating. |
| `start_date` | A year-month-day string that indicates when the user started watching the entry (eg. `2020-1-1`). |
| `finish_date` | A year-month-day string that indicates when the user finished watching the entry (eg. `2020-1-1`). |

#### Response

* Root Response Object: [`MyListStatusObject`](#myliststatusobject) - The updated status object

#### Example Flow

> Example Request

```
PATCH https://api.myanimelist.net/v2/anime/34881/my_list_status HTTP/2.0
authorization: Bearer <bearer token>
content-type: application/x-www-form-urlencoded

finish_date=2020-01-1&start_date=2020-02-01&num_watched_episodes=2
```

> Example Response

```
HTTP/2.0 200
content-type: application/json; charset=UTF-8

{"status":"on_hold","score":0,"num_episodes_watched":2,"is_rewatching":false,"updated_at":"2020-01-1T20:50:00+00:00","start_date":"2020-01-01","finish_date":"2020-02-1","priority":0,"num_times_rewatched":0,"rewatch_value":0,"tags":[],"comments":""}
```

---

### Remove Entries

* **Request Path**: `/anime/<Anime Identifier>/my_list_status`
* Method: `DELETE`

#### Parameters

No parameters needed for this operation.

#### Response

An empty array `[]` for a successful removal.

#### Example Flow

> Example Request

```
DELETE https://api.myanimelist.net/v2/anime/39590/my_list_status HTTP/2.0
authorization: Bearer <bearer token>
```

> Example Response

```
HTTP/2.0 200
content-type: application/json; charset=UTF-8

[]
```

---

### Referring the Current User

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

### `UserObject`

* **`UserObject`**: `Object`
  * **`id`**: `Int` An integer id of the user.
  * **`name`**: `String`
  * **`location`**: `String`
  * **`joined_at`**: [`Date`](#date)
