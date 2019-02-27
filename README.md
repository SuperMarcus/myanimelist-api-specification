MyAnimeList Unofficial API Specification
========================================

An **unofficial** specification for [MyAnimeList.net](//myanimelist.net)'s long
undocumented APIs. GPLv3 Licensed.

## Introduction

For starters, I have only been a MyAnimeList user for a short period of time.
Nevertheless, as far as I know, MyAnimeList was and still is the largest anime
information database website, which also happens to be infamous to the developers
for its unstable APIs.

This document is intended to create a up-to-date speficiation for the private
APIs that MyAnimeList is using in its official mobile apps. The goal is to
encourage the developments of third party and (particuarly) **open source**
applications. Feel free to create a pull request if you have something related
that you want to share or change.

As mentioned, most of the specifications here come from MyAnimeList's official
Android mobile application. It is subject to change and is not guaranteed to
perform as expected.

## Requests

API Endpoint: `https://api.myanimelist.net/v0.8`

The Android app always seem to use `MAL (android, 0.11.8)` as its user agent.
Nevertheless, MyAnimeList currently does not seem to check any of the headers.

## Responses

The responses are always formatted in json with content type
`application/json; charset=UTF-8`.

If the request **succeeded**, the server returns `200` with a `data` object at the
root of its response JSON object.

> Example Request
```
GET /v0.8/anime/search?status=not_yet_aired&limit=1&offset=0&fields=alternative_titles HTTP/1.1
```

> Example Response
```JavaScript
HTTP/1.1 200 OK

{
  "data": [
    {
      "node": {
        "id":34134,
        "title":"One Punch Man Season 2",
        "main_picture":{
          "medium":"https:\/\/myanimelist.cdn-dena.com\/images\/anime\/1797\/93459.jpg",
          "large":"https:\/\/myanimelist.cdn-dena.com\/images\/anime\/1797\/93459l.jpg"
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
```JavaScript
HTTP/1.1 400 Bad Request

{ "message": "invalid q", "error": "bad_request" }
```
