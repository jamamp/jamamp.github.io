---
title: Floatplane REST API v3.10.0-c
language_tabs:
  - shell: Shell
  - http: HTTP
  - javascript: JavaScript
  - ruby: Ruby
  - python: Python
  - php: PHP
  - java: Java
  - go: Go
toc_footers: []
includes: []
search: true
highlight_theme: darkula
headingLevel: 2

---

<!-- Generator: Widdershins v4.0.1 -->

<h1 id="floatplane-rest-api">Floatplane REST API v3.10.0-c</h1>

> Scroll down for code samples, example requests and responses. Select a language for code samples from the tabs above or the mobile navigation menu.

Homepage: [https://jman012.github.io/FloatplaneAPIDocs](https://jman012.github.io/FloatplaneAPIDocs)

This document describes the REST API layer of [https://www.floatplane.com](https://www.floatplane.com), a content creation and video streaming website created by Floatplane Media Inc. and Linus Media Group, where users can support their favorite creators via paid subscriptions in order to watch their video and livestream content in higher quality, in addition to other perks.

While this document contains stubs for all of the Floatplane APIs for this version, many are not filled out because they are related only to content creation, moderation, or administration and are not needed for regular use. These have "TODO" as the description, and are automatically removed before document generation. If you are viewing the "Trimmed" version of this document, they have been removed for brevity.

## API Object Organization

- **Users** and **Creators** exist on Floatplane at the highest level
	- The highest-level object in Floatplane is the Creator. This is an entity, such as Linus Tech Tips, that produces media for Users.
- A Creator owns one or more **Subscription Plans**
- A User can view a Creator's Content if they are subscribed to them
- A Creator publishes **Content**, in the form of **Blog Posts**
	- Content is produced by Creators, and show up for subscribed Users to view when it is released. A piece of Content is meant to be generic, and may contain different types of sub-Content. Currently, the only type is a Blog Post.
	- A Blog Post is the main type of Content that a Creator produces. Blog Posts are how a Creator can share text and/or media attachments with their subscribers.
- A Blog Post is comprised of one or more of: video, audio, or picture **Attachments**
	- A media Attachment may be: video, audio, picture. Attachments are a part of Blog Posts, and are in a particular order.
- A Creator may also have a single **Livestream**
- Creators also may have one or more **Channels**

## API Flow

As of Floatplane version 4.0.13, these are the recommended endpoints to use for normal operations.

1. Login
	1. `/api/v3/auth/captcha/info` - Get captcha information
	1. `/api/v2/auth/login` - Login with username, password, and optional captcha token
	1. `/api/v2/auth/checkFor2faLogin` - Optionally provide 2FA token to complete login
	1. `/api/v2/auth/logout` - Logout at a later point in time
1. Home page
	1. `/api/v3/user/subscriptions` - Get the user's active subscriptions
    1. `/api/v3/creator/info` - Get more information on subscribed creators
        1. Shows a list of creators that the user can select
		1. Note that this can search and return multiple creators. The V3 version only works for a single creator at a time.
	1. `/api/v3/content/creator/list` - Using the subscriptions, show a home page with content from all subscriptions/subscribed creators
		1. Supply all creator identifiers from the subscriptions
		1. This is be paginated
1. Creator page
	1. `/api/v3/creator/info` - Get more details for the creator to display, including if livestreams are available
	1. `/api/v3/content/creator` - Show recent content by that creator (as opposed to all subscribed creators, above)
	1. `/api/v2/plan/info` - Show available plans the user can subscribe to for the creator
1. Content page
	1. `/api/v3/content/post` - Show more detailed information about a piece of content, including text description, available attachments, metadata, interactions, etc.
	1. `/api/v3/content/related` - List some related content for the user to watch next
	1. `/api/v3/comment` - Load comments for the content for the user to read
		1. There are several more comment APIs to post, like, dislike, etc.
	1. `/api/v2/user/ban/status` - Determine if the user is banned from this creator
	1. `/api/v3/content/{video|audio|picture|gallery}` - Load the attached media for the post. This is usually video, but audio, pictures, and galleries are also available.
	1. `/api/v3/delivery/info` - For video and audio, this is required to get the information to stream or download the content in media players
1. Livestream
	1. `/api/v3/delivery/info` - Using the type "livestream" to load the livestream media in a media player
	1. `wss://chat.floatplane.com/sails.io/?...` - To connect to the livestream chat over websocket. See https://jman012.github.io/FloatplaneAPIDocs/ for more information on the FP Async API with Websockets.
1. User Profile
	1. `/api/v3/user/self` - Display username, name, email, and profile pictures

## API Organization

The organization of APIs into categories in this document are reflected from the internal organization of the Floatplane website bundled code, from `frontend.floatplane.com/{version}/main.js`. This is in order to use the best organization from the original developers' point of view.

For instance, Floatplane's authentication endpoints are organized into `Auth.v2.login(...)`, `Auth.v2.logout()`, and `Auth.v3.getCaptchaInfo()`. A limitation in OpenAPI is the lack of nested tagging/structure, so this document splits `Auth` into `AuthV2` and `AuthV3` to emulate the nested structure.

## Rate Limiting

The Floatplane API may employ rate limiting on certain or all endpoints. If too many requests are sent by a client to the API, it will be rejected and rate-limited. This may be by IP address per endpoint in a certain unit of time, but is subject to change.

Rate-limited requests will respond with an HTTP 429 response. The content of the response may be HTML or JSON and is subject to change. The response will also have a `Retry-After` header, which contains the number of seconds remaining until the rate limiting will cease for the client on that endpoint. 

Clients are expected to both 1) prevent too many requests from executing at a time, usually for specific endpoints, and particulay for the `/api/v2/cdn/delivery` and `/api/v3/delivery/info` endpoints, and 2) properly handle rate-limited responses by ceasing requests until the `Retry-After` expiration.

## Notes

Note that the Floatplane API does support the use of [ETags](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag) for retrieving some information, such as retrieving information about creators, users, etc. Expect an HTTP 304 if the content has not changed, and to re-use cached responses. This is useful to ease the strain on Floatplane's API server.

The date-time format used by Floatplane API is not standard ISO 8601 format. The dates/times given by Floatplane include milliseconds. Depending on your code generator, you may need to override the date-time format to something similar to `yyyy-MM-dd'T'HH:mm:ss.SSSZ`, for both encoding and decoding.

Base URLs:

* <a href="https://www.floatplane.com">https://www.floatplane.com</a>

Web: <a href="https://github.com/Jman012/FloatplaneAPI/">James Linnell</a> 
License: <a href="https://github.com/Jman012/FloatplaneAPI/blob/main/LICENSE">MIT</a>

# Authentication

* API Key (CookieAuth)
    - Parameter Name: **sails.sid**, in: cookie. Authentication and authorization to Floatplane API calls is governed by the `sails.sid` HTTP Cookie. To obtain this cookie, you must authenticate to `/api/v2/auth/login`.

When dealing with cookies in native applications (not via a website inside of a browser), please keep in mind that some languages/libraries may keep cookies across requests by default, and some may not. For instance, in Swift the `URLSession.shared` object with default configuration will automatically track and persist cookies across requests.

<h1 id="floatplane-rest-api-authv2">AuthV2</h1>

Sign up, login, 2FA, and logout. Additionally, login spoofing for administrators.

## login

<a id="opIdlogin"></a>

> Code samples

```shell
# You can also use wget
curl -X POST https://www.floatplane.com/api/v2/auth/login \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json'

```

```http
POST https://www.floatplane.com/api/v2/auth/login HTTP/1.1
Host: www.floatplane.com
Content-Type: application/json
Accept: application/json

```

```javascript
const inputBody = '{
  "username": "string",
  "password": "string",
  "captchaToken": "string"
}';
const headers = {
  'Content-Type':'application/json',
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v2/auth/login',
{
  method: 'POST',
  body: inputBody,
  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Content-Type' => 'application/json',
  'Accept' => 'application/json'
}

result = RestClient.post 'https://www.floatplane.com/api/v2/auth/login',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Content-Type': 'application/json',
  'Accept': 'application/json'
}

r = requests.post('https://www.floatplane.com/api/v2/auth/login', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Content-Type' => 'application/json',
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('POST','https://www.floatplane.com/api/v2/auth/login', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v2/auth/login");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("POST");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Content-Type": []string{"application/json"},
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("POST", "https://www.floatplane.com/api/v2/auth/login", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`POST /api/v2/auth/login`

*Login*

Login to Floatplane with the provided username and password, retrieving the authentication/authorization cookie from the response for subsequent requests.

> Body parameter

```json
{
  "username": "string",
  "password": "string",
  "captchaToken": "string"
}
```

<h3 id="login-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|[AuthLoginV2Request](#schemaauthloginv2request)|true|none|

> Example responses

> 200 Response

```json
{
  "user": {
    "id": "0123456789abcdef01234567",
    "username": "my_username",
    "profileImage": {
      "width": 512,
      "height": 512,
      "path": "https://pbs.floatplane.com/profile_images/default/user12.png",
      "childImages": [
        {
          "width": 250,
          "height": 250,
          "path": "https://pbs.floatplane.com/profile_images/default/user12_250x250.png"
        },
        {
          "width": 100,
          "height": 100,
          "path": "https://pbs.floatplane.com/profile_images/default/user12_100x100.png"
        }
      ]
    }
  },
  "needs2FA": false
}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "string",
  "errors": [
    {
      "id": "string",
      "name": "string",
      "message": "string",
      "data": {}
    }
  ],
  "message": "string"
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="login-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Returns the header and information about the logged-in user, including the id, username, and profile image.|[AuthLoginV2Response](#schemaauthloginv2response)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The login attempt failed, either due to a bad username or password.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="login-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|200|Set-Cookie|string||Contains the cookie used in subsequent authenticated requests.|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="success">
This operation does not require authentication
</aside>

## logout

<a id="opIdlogout"></a>

> Code samples

```shell
# You can also use wget
curl -X POST https://www.floatplane.com/api/v2/auth/logout \
  -H 'Accept: text/plain'

```

```http
POST https://www.floatplane.com/api/v2/auth/logout HTTP/1.1
Host: www.floatplane.com
Accept: text/plain

```

```javascript

const headers = {
  'Accept':'text/plain'
};

fetch('https://www.floatplane.com/api/v2/auth/logout',
{
  method: 'POST',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'text/plain'
}

result = RestClient.post 'https://www.floatplane.com/api/v2/auth/logout',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'text/plain'
}

r = requests.post('https://www.floatplane.com/api/v2/auth/logout', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'text/plain',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('POST','https://www.floatplane.com/api/v2/auth/logout', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v2/auth/logout");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("POST");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"text/plain"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("POST", "https://www.floatplane.com/api/v2/auth/logout", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`POST /api/v2/auth/logout`

*Logout*

Log out of Floatplane, invalidating the authentication/authorization cookie.

> Example responses

> 200 Response

```
"OK"
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="logout-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|string|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="logout-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|200|Set-Cookie|string||Obtain a new authentication/authorization cookie after logging out. This new cookie will not be authenticated to perform subsequent requests.|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## checkFor2faLogin

<a id="opIdcheckFor2faLogin"></a>

> Code samples

```shell
# You can also use wget
curl -X POST https://www.floatplane.com/api/v2/auth/checkFor2faLogin \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json'

```

```http
POST https://www.floatplane.com/api/v2/auth/checkFor2faLogin HTTP/1.1
Host: www.floatplane.com
Content-Type: application/json
Accept: application/json

```

```javascript
const inputBody = '{
  "token": "string"
}';
const headers = {
  'Content-Type':'application/json',
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v2/auth/checkFor2faLogin',
{
  method: 'POST',
  body: inputBody,
  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Content-Type' => 'application/json',
  'Accept' => 'application/json'
}

result = RestClient.post 'https://www.floatplane.com/api/v2/auth/checkFor2faLogin',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Content-Type': 'application/json',
  'Accept': 'application/json'
}

r = requests.post('https://www.floatplane.com/api/v2/auth/checkFor2faLogin', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Content-Type' => 'application/json',
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('POST','https://www.floatplane.com/api/v2/auth/checkFor2faLogin', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v2/auth/checkFor2faLogin");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("POST");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Content-Type": []string{"application/json"},
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("POST", "https://www.floatplane.com/api/v2/auth/checkFor2faLogin", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`POST /api/v2/auth/checkFor2faLogin`

*Check For 2FA Login*

Complete the login process if a two-factor authentication token is required from the beginning of the login process.

> Body parameter

```json
{
  "token": "string"
}
```

<h3 id="checkfor2falogin-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|[CheckFor2faLoginRequest](#schemacheckfor2faloginrequest)|true|none|

> Example responses

> 200 Response

```json
{
  "user": {
    "id": "0123456789abcdef01234567",
    "username": "my_username",
    "profileImage": {
      "width": 512,
      "height": 512,
      "path": "https://pbs.floatplane.com/profile_images/default/user12.png",
      "childImages": [
        {
          "width": 250,
          "height": 250,
          "path": "https://pbs.floatplane.com/profile_images/default/user12_250x250.png"
        },
        {
          "width": 100,
          "height": 100,
          "path": "https://pbs.floatplane.com/profile_images/default/user12_100x100.png"
        }
      ]
    }
  },
  "needs2FA": false
}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "string",
  "errors": [
    {
      "id": "string",
      "name": "string",
      "message": "string",
      "data": {}
    }
  ],
  "message": "string"
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="checkfor2falogin-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Returns the header and information about the logged-in user, including the id, username, and profile image.|[AuthLoginV2Response](#schemaauthloginv2response)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The login attempt failed, either due to a bad username or password.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="checkfor2falogin-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|200|Set-Cookie|string||Contains the cookie used in subsequent authenticated requests.|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

<h1 id="floatplane-rest-api-authv3">AuthV3</h1>

Captchas information.

## getCaptchaInfo

<a id="opIdgetCaptchaInfo"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v3/auth/captcha/info \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v3/auth/captcha/info HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/auth/captcha/info',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v3/auth/captcha/info',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v3/auth/captcha/info', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v3/auth/captcha/info', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/auth/captcha/info");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v3/auth/captcha/info", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v3/auth/captcha/info`

*Get Captcha Info*

Gets the site keys used for Google Recaptcha V2 and V3. These are useful when providing a captcha token when logging in or signing up.

> Example responses

> 200 Response

```json
{
  "v2": {
    "variants": {
      "android": {
        "siteKey": "..."
      },
      "checkbox": {
        "siteKey": "..."
      },
      "invisible": {
        "siteKey": "..."
      }
    }
  },
  "v3": {
    "variants": {
      "invisible": {
        "siteKey": "..."
      }
    }
  }
}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getcaptchainfo-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|[GetCaptchaInfoResponse](#schemagetcaptchainforesponse)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getcaptchainfo-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="success">
This operation does not require authentication
</aside>

<h1 id="floatplane-rest-api-cdnv2">CDNV2</h1>

Content Delivery mechanisms for Floatplane media.

## getDeliveryInfo

<a id="opIdgetDeliveryInfo"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v2/cdn/delivery?type=vod \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v2/cdn/delivery?type=vod HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v2/cdn/delivery?type=vod',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v2/cdn/delivery',
  params: {
  'type' => 'string'
}, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v2/cdn/delivery', params={
  'type': 'vod'
}, headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v2/cdn/delivery', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v2/cdn/delivery?type=vod");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v2/cdn/delivery", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v2/cdn/delivery`

*Get Delivery Info*

Given an video/audio attachment identifier, retrieves the information necessary to play, download, or livestream the video/audio at various quality levels.

<h3 id="getdeliveryinfo-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|type|query|string|true|Used to determine which kind of retrieval method is requested for the video.|
|guid|query|string|false|The GUID of the attachment for a post, retrievable from the `videoAttachments` or `audioAttachments` object. Required when `type` is `vod`, `aod`, or `download`. Note: either this or `creator` must be supplied.|
|creator|query|string|false|The GUID of the creator for a livestream, retrievable from `CreatorModelV2.id`. Required when `type` is `live`. Note: either this or `guid` must be supplied. Note: for `vod` and `download`, including this `creator` parameter *will* cause an error to be returned.|

#### Detailed descriptions

**type**: Used to determine which kind of retrieval method is requested for the video.

- VOD = stream a Video On Demand
- AOD = stream Audio On Demand
- Live = Livestream the content
- Download = Download the content for the user to play later.

#### Enumerated Values

|Parameter|Value|
|---|---|
|type|vod|
|type|aod|
|type|live|
|type|download|

> Example responses

> 200 Response

```json
{
  "cdn": "https://cdn-vod-drm2.floatplane.com",
  "strategy": "cdn",
  "resource": {
    "uri": "/Videos/TViGzkuIic/{qualityLevels}.mp4/chunk.m3u8?token={qualityLevelParams.token}",
    "data": {
      "qualityLevels": [
        {
          "name": "360",
          "width": 640,
          "height": 360,
          "label": "360p",
          "order": 0
        },
        {
          "name": "480",
          "width": 854,
          "height": 480,
          "label": "480p",
          "order": 1
        },
        {
          "name": "720",
          "width": 1280,
          "height": 720,
          "label": "720p",
          "order": 2
        },
        {
          "name": "1080",
          "width": 2160,
          "height": 1080,
          "label": "1080p",
          "order": 4
        }
      ],
      "qualityLevelParams": {
        "360": {
          "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyZXNzb3VyY2VQYXRoIjoiL1ZpZGVvcy9UVmlHemt1SWljLzM2MC5tcDQvY2h1bmsubTN1OCIsInVzZXJJZCI6IjAxMjM0NTY3ODlhYmNkZWYwMTIzNDU2NyIsImlhdCI6MTYzMzc5NzMxMSwiZXhwIjoxNjMzODE4OTExfQ.uaLzZ4wSc0jrYbjkdhuF4_UY92iWQsq2efrWUutYUvQ"
        },
        "480": {
          "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyZXNzb3VyY2VQYXRoIjoiL1ZpZGVvcy9UVmlHemt1SWljLzQ4MC5tcDQvY2h1bmsubTN1OCIsInVzZXJJZCI6IjAxMjM0NTY3ODlhYmNkZWYwMTIzNDU2NyIsImlhdCI6MTYzMzc5NzMxMSwiZXhwIjoxNjMzODE4OTExfQ.O6PHCJKcLW7ohuKj6UcMa8QGoN-vZr6xTtfXsUMRki0"
        },
        "720": {
          "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyZXNzb3VyY2VQYXRoIjoiL1ZpZGVvcy9UVmlHemt1SWljLzcyMC5tcDQvY2h1bmsubTN1OCIsInVzZXJJZCI6IjAxMjM0NTY3ODlhYmNkZWYwMTIzNDU2NyIsImlhdCI6MTYzMzc5NzMxMSwiZXhwIjoxNjMzODE4OTExfQ.lbOTTBXBjA-i9gBzm8ydFQ8fa8q07Z2vaLsYMKUp4Ik"
        },
        "1080": {
          "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyZXNzb3VyY2VQYXRoIjoiL1ZpZGVvcy9UVmlHemt1SWljLzEwODAubXA0L2NodW5rLm0zdTgiLCJ1c2VySWQiOiIwMTIzNDU2Nzg5YWJjZGVmMDEyMzQ1NjciLCJpYXQiOjE2MzM3OTczMTEsImV4cCI6MTYzMzgxODkxMX0.E-bw_gnUzKUpYeL2l-kTmj5CbwmDb519ohjf5LlLyQg"
        }
      }
    }
  }
}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getdeliveryinfo-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Information on how to stream or download the requested video from the CDN in various levels of quality.|[CdnDeliveryV2Response](#schemacdndeliveryv2response)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getdeliveryinfo-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

<h1 id="floatplane-rest-api-deliveryv3">DeliveryV3</h1>

Content Delivery mechanisms for Floatplane media.

## getDeliveryInfoV3

<a id="opIdgetDeliveryInfoV3"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v3/delivery/info?scenario=onDemand&entityId=string \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v3/delivery/info?scenario=onDemand&entityId=string HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/delivery/info?scenario=onDemand&entityId=string',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v3/delivery/info',
  params: {
  'scenario' => 'string',
'entityId' => 'string'
}, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v3/delivery/info', params={
  'scenario': 'onDemand',  'entityId': 'string'
}, headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v3/delivery/info', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/delivery/info?scenario=onDemand&entityId=string");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v3/delivery/info", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v3/delivery/info`

*Get Delivery Info*

Given an video/audio attachment or livestream identifier, retrieves the information necessary to play, download, or livestream the media at various quality levels.

<h3 id="getdeliveryinfov3-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|scenario|query|string|true|Used to determine the scenario in which to consume the media.|
|entityId|query|string|true|The attachment or livestream identifier for the requested media. For video and audio, this would be from the `videoAttachments` or `audioAttachments` objects. For livestreams, this is the `liveStream.id` from the creator object.|
|outputKind|query|string|false|Use `outputKind` to ensure the right vehicle is used for your client, e.g. `outputKind=hls.fmp4` is optimal for tvOS 10+.|

#### Detailed descriptions

**scenario**: Used to determine the scenario in which to consume the media.

- `onDemand` = stream a Video/Audio On Demand
- `download` = Download the content for the user to play later.
- `live` = Livestream the content

#### Enumerated Values

|Parameter|Value|
|---|---|
|scenario|onDemand|
|scenario|download|
|scenario|live|
|outputKind|hls.mpegts|
|outputKind|hls.fmp4|
|outputKind|dash.mpegts|
|outputKind|dash.m4s|
|outputKind|flat|

> Example responses

> https://www.floatplane.com/api/v3/delivery/info?entityId=Lug14XXgLx&scenario=onDemand 

Note that tokens are JWT tokens, but have been replaced with `<token>` in the examples below.

```json
{
  "groups": [
    {
      "origins": [
        {
          "url": "https://cdn-vod-drm2.floatplane.com"
        }
      ],
      "variants": [
        {
          "name": "360-avc1",
          "label": "360p",
          "url": "/Videos/Lug14XXgLx/360.mp4/chunk.m3u8?token=<token>",
          "mimeType": "application/x-mpegURL",
          "order": 11016384,
          "hidden": false,
          "enabled": true,
          "meta": {
            "video": {
              "codec": "avc1.64001e",
              "codecSimple": "avc1",
              "bitrate": {
                "average": 255886
              },
              "width": 640,
              "height": 320,
              "isHdr": false,
              "fps": 29.97,
              "mimeType": "video/MP2T"
            },
            "audio": {
              "codec": "mp4a.40.2",
              "bitrate": {
                "average": 93340,
                "maximum": 93340
              },
              "channelCount": 2,
              "samplerate": 48000,
              "mimeType": "video/MP2T"
            }
          }
        },
        {
          "name": "480-avc1",
          "label": "480p",
          "url": "/Videos/Lug14XXgLx/480.mp4/chunk.m3u8?token=<token>",
          "mimeType": "application/x-mpegURL",
          "order": 19339456,
          "hidden": false,
          "enabled": true,
          "meta": {
            "video": {
              "codec": "avc1.64001f",
              "codecSimple": "avc1",
              "bitrate": {
                "average": 395615
              },
              "width": 848,
              "height": 424,
              "isHdr": false,
              "fps": 29.97,
              "mimeType": "video/MP2T"
            },
            "audio": {
              "codec": "mp4a.40.2",
              "bitrate": {
                "average": 93340,
                "maximum": 93340
              },
              "channelCount": 2,
              "samplerate": 48000,
              "mimeType": "video/MP2T"
            }
          }
        },
        {
          "name": "720-avc1",
          "label": "720p",
          "url": "/Videos/Lug14XXgLx/720.mp4/chunk.m3u8?token=<token>",
          "mimeType": "application/x-mpegURL",
          "order": 44112064,
          "hidden": false,
          "enabled": true,
          "meta": {
            "video": {
              "codec": "avc1.640020",
              "codecSimple": "avc1",
              "bitrate": {
                "average": 736634
              },
              "width": 1280,
              "height": 640,
              "isHdr": false,
              "fps": 29.97,
              "mimeType": "video/MP2T"
            },
            "audio": {
              "codec": "mp4a.40.2",
              "bitrate": {
                "average": 93340,
                "maximum": 93340
              },
              "channelCount": 2,
              "samplerate": 48000,
              "mimeType": "video/MP2T"
            }
          }
        },
        {
          "name": "1080-avc1",
          "label": "1080p",
          "url": "/Videos/Lug14XXgLx/1080.mp4/chunk.m3u8?token=<token>",
          "mimeType": "application/x-mpegURL",
          "order": 99293376,
          "hidden": false,
          "enabled": true,
          "meta": {
            "video": {
              "codec": "avc1.64002a",
              "codecSimple": "avc1",
              "bitrate": {
                "average": 1448573
              },
              "width": 1920,
              "height": 960,
              "isHdr": false,
              "fps": 29.97,
              "mimeType": "video/MP2T"
            },
            "audio": {
              "codec": "mp4a.40.2",
              "bitrate": {
                "average": 93340,
                "maximum": 93340
              },
              "channelCount": 2,
              "samplerate": 48000,
              "mimeType": "video/MP2T"
            }
          }
        },
        {
          "name": "2160-avc1",
          "label": "4K",
          "url": "/Videos/Lug14XXgLx/2160.mp4/chunk.m3u8?token=<token>",
          "mimeType": "application/x-mpegURL",
          "order": 397351104,
          "hidden": false,
          "enabled": true,
          "meta": {
            "video": {
              "codec": "avc1.640034",
              "codecSimple": "avc1",
              "bitrate": {
                "average": 5904212
              },
              "width": 3840,
              "height": 1920,
              "isHdr": false,
              "fps": 29.97,
              "mimeType": "video/MP2T"
            },
            "audio": {
              "codec": "mp4a.40.2",
              "bitrate": {
                "average": 93340,
                "maximum": 93340
              },
              "channelCount": 2,
              "samplerate": 48000,
              "mimeType": "video/MP2T"
            }
          }
        }
      ]
    }
  ]
}
```

> https://www.floatplane.com/api/v3/delivery/info?entityId=Lug14XXgLx&scenario=download 

Note that tokens are JWT tokens, but have been replaced with `<token>` in the examples below.

```json
{
  "groups": [
    {
      "origins": [
        {
          "url": "https://edge01-na.floatplane.com",
          "queryUrl": "https://edge01-na-query.floatplane.com",
          "datacenter": {
            "latitude": 45.3168,
            "longitude": -73.8659,
            "countryCode": "CA",
            "regionCode": "QC"
          }
        },
        {
          "url": "https://edge02-na.floatplane.com",
          "queryUrl": "https://edge02-na-query.floatplane.com",
          "datacenter": {
            "latitude": 45.3168,
            "longitude": -73.8659,
            "countryCode": "CA",
            "regionCode": "QC"
          }
        }
      ],
      "variants": [
        {
          "name": "360-avc1",
          "label": "360p",
          "url": "/Videos/Lug14XXgLx/360.mp4?token=<token>",
          "mimeType": "video/mp4",
          "order": 11016384,
          "hidden": false,
          "enabled": true,
          "meta": {
            "video": {
              "codec": "avc1.64001e",
              "codecSimple": "avc1",
              "bitrate": {
                "average": 255886
              },
              "width": 640,
              "height": 320,
              "isHdr": false,
              "fps": 29.97
            },
            "audio": {
              "codec": "mp4a.40.2",
              "bitrate": {
                "average": 93340,
                "maximum": 93340
              },
              "channelCount": 2,
              "samplerate": 48000
            }
          }
        },
        {
          "name": "480-avc1",
          "label": "480p",
          "url": "/Videos/Lug14XXgLx/480.mp4?token=<token>",
          "mimeType": "video/mp4",
          "order": 19339456,
          "hidden": false,
          "enabled": true,
          "meta": {
            "video": {
              "codec": "avc1.64001f",
              "codecSimple": "avc1",
              "bitrate": {
                "average": 395615
              },
              "width": 848,
              "height": 424,
              "isHdr": false,
              "fps": 29.97
            },
            "audio": {
              "codec": "mp4a.40.2",
              "bitrate": {
                "average": 93340,
                "maximum": 93340
              },
              "channelCount": 2,
              "samplerate": 48000
            }
          }
        },
        {
          "name": "720-avc1",
          "label": "720p",
          "url": "/Videos/Lug14XXgLx/720.mp4?token=<token>",
          "mimeType": "video/mp4",
          "order": 44112064,
          "hidden": false,
          "enabled": true,
          "meta": {
            "video": {
              "codec": "avc1.640020",
              "codecSimple": "avc1",
              "bitrate": {
                "average": 736634
              },
              "width": 1280,
              "height": 640,
              "isHdr": false,
              "fps": 29.97
            },
            "audio": {
              "codec": "mp4a.40.2",
              "bitrate": {
                "average": 93340,
                "maximum": 93340
              },
              "channelCount": 2,
              "samplerate": 48000
            }
          }
        },
        {
          "name": "1080-avc1",
          "label": "1080p",
          "url": "/Videos/Lug14XXgLx/1080.mp4?token=<token>",
          "mimeType": "video/mp4",
          "order": 99293376,
          "hidden": false,
          "enabled": true,
          "meta": {
            "video": {
              "codec": "avc1.64002a",
              "codecSimple": "avc1",
              "bitrate": {
                "average": 1448573
              },
              "width": 1920,
              "height": 960,
              "isHdr": false,
              "fps": 29.97
            },
            "audio": {
              "codec": "mp4a.40.2",
              "bitrate": {
                "average": 93340,
                "maximum": 93340
              },
              "channelCount": 2,
              "samplerate": 48000
            }
          }
        },
        {
          "name": "2160-avc1",
          "label": "4K",
          "url": "/Videos/Lug14XXgLx/2160.mp4?token=<token>",
          "mimeType": "video/mp4",
          "order": 397351104,
          "hidden": false,
          "enabled": true,
          "meta": {
            "video": {
              "codec": "avc1.640034",
              "codecSimple": "avc1",
              "bitrate": {
                "average": 5904212
              },
              "width": 3840,
              "height": 1920,
              "isHdr": false,
              "fps": 29.97
            },
            "audio": {
              "codec": "mp4a.40.2",
              "bitrate": {
                "average": 93340,
                "maximum": 93340
              },
              "channelCount": 2,
              "samplerate": 48000
            }
          }
        }
      ]
    }
  ]
}
```

> https://www.floatplane.com/api/v3/delivery/info?entityId=5c13f3c006f1be15e08e05c0&scenario=live 

Note that tokens are JWT tokens, but have been replaced with `<token>` in the examples below.

```json
{
  "groups": [
    {
      "origins": [
        {
          "url": "https://de488bcb61af.us-east-1.playback.live-video.net"
        }
      ],
      "variants": [
        {
          "name": "live-abr",
          "label": "Auto",
          "url": "/api/video/v1/us-east-1.758417551536.channel.yKkxur4ukc0B.m3u8?allow_source=false&token=<token>",
          "mimeType": "application/x-mpegURL",
          "hidden": false,
          "enabled": true,
          "meta": {
            "live": {
              "lowLatencyExtension": "ivshls"
            }
          }
        }
      ]
    }
  ]
}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getdeliveryinfov3-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Information on how to stream or download the requested video from the CDN in various levels of quality.|[CdnDeliveryV3Response](#schemacdndeliveryv3response)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getdeliveryinfov3-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

<h1 id="floatplane-rest-api-connectedaccountsv2">ConnectedAccountsV2</h1>

3rd party account management, such as Discord or LTT Forums.

## listConnections

<a id="opIdlistConnections"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v2/connect/list \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v2/connect/list HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v2/connect/list',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v2/connect/list',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v2/connect/list', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v2/connect/list', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v2/connect/list");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v2/connect/list", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v2/connect/list`

*List Connections*

List the available 3rd party accounts for the user's profile.

> Example responses

> 200 Response

```json
[
  {
    "key": "ltt",
    "name": "LinusTechTips",
    "enabled": true,
    "iconWhite": "/images/connections/ltt/white@2x.png",
    "connectedAccount": null,
    "connected": false,
    "isAccountProvider": false
  },
  {
    "key": "discord",
    "name": "Discord",
    "enabled": true,
    "iconWhite": "/images/connections/discord/white@2x.png",
    "connectedAccount": {
      "id": "9a8a4140fdd2b0c15b54333a",
      "remoteUserId": "012345678912345678",
      "remoteUserName": "my_username#2673",
      "data": {
        "canJoinGuilds": true
      }
    },
    "connected": true,
    "isAccountProvider": false
  }
]
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="listconnections-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Returns the list of connected and available accounts.|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="listconnections-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|[[ConnectedAccountModel](#schemaconnectedaccountmodel)]|false|none|none|
|» key|string|true|none|Unique identifier for the account type.|
|» name|string|true|none|Display-friendly label for the `key`.|
|» enabled|boolean|true|none|Determines if the system allows this account to be connected to.|
|» iconWhite|string|true|none|none|
|» connectedAccount|object¦null|true|none|none|
|»» id|string|true|none|none|
|»» remoteUserId|string|true|none|none|
|»» remoteUserName|string|true|none|none|
|»» data|object¦null|true|none|none|
|»»» canJoinGuilds|boolean|true|none|none|
|» connected|boolean|true|none|If true, the user is connected and the `connectedAccount` will have data about the account.|
|» isAccountProvider|boolean|true|none|none|

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

<h1 id="floatplane-rest-api-creatorv2">CreatorV2</h1>

Get and discover creators on the platform. Creator invitation and profile management.

## getInfo

<a id="opIdgetInfo"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v2/creator/info?creatorGUID=string \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v2/creator/info?creatorGUID=string HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v2/creator/info?creatorGUID=string',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v2/creator/info',
  params: {
  'creatorGUID' => 'array[string]'
}, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v2/creator/info', params={
  'creatorGUID': [
  "string"
]
}, headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v2/creator/info', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v2/creator/info?creatorGUID=string");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v2/creator/info", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v2/creator/info`

*Get Info*

Retrieve detailed information on one or more creators on Floatplane.

<h3 id="getinfo-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|creatorGUID|query|array[string]|true|The GUID identifer(s) of the creator(s) to be retrieved.|

> Example responses

> 200 Response

```json
[
  {
    "id": "59f94c0bdd241b70349eb72b",
    "owner": "59f94c0bdd241b70349eb723",
    "title": "LinusTechTips",
    "urlname": "linustechtips",
    "description": "We make entertaining videos about technology, including tech reviews, showcases and other content.",
    "about": "# We're LinusTechTips\nWe make videos and stuff, cool eh?",
    "category": "59f94c0bdd241b70349eb727",
    "cover": {
      "width": 1990,
      "height": 519,
      "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867.jpeg",
      "childImages": [
        {
          "width": 1245,
          "height": 325,
          "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867_1245x325.jpeg"
        }
      ]
    },
    "icon": {
      "width": 600,
      "height": 600,
      "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205.jpeg",
      "childImages": [
        {
          "width": 250,
          "height": 250,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_250x250.jpeg"
        },
        {
          "width": 100,
          "height": 100,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_100x100.jpeg"
        }
      ]
    },
    "liveStream": {
      "id": "5c13f3c006f1be15e08e05c0",
      "title": "First Linux Stream",
      "description": "<p>chat on Twitch</p>",
      "thumbnail": {
        "width": 1200,
        "height": 675,
        "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412.jpeg",
        "childImages": [
          {
            "width": 400,
            "height": 225,
            "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412_400x225.jpeg"
          }
        ]
      },
      "owner": "59f94c0bdd241b70349eb72b",
      "streamPath": "/api/video/v1/us-east-1.758417551536.channel.yKkxur4ukc0B.m3u8",
      "offline": {
        "title": "Offline",
        "description": "We're offline for now – please check back later!",
        "thumbnail": {
          "width": 1920,
          "height": 1080,
          "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026.jpeg",
          "childImages": [
            {
              "width": 400,
              "height": 225,
              "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_400x225.jpeg"
            },
            {
              "width": 1200,
              "height": 675,
              "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_1200x675.jpeg"
            }
          ]
        }
      }
    },
    "subscriptionPlans": null,
    "discoverable": true,
    "subscriberCountDisplay": "total",
    "incomeDisplay": false
  }
]
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getinfo-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - The creators are found from their identifiers and returned in an array|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getinfo-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|[[CreatorModelV2](#schemacreatormodelv2)]|false|none|none|
|» id|string|true|none|none|
|» owner|string|true|none|none|
|» title|string|true|none|none|
|» urlname|string|true|none|Shown in the browser URL, and used in `/creator/named` queries.|
|» description|string|true|none|none|
|» about|string|true|none|none|
|» category|string|true|none|none|
|» cover|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»» width|integer|true|none|none|
|»» height|integer|true|none|none|
|»» path|string(uri)|true|none|none|
|»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»»» width|integer|true|none|none|
|»»» height|integer|true|none|none|
|»»» path|string(uri)|true|none|none|
|» icon|[ImageModel](#schemaimagemodel)|true|none|none|
|»» width|integer|true|none|none|
|»» height|integer|true|none|none|
|»» path|string(uri)|true|none|none|
|»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|» liveStream|[LiveStreamModel](#schemalivestreammodel)¦null|true|none|none|
|»» id|string|true|none|none|
|»» title|string|true|none|none|
|»» description|string|true|none|none|
|»» thumbnail|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»» width|integer|true|none|none|
|»»» height|integer|true|none|none|
|»»» path|string(uri)|true|none|none|
|»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»» owner|string|true|none|none|
|»» channel|string|false|none|The creator channel this livestream belongs to.|
|»» streamPath|string|true|none|none|
|»» offline|object|true|none|none|
|»»» title|string¦null|true|none|none|
|»»» description|string¦null|true|none|none|
|»»» thumbnail|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»»» width|integer|true|none|none|
|»»»» height|integer|true|none|none|
|»»»» path|string(uri)|true|none|none|
|»»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|» subscriptionPlans|[[SubscriptionPlanModel](#schemasubscriptionplanmodel)]¦null|true|none|none|
|»» id|string|true|none|none|
|»» title|string|true|none|none|
|»» description|string|true|none|none|
|»» price|string¦null|true|none|none|
|»» priceYearly|string¦null|false|none|none|
|»» currency|string|true|none|none|
|»» logo|string¦null|true|none|none|
|»» interval|string|true|none|none|
|»» featured|boolean|true|none|none|
|»» allowGrandfatheredAccess|boolean¦null|false|none|none|
|»» discordServers|[[DiscordServerModel](#schemadiscordservermodel)]|true|none|none|
|»»» id|string|true|none|none|
|»»» guildName|string|true|none|none|
|»»» guildIcon|string|true|none|none|
|»»» inviteLink|string(uri)¦null|true|none|none|
|»»» inviteMode|string|true|none|none|
|»» discordRoles|[[DiscordRoleModel](#schemadiscordrolemodel)]|true|none|none|
|»»» server|string|true|none|none|
|»»» roleName|string|true|none|none|
|» discoverable|boolean|true|none|none|
|» subscriberCountDisplay|string|true|none|none|
|» incomeDisplay|boolean|true|none|none|
|» defaultChannel|string|false|none|none|

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## getCreatorInfoByName

<a id="opIdgetCreatorInfoByName"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v2/creator/named?creatorURL=string \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v2/creator/named?creatorURL=string HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v2/creator/named?creatorURL=string',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v2/creator/named',
  params: {
  'creatorURL' => 'array[string]'
}, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v2/creator/named', params={
  'creatorURL': [
  "string"
]
}, headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v2/creator/named', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v2/creator/named?creatorURL=string");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v2/creator/named", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v2/creator/named`

*Get Info By Name*

Retrieve detailed information on one or more creators on Floatplane.

<h3 id="getcreatorinfobyname-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|creatorURL|query|array[string]|true|The string identifer(s) of the creator(s) to be retrieved.|

> Example responses

> 200 Response

```json
[
  {
    "id": "59f94c0bdd241b70349eb72b",
    "owner": "59f94c0bdd241b70349eb723",
    "title": "LinusTechTips",
    "urlname": "linustechtips",
    "description": "We make entertaining videos about technology, including tech reviews, showcases and other content.",
    "about": "# We're LinusTechTips\nWe make videos and stuff, cool eh?",
    "category": "59f94c0bdd241b70349eb727",
    "cover": {
      "width": 1990,
      "height": 519,
      "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867.jpeg",
      "childImages": [
        {
          "width": 1245,
          "height": 325,
          "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867_1245x325.jpeg"
        }
      ]
    },
    "icon": {
      "width": 600,
      "height": 600,
      "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205.jpeg",
      "childImages": [
        {
          "width": 250,
          "height": 250,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_250x250.jpeg"
        },
        {
          "width": 100,
          "height": 100,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_100x100.jpeg"
        }
      ]
    },
    "liveStream": {
      "id": "5c13f3c006f1be15e08e05c0",
      "title": "First Linux Stream",
      "description": "<p>chat on Twitch</p>",
      "thumbnail": {
        "width": 1200,
        "height": 675,
        "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412.jpeg",
        "childImages": [
          {
            "width": 400,
            "height": 225,
            "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412_400x225.jpeg"
          }
        ]
      },
      "owner": "59f94c0bdd241b70349eb72b",
      "streamPath": "/api/video/v1/us-east-1.758417551536.channel.yKkxur4ukc0B.m3u8",
      "offline": {
        "title": "Offline",
        "description": "We're offline for now – please check back later!",
        "thumbnail": {
          "width": 1920,
          "height": 1080,
          "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026.jpeg",
          "childImages": [
            {
              "width": 400,
              "height": 225,
              "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_400x225.jpeg"
            },
            {
              "width": 1200,
              "height": 675,
              "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_1200x675.jpeg"
            }
          ]
        }
      }
    },
    "subscriptionPlans": null,
    "discoverable": true,
    "subscriberCountDisplay": "total",
    "incomeDisplay": false,
    "socialLinks": {
      "instagram": "https://www.instagram.com/linustech/",
      "twitter": "https://twitter.com/linustech",
      "website": "https://linustechtips.com",
      "facebook": "https://www.facebook.com/LinusTech",
      "youtube": "https://www.youtube.com/user/LinusTechTips"
    },
    "discordServers": [
      {
        "id": "5baa8838d9f3aa0a83acd429",
        "guildName": "LinusTechTips",
        "guildIcon": "a_528743a32b33b5eb227a8405d5593473",
        "inviteLink": "https://discord.gg/LTT",
        "inviteMode": "link"
      },
      {
        "id": "5e34cd9a9dbb744872192895",
        "guildName": "LTT Minecraft Network",
        "guildIcon": "4f7f812b49196b1646bdcdb84b948c84",
        "inviteLink": "https://discord.gg/VVpwBPXrMc",
        "inviteMode": "link"
      }
    ]
  }
]
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getcreatorinfobyname-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getcreatorinfobyname-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|[allOf]|false|none|none|

*allOf*

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» *anonymous*|[CreatorModelV2](#schemacreatormodelv2)|false|none|none|
|»» id|string|true|none|none|
|»» owner|string|true|none|none|
|»» title|string|true|none|none|
|»» urlname|string|true|none|Shown in the browser URL, and used in `/creator/named` queries.|
|»» description|string|true|none|none|
|»» about|string|true|none|none|
|»» category|string|true|none|none|
|»» cover|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»» width|integer|true|none|none|
|»»» height|integer|true|none|none|
|»»» path|string(uri)|true|none|none|
|»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»»»» width|integer|true|none|none|
|»»»» height|integer|true|none|none|
|»»»» path|string(uri)|true|none|none|
|»» icon|[ImageModel](#schemaimagemodel)|true|none|none|
|»»» width|integer|true|none|none|
|»»» height|integer|true|none|none|
|»»» path|string(uri)|true|none|none|
|»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»» liveStream|[LiveStreamModel](#schemalivestreammodel)¦null|true|none|none|
|»»» id|string|true|none|none|
|»»» title|string|true|none|none|
|»»» description|string|true|none|none|
|»»» thumbnail|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»»» width|integer|true|none|none|
|»»»» height|integer|true|none|none|
|»»»» path|string(uri)|true|none|none|
|»»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»»» owner|string|true|none|none|
|»»» channel|string|false|none|The creator channel this livestream belongs to.|
|»»» streamPath|string|true|none|none|
|»»» offline|object|true|none|none|
|»»»» title|string¦null|true|none|none|
|»»»» description|string¦null|true|none|none|
|»»»» thumbnail|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»»»» width|integer|true|none|none|
|»»»»» height|integer|true|none|none|
|»»»»» path|string(uri)|true|none|none|
|»»»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»» subscriptionPlans|[[SubscriptionPlanModel](#schemasubscriptionplanmodel)]¦null|true|none|none|
|»»» id|string|true|none|none|
|»»» title|string|true|none|none|
|»»» description|string|true|none|none|
|»»» price|string¦null|true|none|none|
|»»» priceYearly|string¦null|false|none|none|
|»»» currency|string|true|none|none|
|»»» logo|string¦null|true|none|none|
|»»» interval|string|true|none|none|
|»»» featured|boolean|true|none|none|
|»»» allowGrandfatheredAccess|boolean¦null|false|none|none|
|»»» discordServers|[[DiscordServerModel](#schemadiscordservermodel)]|true|none|none|
|»»»» id|string|true|none|none|
|»»»» guildName|string|true|none|none|
|»»»» guildIcon|string|true|none|none|
|»»»» inviteLink|string(uri)¦null|true|none|none|
|»»»» inviteMode|string|true|none|none|
|»»» discordRoles|[[DiscordRoleModel](#schemadiscordrolemodel)]|true|none|none|
|»»»» server|string|true|none|none|
|»»»» roleName|string|true|none|none|
|»» discoverable|boolean|true|none|none|
|»» subscriberCountDisplay|string|true|none|none|
|»» incomeDisplay|boolean|true|none|none|
|»» defaultChannel|string|false|none|none|

*and*

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» *anonymous*|object|false|none|none|
|»» socialLinks|[SocialLinksModel](#schemasociallinksmodel)|true|none|none|
|»»» **additionalProperties**|string(uri)|false|none|none|
|»» discordServers|[[DiscordServerModel](#schemadiscordservermodel)]|true|none|none|

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

<h1 id="floatplane-rest-api-creatorv3">CreatorV3</h1>

Get and discover creators on the platform. Creator invitation and profile management.

## getCreator

<a id="opIdgetCreator"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v3/creator/info?id=string \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v3/creator/info?id=string HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/creator/info?id=string',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v3/creator/info',
  params: {
  'id' => 'string'
}, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v3/creator/info', params={
  'id': 'string'
}, headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v3/creator/info', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/creator/info?id=string");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v3/creator/info", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v3/creator/info`

*Get Creator*

Retrieve detailed information about a specific creator.

<h3 id="getcreator-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id|query|string|true|The GUID of the creator being searched.|

> Example responses

> 200 Response

```json
{
  "id": "59f94c0bdd241b70349eb72b",
  "owner": "59f94c0bdd241b70349eb723",
  "title": "LinusTechTips",
  "urlname": "linustechtips",
  "description": "We make entertaining videos about technology, including tech reviews, showcases and other content.",
  "about": "# We're LinusTechTips\nWe make videos and stuff, cool eh?",
  "category": {
    "title": "Technology"
  },
  "cover": {
    "width": 1990,
    "height": 519,
    "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867.jpeg",
    "childImages": [
      {
        "width": 1245,
        "height": 325,
        "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867_1245x325.jpeg"
      }
    ]
  },
  "icon": {
    "width": 600,
    "height": 600,
    "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205.jpeg",
    "childImages": [
      {
        "width": 250,
        "height": 250,
        "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_250x250.jpeg"
      },
      {
        "width": 100,
        "height": 100,
        "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_100x100.jpeg"
      }
    ]
  },
  "liveStream": {
    "id": "5c13f3c006f1be15e08e05c0",
    "title": "First Linux Stream",
    "description": "<p>chat on Twitch</p>",
    "thumbnail": {
      "width": 1200,
      "height": 675,
      "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412.jpeg",
      "childImages": [
        {
          "width": 400,
          "height": 225,
          "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412_400x225.jpeg"
        }
      ]
    },
    "owner": "59f94c0bdd241b70349eb72b",
    "streamPath": "/api/video/v1/us-east-1.758417551536.channel.yKkxur4ukc0B.m3u8",
    "offline": {
      "title": "Offline",
      "description": "We're offline for now – please check back later!",
      "thumbnail": {
        "width": 1920,
        "height": 1080,
        "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026.jpeg",
        "childImages": [
          {
            "width": 400,
            "height": 225,
            "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_400x225.jpeg"
          },
          {
            "width": 1200,
            "height": 675,
            "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_1200x675.jpeg"
          }
        ]
      }
    }
  },
  "subscriptionPlans": [
    {
      "id": "5d48d0306825b5780db93d07",
      "title": "LTT Supporter (1080p)",
      "description": "Includes:\n- Early access (when possible)\n- Live Streaming\n- Behind-the-scenes, cutting room floor & exclusives\n\nNOTE: Tech Quickie and TechLinked are included for now, but will move to their own Floatplane pages in the future",
      "price": "5.00",
      "priceYearly": "50.00",
      "currency": "usd",
      "logo": null,
      "interval": "month",
      "featured": true,
      "allowGrandfatheredAccess": false,
      "discordServers": [],
      "discordRoles": []
    },
    {
      "id": "5e0ba6ac14e2590f760a0f0f",
      "title": "LTT Supporter Plus",
      "description": "You are the real MVP. \n\nYour support helps us continue to build out our team, drive up production values, run experiments that might lose money for a long time (*cough* LTX *cough*) and otherwise be the best content creators we can be.\n\nThis tier includes all the perks of the previous ones, but at floatplane's glorious high bitrate 4K!",
      "price": "10.00",
      "priceYearly": "100.00",
      "currency": "usd",
      "logo": null,
      "interval": "month",
      "featured": false,
      "allowGrandfatheredAccess": false,
      "discordServers": [],
      "discordRoles": []
    }
  ],
  "discoverable": true,
  "subscriberCountDisplay": "total",
  "incomeDisplay": false,
  "socialLinks": {
    "instagram": "https://www.instagram.com/linustech/",
    "website": "https://linustechtips.com",
    "facebook": "https://www.facebook.com/LinusTech",
    "youtube": "https://www.youtube.com/user/LinusTechTips",
    "twitter": "https://twitter.com/linustech"
  }
}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getcreator-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Creator information returned|[CreatorModelV3](#schemacreatormodelv3)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getcreator-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## getCreators

<a id="opIdgetCreators"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v3/creator/list?search=string \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v3/creator/list?search=string HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/creator/list?search=string',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v3/creator/list',
  params: {
  'search' => 'string'
}, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v3/creator/list', params={
  'search': 'string'
}, headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v3/creator/list', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/creator/list?search=string");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v3/creator/list", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v3/creator/list`

*Get Creators*

Retrieve and search for all creators on Floatplane. Useful for creator discovery and filtering.

<h3 id="getcreators-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|search|query|string|true|Optional search string for finding particular creators on the platform.|

> Example responses

> 200 Response

```json
[
  {
    "id": "59f94c0bdd241b70349eb72b",
    "owner": "59f94c0bdd241b70349eb723",
    "title": "LinusTechTips",
    "urlname": "linustechtips",
    "description": "We make entertaining videos about technology, including tech reviews, showcases and other content.",
    "about": "# We're LinusTechTips\nWe make videos and stuff, cool eh?",
    "category": {
      "title": "Technology"
    },
    "cover": {
      "width": 1990,
      "height": 519,
      "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867.jpeg",
      "childImages": [
        {
          "width": 1245,
          "height": 325,
          "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867_1245x325.jpeg"
        }
      ]
    },
    "icon": {
      "width": 600,
      "height": 600,
      "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205.jpeg",
      "childImages": [
        {
          "width": 250,
          "height": 250,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_250x250.jpeg"
        },
        {
          "width": 100,
          "height": 100,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_100x100.jpeg"
        }
      ]
    },
    "liveStream": {
      "id": "5c13f3c006f1be15e08e05c0",
      "title": "First Linux Stream",
      "description": "<p>chat on Twitch</p>",
      "thumbnail": {
        "width": 1200,
        "height": 675,
        "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412.jpeg",
        "childImages": [
          {
            "width": 400,
            "height": 225,
            "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412_400x225.jpeg"
          }
        ]
      },
      "owner": "59f94c0bdd241b70349eb72b",
      "streamPath": "/api/video/v1/us-east-1.758417551536.channel.yKkxur4ukc0B.m3u8",
      "offline": {
        "title": "Offline",
        "description": "We're offline for now – please check back later!",
        "thumbnail": {
          "width": 1920,
          "height": 1080,
          "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026.jpeg",
          "childImages": [
            {
              "width": 400,
              "height": 225,
              "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_400x225.jpeg"
            },
            {
              "width": 1200,
              "height": 675,
              "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_1200x675.jpeg"
            }
          ]
        }
      }
    },
    "subscriptionPlans": [
      {
        "id": "5d48d0306825b5780db93d07",
        "title": "LTT Supporter (1080p)",
        "description": "Includes:\n- Early access (when possible)\n- Live Streaming\n- Behind-the-scenes, cutting room floor & exclusives\n\nNOTE: Tech Quickie and TechLinked are included for now, but will move to their own Floatplane pages in the future",
        "price": "5.00",
        "priceYearly": "50.00",
        "currency": "usd",
        "logo": null,
        "interval": "month",
        "featured": true,
        "allowGrandfatheredAccess": false,
        "discordServers": [],
        "discordRoles": []
      },
      {
        "id": "5e0ba6ac14e2590f760a0f0f",
        "title": "LTT Supporter Plus",
        "description": "You are the real MVP. \n\nYour support helps us continue to build out our team, drive up production values, run experiments that might lose money for a long time (*cough* LTX *cough*) and otherwise be the best content creators we can be.\n\nThis tier includes all the perks of the previous ones, but at floatplane's glorious high bitrate 4K!",
        "price": "10.00",
        "priceYearly": "100.00",
        "currency": "usd",
        "logo": null,
        "interval": "month",
        "featured": false,
        "allowGrandfatheredAccess": false,
        "discordServers": [],
        "discordRoles": []
      }
    ],
    "discoverable": true,
    "subscriberCountDisplay": "total",
    "incomeDisplay": false,
    "socialLinks": {
      "instagram": "https://www.instagram.com/linustech/",
      "twitter": "https://twitter.com/linustech",
      "website": "https://linustechtips.com",
      "facebook": "https://www.facebook.com/LinusTech",
      "youtube": "https://www.youtube.com/user/LinusTechTips"
    }
  },
  {
    "id": "5ae0f8114336369a2c3619b6",
    "owner": "5ae0f8114336369a2c3619b4",
    "title": "TechDeals",
    "urlname": "tech_deals",
    "description": "Welcome to Tech Deals on Floatplane!  Having nothing to do with actual floatplanes since 2016, we are proud to be part of the launch of Floatplane!  We make videos about technology!",
    "about": "Welcome to Tech Deals on Floatplane!  Having nothing to do with actual floatplanes since 2016, we are proud to be part of the launch of Floatplane!  We make videos about technology!",
    "category": {
      "title": "Technology"
    },
    "cover": {
      "width": 1923,
      "height": 502,
      "path": "https://pbs.floatplane.com/cover_images/5ae0f8114336369a2c3619b6/264955378957772_1600880420171.jpeg",
      "childImages": [
        {
          "width": 1245,
          "height": 325,
          "path": "https://pbs.floatplane.com/cover_images/5ae0f8114336369a2c3619b6/264955378957772_1600880420171_1245x325.jpeg"
        }
      ]
    },
    "icon": {
      "width": 720,
      "height": 720,
      "path": "https://pbs.floatplane.com/creator_icons/5ae0f8114336369a2c3619b6/223941270270735_1600882905853.jpeg",
      "childImages": [
        {
          "width": 100,
          "height": 100,
          "path": "https://pbs.floatplane.com/creator_icons/5ae0f8114336369a2c3619b6/223941270270735_1600882905853_100x100.jpeg"
        },
        {
          "width": 250,
          "height": 250,
          "path": "https://pbs.floatplane.com/creator_icons/5ae0f8114336369a2c3619b6/223941270270735_1600882905853_250x250.jpeg"
        }
      ]
    },
    "liveStream": {
      "id": "5c3d7ec606f1be114ca1e59c",
      "title": "CES 2020 - Bonus Coverage for Floatplane Subs",
      "description": "Welcome to the Tech Deals stream – this should be fun!",
      "thumbnail": {
        "width": 1199,
        "height": 674,
        "path": "https://pbs.floatplane.com/stream_thumbnails/5c3d7ec606f1be114ca1e59c/973032281888832_1560301350562.jpeg",
        "childImages": [
          {
            "width": 400,
            "height": 225,
            "path": "https://pbs.floatplane.com/stream_thumbnails/5c3d7ec606f1be114ca1e59c/973032281888832_1560301350562_400x225.jpeg"
          }
        ]
      },
      "owner": "5ae0f8114336369a2c3619b6",
      "streamPath": "/api/video/v1/us-east-1.758417551536.channel.kdUMaxvL2eyW.m3u8",
      "offline": {
        "title": "We're offline at the moment. Please check back later!",
        "description": "We're offline at the moment. Please check back later!",
        "thumbnail": null
      }
    },
    "subscriptionPlans": [
      {
        "id": "5d506f2f7c7e6afa2ef1e246",
        "title": "1080p Plan - Videos on Demand + Live Streams",
        "description": "This plan gives you access to all published videos at up to 1080p detail.  You also get access to 1080p live streams exclusive to Floatplane.  This also grants you access to the private channels on the Tech Deals Discord, link your Floatplane account to Discord to be automatically upgraded.",
        "price": "5.00",
        "priceYearly": "50.00",
        "currency": "usd",
        "logo": null,
        "interval": "month",
        "featured": false,
        "allowGrandfatheredAccess": false,
        "discordServers": [],
        "discordRoles": []
      },
      {
        "id": "5e1710272aae3bc9cabdf505",
        "title": "4K - All Access Plan",
        "description": "This plan gives you access to all published videos at up to 4K detail.  You also get access to 1080p live streams exclusive to Floatplane.  This also grants you access to the private channels on the Tech Deals Discord, link your Floatplane account to Discord to be automatically upgraded.  BONUS - This plan allows you to download the videos in high quality for off-line viewing!  This is a great way to increase your level of support if you really love our content!",
        "price": "10.00",
        "priceYearly": "100.00",
        "currency": "usd",
        "logo": null,
        "interval": "month",
        "featured": false,
        "allowGrandfatheredAccess": false,
        "discordServers": [],
        "discordRoles": []
      }
    ],
    "discoverable": true,
    "subscriberCountDisplay": "all",
    "incomeDisplay": false,
    "socialLinks": {
      "youtube": "https://www.youtube.com/techdeals",
      "twitter": "https://twitter.com/TechDeals_16"
    }
  },
  {
    "id": "5d2fd26df33b8d14fc5ff48d",
    "owner": "5c4280ff4160af3309527f37",
    "title": "EposVox",
    "urlname": "eposvox",
    "description": "The ORIGINAL content creator and streaming focused tech education channel. EposVox, the Stream Professor, is here to give you how to videos, tutorials, tips and tricks, as well as gear reviews and benchmarks to get the most out of your tech experience. \nWhile content creation and streaming (especially with software like OBS Studio and XSplit) are primary focuses of content, ultimately the goal is to make technology easier and more fun to use. Education is number one, entertainment is sometimes given.\nThe analog tech nostalgia is just inherent to being a 90s kid.\n\n📬 Shipping: \nP.O. Box 459 \nJeffersonville, IN 47131",
    "about": "The ORIGINAL content creator and streaming focused tech education channel. EposVox, the Stream Professor, is here to give you how to videos, tutorials, tips and tricks, as well as gear reviews and benchmarks to get the most out of your tech experience. \nWhile content creation and streaming (especially with software like OBS Studio and XSplit) are primary focuses of content, ultimately the goal is to make technology easier and more fun to use. Education is number one, entertainment is sometimes given.\nThe analog tech nostalgia is just inherent to being a 90s kid.\n\n📬 Shipping: \nP.O. Box 459 \nJeffersonville, IN 47131",
    "category": {
      "title": "Technology"
    },
    "cover": {
      "width": 1992,
      "height": 520,
      "path": "https://pbs.floatplane.com/cover_images/5d2fd26df33b8d14fc5ff48d/879571788095471_1563437947857.jpeg",
      "childImages": [
        {
          "width": 1245,
          "height": 325,
          "path": "https://pbs.floatplane.com/cover_images/5d2fd26df33b8d14fc5ff48d/879571788095471_1563437947857_1245x325.jpeg"
        }
      ]
    },
    "icon": {
      "width": 720,
      "height": 720,
      "path": "https://pbs.floatplane.com/creator_icons/5d2fd26df33b8d14fc5ff48d/136876584569877_1596766926335.jpeg",
      "childImages": [
        {
          "width": 100,
          "height": 100,
          "path": "https://pbs.floatplane.com/creator_icons/5d2fd26df33b8d14fc5ff48d/136876584569877_1596766926335_100x100.jpeg"
        },
        {
          "width": 250,
          "height": 250,
          "path": "https://pbs.floatplane.com/creator_icons/5d2fd26df33b8d14fc5ff48d/136876584569877_1596766926335_250x250.jpeg"
        }
      ]
    },
    "liveStream": {
      "id": "5d2fd230f33b8d14fc5ff48c",
      "title": "EposVox Live Testing",
      "description": "Not much to see here... yet!",
      "thumbnail": null,
      "owner": "5d2fd26df33b8d14fc5ff48d",
      "streamPath": "/live_abr/eposvox",
      "offline": {
        "title": null,
        "description": "Not much to see here... yet!",
        "thumbnail": {
          "width": 1920,
          "height": 1080,
          "path": "https://pbs.floatplane.com/stream_thumbnails/5d2fd230f33b8d14fc5ff48c/859602689851697_1563602860461.jpeg",
          "childImages": [
            {
              "width": 1200,
              "height": 675,
              "path": "https://pbs.floatplane.com/stream_thumbnails/5d2fd230f33b8d14fc5ff48c/859602689851697_1563602860461_1200x675.jpeg"
            },
            {
              "width": 400,
              "height": 225,
              "path": "https://pbs.floatplane.com/stream_thumbnails/5d2fd230f33b8d14fc5ff48c/859602689851697_1563602860461_400x225.jpeg"
            }
          ]
        }
      }
    },
    "subscriptionPlans": [
      {
        "id": "5d32780027050d0c9a0b10aa",
        "title": "FLOATPLANE FLOATBOATS (1080p)",
        "description": "Primary Floatplane sub. Help me build the best tech education platform on the internet and keep diving into crazy nerdy details no one else covers. This tier gets you all of the currently-available video playback, downloads and live streaming.  BTS Vlogs & Early Access, too! All up to 1080p quality.\nVideos will be given as early access as much as a month (though usually just a week) before YouTube, when available.",
        "price": "5.00",
        "priceYearly": null,
        "currency": "usd",
        "logo": null,
        "interval": "month",
        "featured": false,
        "discordServers": [],
        "discordRoles": []
      },
      {
        "id": "5d48aaee6825b5780db93c80",
        "title": "Super Sub (4k)",
        "description": "You get... EVERYTHING! Future features will expand here, but for now, you get everything already available, and this is just here for those who want to support a little more :)",
        "price": "10.00",
        "priceYearly": null,
        "currency": "usd",
        "logo": null,
        "interval": "month",
        "featured": false,
        "discordServers": [],
        "discordRoles": []
      }
    ],
    "discoverable": true,
    "subscriberCountDisplay": "hide",
    "incomeDisplay": false,
    "socialLinks": {
      "youtube": "https://www.youtube.com/eposvox",
      "instagram": "https://www.instagram.com/eposvox/",
      "website": "https://eposvox.com",
      "facebook": "https://www.facebook.com/eposvoxofficial",
      "twitter": "https://twitter.com/eposvox"
    }
  }
]
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getcreators-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Creators returned|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getcreators-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|[[CreatorModelV3](#schemacreatormodelv3)]|false|none|none|
|» id|string|true|none|none|
|» owner|object|true|none|none|
|»» id|string|true|none|none|
|»» username|string|true|none|none|
|» title|string|true|none|none|
|» urlname|string|true|none|Shown in the browser URL, and used in `/creator/named` queries.|
|» description|string|true|none|none|
|» about|string|true|none|none|
|» category|object|true|none|none|
|»» id|string|true|none|none|
|»» title|string|true|none|none|
|» cover|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»» width|integer|true|none|none|
|»» height|integer|true|none|none|
|»» path|string(uri)|true|none|none|
|»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»»» width|integer|true|none|none|
|»»» height|integer|true|none|none|
|»»» path|string(uri)|true|none|none|
|» icon|[ImageModel](#schemaimagemodel)|true|none|none|
|»» width|integer|true|none|none|
|»» height|integer|true|none|none|
|»» path|string(uri)|true|none|none|
|»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|» liveStream|[LiveStreamModel](#schemalivestreammodel)¦null|true|none|none|
|»» id|string|true|none|none|
|»» title|string|true|none|none|
|»» description|string|true|none|none|
|»» thumbnail|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»» width|integer|true|none|none|
|»»» height|integer|true|none|none|
|»»» path|string(uri)|true|none|none|
|»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»» owner|string|true|none|none|
|»» channel|string|false|none|The creator channel this livestream belongs to.|
|»» streamPath|string|true|none|none|
|»» offline|object|true|none|none|
|»»» title|string¦null|true|none|none|
|»»» description|string¦null|true|none|none|
|»»» thumbnail|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»»» width|integer|true|none|none|
|»»»» height|integer|true|none|none|
|»»»» path|string(uri)|true|none|none|
|»»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|» subscriptionPlans|[[SubscriptionPlanModel](#schemasubscriptionplanmodel)]¦null|true|none|none|
|»» id|string|true|none|none|
|»» title|string|true|none|none|
|»» description|string|true|none|none|
|»» price|string¦null|true|none|none|
|»» priceYearly|string¦null|false|none|none|
|»» currency|string|true|none|none|
|»» logo|string¦null|true|none|none|
|»» interval|string|true|none|none|
|»» featured|boolean|true|none|none|
|»» allowGrandfatheredAccess|boolean¦null|false|none|none|
|»» discordServers|[[DiscordServerModel](#schemadiscordservermodel)]|true|none|none|
|»»» id|string|true|none|none|
|»»» guildName|string|true|none|none|
|»»» guildIcon|string|true|none|none|
|»»» inviteLink|string(uri)¦null|true|none|none|
|»»» inviteMode|string|true|none|none|
|»» discordRoles|[[DiscordRoleModel](#schemadiscordrolemodel)]|true|none|none|
|»»» server|string|true|none|none|
|»»» roleName|string|true|none|none|
|» discoverable|boolean|true|none|none|
|» subscriberCountDisplay|string|true|none|none|
|» incomeDisplay|boolean|true|none|none|
|» defaultChannel|string|true|none|none|
|» socialLinks|[SocialLinksModel](#schemasociallinksmodel)|true|none|none|
|»» **additionalProperties**|string(uri)|false|none|none|
|» channels|[[ChannelModel](#schemachannelmodel)]|true|none|none|
|»» id|string|true|none|none|
|»» creator|string|true|none|none|
|»» title|string|true|none|none|
|»» urlname|string|true|none|Shown in the browser URL.|
|»» about|string|true|none|none|
|»» order|integer|false|none|none|
|»» cover|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»» width|integer|true|none|none|
|»»» height|integer|true|none|none|
|»»» path|string(uri)|true|none|none|
|»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»» card|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»» width|integer|true|none|none|
|»»» height|integer|true|none|none|
|»»» path|string(uri)|true|none|none|
|»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»» icon|[ImageModel](#schemaimagemodel)|true|none|none|
|»» socialLinks|[SocialLinksModel](#schemasociallinksmodel)|false|none|none|
|» discordServers|[[DiscordServerModel](#schemadiscordservermodel)]|false|none|Present in `/creator/named` queries|
|» card|[ImageModel](#schemaimagemodel)|false|none|none|

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## getCreatorByName

<a id="opIdgetCreatorByName"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v3/creator/named?creatorURL=string \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v3/creator/named?creatorURL=string HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/creator/named?creatorURL=string',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v3/creator/named',
  params: {
  'creatorURL' => 'array[string]'
}, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v3/creator/named', params={
  'creatorURL': [
  "string"
]
}, headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v3/creator/named', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/creator/named?creatorURL=string");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v3/creator/named", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v3/creator/named`

*Get Creator By Name*

Retrieve detailed information on one or more creators on Floatplane.

<h3 id="getcreatorbyname-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|creatorURL|query|array[string]|true|The `urlname`(s) of the creator(s) to be retrieved. See `CreatorModelV3`.|

> Example responses

> 200 Response

```json
[
  {
    "id": "59f94c0bdd241b70349eb72b",
    "owner": "59f94c0bdd241b70349eb723",
    "title": "LinusTechTips",
    "urlname": "linustechtips",
    "description": "We make entertaining videos about technology, including tech reviews, showcases and other content.",
    "about": "# We're LinusTechTips\nWe make videos and stuff, cool eh?",
    "category": {
      "id": "59f94c0bdd241b70349eb727",
      "title": "Technology"
    },
    "cover": {
      "width": 1990,
      "height": 519,
      "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867.jpeg",
      "childImages": [
        {
          "width": 1245,
          "height": 325,
          "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867_1245x325.jpeg"
        }
      ]
    },
    "icon": {
      "width": 600,
      "height": 600,
      "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205.jpeg",
      "childImages": [
        {
          "width": 250,
          "height": 250,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_250x250.jpeg"
        },
        {
          "width": 100,
          "height": 100,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_100x100.jpeg"
        }
      ]
    },
    "liveStream": {
      "id": "5c13f3c006f1be15e08e05c0",
      "title": "I Am Hard... [REDACTED] - WAN Show March 17, 2023",
      "description": "<p>Get a 15-day free trial for unlimited backup at<a href=\"https://www.backblaze.com/landing/podcast-wan.html\" rel=\"noopener noreferrer\" target=\"_blank\"> https://www.backblaze.com/landing/podcast-wan.html</a></p><p>Try Zoho One free for 30 days with no credit card required here:<a href=\"https://www.zoho.com/one/lp/linus.html\" rel=\"noopener noreferrer\" target=\"_blank\"> https://www.zoho.com/one/lp/linus.html</a></p><p>Save 15% with our offer code WANSHOW at <a href=\"https://vessi.com/WANSHOW\" rel=\"noopener noreferrer\" target=\"_blank\">https://vessi.com/WANSHOW</a></p><p><br></p><p>Podcast Download: TBD</p>",
      "thumbnail": {
        "width": 1920,
        "height": 1080,
        "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/754645543818842_1679099143677.jpeg",
        "childImages": [
          {
            "width": 400,
            "height": 225,
            "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/754645543818842_1679099143677_400x225.jpeg"
          },
          {
            "width": 1200,
            "height": 675,
            "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/754645543818842_1679099143677_1200x675.jpeg"
          }
        ]
      },
      "owner": "59f94c0bdd241b70349eb72b",
      "channel": "63fe42c309e691e4e36de93d",
      "streamPath": "/api/video/v1/us-east-1.758417551536.channel.yKkxur4ukc0B.m3u8",
      "offline": {
        "title": "Offline",
        "description": "We're offline for now – please check back later!",
        "thumbnail": {
          "width": 1920,
          "height": 1080,
          "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026.jpeg",
          "childImages": [
            {
              "width": 400,
              "height": 225,
              "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_400x225.jpeg"
            },
            {
              "width": 1200,
              "height": 675,
              "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_1200x675.jpeg"
            }
          ]
        }
      }
    },
    "subscriptionPlans": [
      {
        "id": "5d48d0306825b5780db93d07",
        "title": "LTT Supporter",
        "description": "- 2 Exclusives Per Week (Meet the Team, Extras, Behind the Scenes) \n- Exclusive livestreams\n- Save $10 by purchasing an annual subscription\n- Our gratitude for your support",
        "price": "5.00",
        "priceYearly": "50.00",
        "currency": "usd",
        "logo": null,
        "interval": "month",
        "featured": true,
        "allowGrandfatheredAccess": false,
        "discordServers": [],
        "discordRoles": []
      },
      {
        "id": "5e0ba6ac14e2590f760a0f0f",
        "title": "LTT Supporter Plus",
        "description": "- 4K Bitrate Streaming\n- 2 Exclusives Per Week (Meet the Team, Extras, Behind the Scenes) \n- Exclusive livestreams\n- Save $20 by purchasing an annual subscription\n- Our gratitude for your support",
        "price": "10.00",
        "priceYearly": "100.00",
        "currency": "usd",
        "logo": null,
        "interval": "month",
        "featured": true,
        "allowGrandfatheredAccess": false,
        "discordServers": [],
        "discordRoles": []
      }
    ],
    "discoverable": true,
    "subscriberCountDisplay": "total",
    "incomeDisplay": false,
    "defaultChannel": "63fe42c309e691e4e36de93d",
    "socialLinks": {
      "instagram": "https://www.instagram.com/linustech",
      "website": "https://linustechtips.com",
      "facebook": "https://www.facebook.com/LinusTech",
      "youtube": "https://www.youtube.com/user/LinusTechTips",
      "twitter": "https://twitter.com/linustech"
    },
    "channels": [
      {
        "id": "63fe42c309e691e4e36de93d",
        "creator": "59f94c0bdd241b70349eb72b",
        "title": "Linus Tech Tips",
        "urlname": "main",
        "about": "# We're LinusTechTips\nWe make videos and stuff, cool eh?",
        "order": 0,
        "cover": {
          "width": 1990,
          "height": 519,
          "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867.jpeg",
          "childImages": [
            {
              "width": 1245,
              "height": 325,
              "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867_1245x325.jpeg"
            }
          ]
        },
        "card": null,
        "icon": {
          "width": 600,
          "height": 600,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205.jpeg",
          "childImages": [
            {
              "width": 250,
              "height": 250,
              "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_250x250.jpeg"
            },
            {
              "width": 100,
              "height": 100,
              "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_100x100.jpeg"
            }
          ]
        },
        "socialLinks": {}
      },
      {
        "id": "6413534d88c13c181c3e2809",
        "creator": "59f94c0bdd241b70349eb72b",
        "title": "TechLinked",
        "urlname": "techlinked",
        "about": "News about tech + gaming culture, delivered thrice weekly.\n\nWe're also doing long-form video essays now, apparently. \n\nThe TalkLinked talk show/podcast will probably come back at some point, too!\n\nWriters: Riley Murdock, Jon Martin, James Strieb",
        "order": 2,
        "cover": {
          "width": 1080,
          "height": 282,
          "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/6413534d88c13c181c3e2809/231100243161134_1678988109632.jpeg",
          "childImages": []
        },
        "card": null,
        "icon": {
          "width": 88,
          "height": 88,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/6413534d88c13c181c3e2809/955526950207988_1678988110287.jpeg",
          "childImages": []
        },
        "socialLinks": {}
      },
      {
        "id": "64135da7ce81077a8480c679",
        "creator": "59f94c0bdd241b70349eb72b",
        "title": "ShortCircuit",
        "urlname": "shortcircuit",
        "about": "What's in the box? Let's find out!\n\nOfficial channel under Linus Media Group.",
        "order": 3,
        "cover": {
          "width": 1084,
          "height": 283,
          "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/64135da7ce81077a8480c679/745715133852622_1678990806332.jpeg",
          "childImages": []
        },
        "card": null,
        "icon": {
          "width": 88,
          "height": 88,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/64135da7ce81077a8480c679/470304051261927_1678990806883.jpeg",
          "childImages": []
        },
        "socialLinks": {}
      },
      {
        "id": "64135e27c773b27ff22c97eb",
        "creator": "59f94c0bdd241b70349eb72b",
        "title": "Techquickie",
        "urlname": "techquickie",
        "about": "Ever wanted to learn more about your favorite gadgets or a trending topic in tech? \n\nWith a mix of humor, cynicism, and insight, Techquickie brings you the answers to all your tech questions every Tuesday and Friday.",
        "order": 5,
        "cover": {
          "width": 1080,
          "height": 282,
          "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/64135e27c773b27ff22c97eb/721553790654237_1678990887992.jpeg",
          "childImages": []
        },
        "card": null,
        "icon": {
          "width": 88,
          "height": 88,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/64135e27c773b27ff22c97eb/666841640245092_1678990909616.jpeg",
          "childImages": []
        },
        "socialLinks": {}
      },
      {
        "id": "64135e901ebaee42e258eb0b",
        "creator": "59f94c0bdd241b70349eb72b",
        "title": "Mac Address",
        "urlname": "macaddress",
        "about": "The exploration of all things Apple, from iPhones underwater to full iClouds in the sky. We want to be the channel that you come to first for an unexpected viewpoint about the devices you love.",
        "order": 4,
        "cover": {
          "width": 1080,
          "height": 282,
          "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/64135e901ebaee42e258eb0b/254417940627493_1678990992632.jpeg",
          "childImages": []
        },
        "card": null,
        "icon": {
          "width": 88,
          "height": 88,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/64135e901ebaee42e258eb0b/979475909700348_1678990993114.jpeg",
          "childImages": []
        },
        "socialLinks": {}
      },
      {
        "id": "64135ed078d6262f717341b7",
        "creator": "59f94c0bdd241b70349eb72b",
        "title": "Channel Super Fun",
        "urlname": "channelsuperfun",
        "about": "Channel Super Fun is all about the name. Games, toys, and challenges. Expect to find them all here!",
        "order": 6,
        "cover": {
          "width": 1080,
          "height": 282,
          "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/64135ed078d6262f717341b7/881886551214964_1678991123807.jpeg",
          "childImages": []
        },
        "card": null,
        "icon": {
          "width": 88,
          "height": 88,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/64135ed078d6262f717341b7/317924815973639_1678991124672.jpeg",
          "childImages": []
        },
        "socialLinks": {}
      },
      {
        "id": "64135f82fc76ab7f9fbdc876",
        "creator": "59f94c0bdd241b70349eb72b",
        "title": "They're Just Movies",
        "urlname": "tajm",
        "about": "Each week our small group of nerds sits down for a not-so-serious, SPOILER-FILLED, chat about the movies you love.\n\nFormerly known as Carpool Critics, we're part of Linus Media Group!",
        "order": 7,
        "cover": {
          "width": 1080,
          "height": 282,
          "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/64135f82fc76ab7f9fbdc876/190277198232475_1678991235439.jpeg",
          "childImages": []
        },
        "card": null,
        "icon": {
          "width": 88,
          "height": 88,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/64135f82fc76ab7f9fbdc876/570806971094170_1678991236419.jpeg",
          "childImages": []
        },
        "socialLinks": {}
      },
      {
        "id": "6413623f5b12cca228a28e78",
        "creator": "59f94c0bdd241b70349eb72b",
        "title": "FP Exclusive",
        "urlname": "fpexclusive",
        "about": "wow... so empty",
        "order": 1,
        "cover": {
          "width": 1200,
          "height": 313,
          "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/6413623f5b12cca228a28e78/072932633007415_1678991935461.jpeg",
          "childImages": []
        },
        "card": null,
        "icon": {
          "width": 720,
          "height": 720,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/6413623f5b12cca228a28e78/069457536750544_1678991936484.jpeg",
          "childImages": [
            {
              "width": 100,
              "height": 100,
              "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/6413623f5b12cca228a28e78/069457536750544_1678991936484_100x100.jpeg"
            },
            {
              "width": 250,
              "height": 250,
              "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/6413623f5b12cca228a28e78/069457536750544_1678991936484_250x250.jpeg"
            }
          ]
        },
        "socialLinks": {}
      }
    ],
    "discordServers": [
      {
        "id": "5baa8838d9f3aa0a83acd429",
        "guildName": "Linus Tech Tips",
        "guildIcon": "803d5f745b30aba3edba719bac671660",
        "inviteLink": "https://discord.gg/LTT",
        "inviteMode": "link"
      },
      {
        "id": "5e34cd9a9dbb744872192895",
        "guildName": "LTT Minecraft Network",
        "guildIcon": "4f7f812b49196b1646bdcdb84b948c84",
        "inviteLink": "https://discord.gg/VVpwBPXrMc",
        "inviteMode": "link"
      }
    ]
  }
]
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getcreatorbyname-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getcreatorbyname-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|[[CreatorModelV3](#schemacreatormodelv3)]|false|none|none|
|» id|string|true|none|none|
|» owner|object|true|none|none|
|»» id|string|true|none|none|
|»» username|string|true|none|none|
|» title|string|true|none|none|
|» urlname|string|true|none|Shown in the browser URL, and used in `/creator/named` queries.|
|» description|string|true|none|none|
|» about|string|true|none|none|
|» category|object|true|none|none|
|»» id|string|true|none|none|
|»» title|string|true|none|none|
|» cover|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»» width|integer|true|none|none|
|»» height|integer|true|none|none|
|»» path|string(uri)|true|none|none|
|»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»»» width|integer|true|none|none|
|»»» height|integer|true|none|none|
|»»» path|string(uri)|true|none|none|
|» icon|[ImageModel](#schemaimagemodel)|true|none|none|
|»» width|integer|true|none|none|
|»» height|integer|true|none|none|
|»» path|string(uri)|true|none|none|
|»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|» liveStream|[LiveStreamModel](#schemalivestreammodel)¦null|true|none|none|
|»» id|string|true|none|none|
|»» title|string|true|none|none|
|»» description|string|true|none|none|
|»» thumbnail|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»» width|integer|true|none|none|
|»»» height|integer|true|none|none|
|»»» path|string(uri)|true|none|none|
|»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»» owner|string|true|none|none|
|»» channel|string|false|none|The creator channel this livestream belongs to.|
|»» streamPath|string|true|none|none|
|»» offline|object|true|none|none|
|»»» title|string¦null|true|none|none|
|»»» description|string¦null|true|none|none|
|»»» thumbnail|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»»» width|integer|true|none|none|
|»»»» height|integer|true|none|none|
|»»»» path|string(uri)|true|none|none|
|»»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|» subscriptionPlans|[[SubscriptionPlanModel](#schemasubscriptionplanmodel)]¦null|true|none|none|
|»» id|string|true|none|none|
|»» title|string|true|none|none|
|»» description|string|true|none|none|
|»» price|string¦null|true|none|none|
|»» priceYearly|string¦null|false|none|none|
|»» currency|string|true|none|none|
|»» logo|string¦null|true|none|none|
|»» interval|string|true|none|none|
|»» featured|boolean|true|none|none|
|»» allowGrandfatheredAccess|boolean¦null|false|none|none|
|»» discordServers|[[DiscordServerModel](#schemadiscordservermodel)]|true|none|none|
|»»» id|string|true|none|none|
|»»» guildName|string|true|none|none|
|»»» guildIcon|string|true|none|none|
|»»» inviteLink|string(uri)¦null|true|none|none|
|»»» inviteMode|string|true|none|none|
|»» discordRoles|[[DiscordRoleModel](#schemadiscordrolemodel)]|true|none|none|
|»»» server|string|true|none|none|
|»»» roleName|string|true|none|none|
|» discoverable|boolean|true|none|none|
|» subscriberCountDisplay|string|true|none|none|
|» incomeDisplay|boolean|true|none|none|
|» defaultChannel|string|true|none|none|
|» socialLinks|[SocialLinksModel](#schemasociallinksmodel)|true|none|none|
|»» **additionalProperties**|string(uri)|false|none|none|
|» channels|[[ChannelModel](#schemachannelmodel)]|true|none|none|
|»» id|string|true|none|none|
|»» creator|string|true|none|none|
|»» title|string|true|none|none|
|»» urlname|string|true|none|Shown in the browser URL.|
|»» about|string|true|none|none|
|»» order|integer|false|none|none|
|»» cover|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»» width|integer|true|none|none|
|»»» height|integer|true|none|none|
|»»» path|string(uri)|true|none|none|
|»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»» card|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»» width|integer|true|none|none|
|»»» height|integer|true|none|none|
|»»» path|string(uri)|true|none|none|
|»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»» icon|[ImageModel](#schemaimagemodel)|true|none|none|
|»» socialLinks|[SocialLinksModel](#schemasociallinksmodel)|false|none|none|
|» discordServers|[[DiscordServerModel](#schemadiscordservermodel)]|false|none|Present in `/creator/named` queries|
|» card|[ImageModel](#schemaimagemodel)|false|none|none|

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## listCreatorChannelsV3

<a id="opIdlistCreatorChannelsV3"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v3/creator/channels/list?ids=string \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v3/creator/channels/list?ids=string HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/creator/channels/list?ids=string',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v3/creator/channels/list',
  params: {
  'ids' => 'array[string]'
}, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v3/creator/channels/list', params={
  'ids': [
  "string"
]
}, headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v3/creator/channels/list', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/creator/channels/list?ids=string");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v3/creator/channels/list", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v3/creator/channels/list`

*List Creator Channels*

Retrieves a list of channels within the given creator(s).

<h3 id="listcreatorchannelsv3-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|ids|query|array[string]|true|The ids of the creator(s) from which to search for channels.|

> Example responses

> 200 Response

```json
[
  {
    "id": "63fe42c309e691e4e36de93d",
    "creator": "59f94c0bdd241b70349eb72b",
    "title": "Linus Tech Tips",
    "urlname": "main",
    "about": "# We're LinusTechTips\nWe make videos and stuff, cool eh?",
    "order": 0,
    "cover": {
      "width": 1990,
      "height": 519,
      "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867.jpeg",
      "childImages": [
        {
          "width": 1245,
          "height": 325,
          "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867_1245x325.jpeg"
        }
      ]
    },
    "card": {
      "width": 375,
      "height": 500,
      "path": "https://pbs.floatplane.com/creator_card/59f94c0bdd241b70349eb72b/281467946609369_1551250329871.jpeg",
      "childImages": [
        {
          "width": 300,
          "height": 400,
          "path": "https://pbs.floatplane.com/creator_card/59f94c0bdd241b70349eb72b/281467946609369_1551250329871_300x400.jpeg"
        }
      ]
    },
    "icon": {
      "width": 600,
      "height": 600,
      "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205.jpeg",
      "childImages": [
        {
          "width": 250,
          "height": 250,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_250x250.jpeg"
        },
        {
          "width": 100,
          "height": 100,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_100x100.jpeg"
        }
      ]
    },
    "socialLinks": {}
  },
  {
    "id": "6413534d88c13c181c3e2809",
    "creator": "59f94c0bdd241b70349eb72b",
    "title": "TechLinked",
    "urlname": "techlinked",
    "about": "News about tech + gaming culture, delivered thrice weekly.\n\nWe're also doing long-form video essays now, apparently. \n\nThe TalkLinked talk show/podcast will probably come back at some point, too!\n\nWriters: Riley Murdock, Jon Martin, James Strieb",
    "order": 2,
    "cover": {
      "width": 1080,
      "height": 282,
      "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/6413534d88c13c181c3e2809/231100243161134_1678988109632.jpeg",
      "childImages": []
    },
    "card": {
      "width": 66,
      "height": 88,
      "path": "https://pbs.floatplane.com/creator_cards/59f94c0bdd241b70349eb72b/6413534d88c13c181c3e2809/953298867250186_1678988110714.jpeg",
      "childImages": []
    },
    "icon": {
      "width": 88,
      "height": 88,
      "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/6413534d88c13c181c3e2809/955526950207988_1678988110287.jpeg",
      "childImages": []
    },
    "socialLinks": {}
  },
  {
    "id": "64135da7ce81077a8480c679",
    "creator": "59f94c0bdd241b70349eb72b",
    "title": "ShortCircuit",
    "urlname": "shortcircuit",
    "about": "What's in the box? Let's find out!\n\nOfficial channel under Linus Media Group.",
    "order": 3,
    "cover": {
      "width": 1084,
      "height": 283,
      "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/64135da7ce81077a8480c679/745715133852622_1678990806332.jpeg",
      "childImages": []
    },
    "card": {
      "width": 66,
      "height": 88,
      "path": "https://pbs.floatplane.com/creator_cards/59f94c0bdd241b70349eb72b/64135da7ce81077a8480c679/939790311068913_1678990807440.jpeg",
      "childImages": []
    },
    "icon": {
      "width": 88,
      "height": 88,
      "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/64135da7ce81077a8480c679/470304051261927_1678990806883.jpeg",
      "childImages": []
    },
    "socialLinks": {}
  },
  {
    "id": "64135e27c773b27ff22c97eb",
    "creator": "59f94c0bdd241b70349eb72b",
    "title": "Techquickie",
    "urlname": "techquickie",
    "about": "Ever wanted to learn more about your favorite gadgets or a trending topic in tech? \n\nWith a mix of humor, cynicism, and insight, Techquickie brings you the answers to all your tech questions every Tuesday and Friday.",
    "order": 5,
    "cover": {
      "width": 1080,
      "height": 282,
      "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/64135e27c773b27ff22c97eb/721553790654237_1678990887992.jpeg",
      "childImages": []
    },
    "card": {
      "width": 66,
      "height": 88,
      "path": "https://pbs.floatplane.com/creator_cards/59f94c0bdd241b70349eb72b/64135e27c773b27ff22c97eb/962826906389296_1678990910103.jpeg",
      "childImages": []
    },
    "icon": {
      "width": 88,
      "height": 88,
      "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/64135e27c773b27ff22c97eb/666841640245092_1678990909616.jpeg",
      "childImages": []
    },
    "socialLinks": {}
  },
  {
    "id": "64135e901ebaee42e258eb0b",
    "creator": "59f94c0bdd241b70349eb72b",
    "title": "Mac Address",
    "urlname": "macaddress",
    "about": "The exploration of all things Apple, from iPhones underwater to full iClouds in the sky. We want to be the channel that you come to first for an unexpected viewpoint about the devices you love.",
    "order": 4,
    "cover": {
      "width": 1080,
      "height": 282,
      "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/64135e901ebaee42e258eb0b/254417940627493_1678990992632.jpeg",
      "childImages": []
    },
    "card": {
      "width": 66,
      "height": 88,
      "path": "https://pbs.floatplane.com/creator_cards/59f94c0bdd241b70349eb72b/64135e901ebaee42e258eb0b/493604923555386_1678990993708.jpeg",
      "childImages": []
    },
    "icon": {
      "width": 88,
      "height": 88,
      "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/64135e901ebaee42e258eb0b/979475909700348_1678990993114.jpeg",
      "childImages": []
    },
    "socialLinks": {}
  },
  {
    "id": "64135ed078d6262f717341b7",
    "creator": "59f94c0bdd241b70349eb72b",
    "title": "Channel Super Fun",
    "urlname": "channelsuperfun",
    "about": "Channel Super Fun is all about the name. Games, toys, and challenges. Expect to find them all here!",
    "order": 6,
    "cover": {
      "width": 1080,
      "height": 282,
      "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/64135ed078d6262f717341b7/881886551214964_1678991123807.jpeg",
      "childImages": []
    },
    "card": {
      "width": 66,
      "height": 88,
      "path": "https://pbs.floatplane.com/creator_cards/59f94c0bdd241b70349eb72b/64135ed078d6262f717341b7/953601834319296_1678991125167.jpeg",
      "childImages": []
    },
    "icon": {
      "width": 88,
      "height": 88,
      "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/64135ed078d6262f717341b7/317924815973639_1678991124672.jpeg",
      "childImages": []
    },
    "socialLinks": {}
  },
  {
    "id": "64135f82fc76ab7f9fbdc876",
    "creator": "59f94c0bdd241b70349eb72b",
    "title": "They're Just Movies",
    "urlname": "tajm",
    "about": "Each week our small group of nerds sits down for a not-so-serious, SPOILER-FILLED, chat about the movies you love.\n\nFormerly known as Carpool Critics, we're part of Linus Media Group!",
    "order": 7,
    "cover": {
      "width": 1080,
      "height": 282,
      "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/64135f82fc76ab7f9fbdc876/190277198232475_1678991235439.jpeg",
      "childImages": []
    },
    "card": {
      "width": 66,
      "height": 88,
      "path": "https://pbs.floatplane.com/creator_cards/59f94c0bdd241b70349eb72b/64135f82fc76ab7f9fbdc876/863827491957963_1678991237126.jpeg",
      "childImages": []
    },
    "icon": {
      "width": 88,
      "height": 88,
      "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/64135f82fc76ab7f9fbdc876/570806971094170_1678991236419.jpeg",
      "childImages": []
    },
    "socialLinks": {}
  },
  {
    "id": "6413623f5b12cca228a28e78",
    "creator": "59f94c0bdd241b70349eb72b",
    "title": "FP Exclusive",
    "urlname": "fpexclusive",
    "about": "wow... so empty",
    "order": 1,
    "cover": {
      "width": 1200,
      "height": 313,
      "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/6413623f5b12cca228a28e78/072932633007415_1678991935461.jpeg",
      "childImages": []
    },
    "card": {
      "width": 750,
      "height": 1000,
      "path": "https://pbs.floatplane.com/creator_cards/59f94c0bdd241b70349eb72b/6413623f5b12cca228a28e78/717727609968419_1678991937240.jpeg",
      "childImages": [
        {
          "width": 375,
          "height": 500,
          "path": "https://pbs.floatplane.com/creator_cards/59f94c0bdd241b70349eb72b/6413623f5b12cca228a28e78/717727609968419_1678991937240_375x500.jpeg"
        },
        {
          "width": 300,
          "height": 400,
          "path": "https://pbs.floatplane.com/creator_cards/59f94c0bdd241b70349eb72b/6413623f5b12cca228a28e78/717727609968419_1678991937240_300x400.jpeg"
        }
      ]
    },
    "icon": {
      "width": 720,
      "height": 720,
      "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/6413623f5b12cca228a28e78/069457536750544_1678991936484.jpeg",
      "childImages": [
        {
          "width": 100,
          "height": 100,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/6413623f5b12cca228a28e78/069457536750544_1678991936484_100x100.jpeg"
        },
        {
          "width": 250,
          "height": 250,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/6413623f5b12cca228a28e78/069457536750544_1678991936484_250x250.jpeg"
        }
      ]
    },
    "socialLinks": {}
  }
]
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="listcreatorchannelsv3-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="listcreatorchannelsv3-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|[[ChannelModel](#schemachannelmodel)]|false|none|none|
|» id|string|true|none|none|
|» creator|string|true|none|none|
|» title|string|true|none|none|
|» urlname|string|true|none|Shown in the browser URL.|
|» about|string|true|none|none|
|» order|integer|false|none|none|
|» cover|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»» width|integer|true|none|none|
|»» height|integer|true|none|none|
|»» path|string(uri)|true|none|none|
|»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»»» width|integer|true|none|none|
|»»» height|integer|true|none|none|
|»»» path|string(uri)|true|none|none|
|» card|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»» width|integer|true|none|none|
|»» height|integer|true|none|none|
|»» path|string(uri)|true|none|none|
|»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|» icon|[ImageModel](#schemaimagemodel)|true|none|none|
|»» width|integer|true|none|none|
|»» height|integer|true|none|none|
|»» path|string(uri)|true|none|none|
|»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|» socialLinks|[SocialLinksModel](#schemasociallinksmodel)|false|none|none|
|»» **additionalProperties**|string(uri)|false|none|none|

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

<h1 id="floatplane-rest-api-creatorsubscriptionplanv2">CreatorSubscriptionPlanV2</h1>

Manage creator subscription plans.

## getCreatorSubInfoPublic

<a id="opIdgetCreatorSubInfoPublic"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v2/plan/info?creatorId=string \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v2/plan/info?creatorId=string HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v2/plan/info?creatorId=string',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v2/plan/info',
  params: {
  'creatorId' => 'string'
}, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v2/plan/info', params={
  'creatorId': 'string'
}, headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v2/plan/info', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v2/plan/info?creatorId=string");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v2/plan/info", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v2/plan/info`

*Get Creator Sub Info Public*

Retrieve detailed information about a creator's subscription plans and their subscriber count.

<h3 id="getcreatorsubinfopublic-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|creatorId|query|string|true|The GUID for the creator being search.|

> Example responses

> 200 Response

```json
{
  "totalSubscriberCount": 19256,
  "totalIncome": null,
  "plans": [
    {
      "discordRoles": [
        {
          "server": "5baa8838d9f3aa0a83acd429",
          "roleName": "Floatplane.com Pilot"
        },
        {
          "server": "5e34cd9a9dbb744872192895",
          "roleName": "Pilot"
        }
      ],
      "createdAt": "2019-08-06T00:56:16.180Z",
      "updatedAt": "2021-09-09T16:55:02.620Z",
      "id": "5d48d0306825b5780db93d07",
      "title": "LTT Supporter (1080p)",
      "enabled": true,
      "featured": true,
      "description": "Includes:\n- Early access (when possible)\n- Live Streaming\n- Behind-the-scenes, cutting room floor & exclusives\n\nNOTE: Tech Quickie and TechLinked are included for now, but will move to their own Floatplane pages in the future",
      "price": "5.00",
      "priceYearly": "50.00",
      "paymentID": 19,
      "currency": "usd",
      "trialPeriod": 0,
      "allowGrandfatheredAccess": false,
      "logo": null,
      "creator": "59f94c0bdd241b70349eb72b",
      "discordServers": [
        {
          "id": "5baa8838d9f3aa0a83acd429",
          "guildName": "LinusTechTips",
          "guildIcon": "a_528743a32b33b5eb227a8405d5593473",
          "inviteLink": "https://discord.gg/LTT",
          "inviteMode": "link"
        },
        {
          "id": "5e34cd9a9dbb744872192895",
          "guildName": "LTT Minecraft Network",
          "guildIcon": "4f7f812b49196b1646bdcdb84b948c84",
          "inviteLink": "https://discord.gg/VVpwBPXrMc",
          "inviteMode": "link"
        }
      ],
      "userIsSubscribed": true,
      "userIsGrandfathered": false,
      "enabledGlobal": true,
      "interval": "month"
    },
    {
      "discordRoles": [
        {
          "server": "5baa8838d9f3aa0a83acd429",
          "roleName": "Floatplane.com Pilot"
        },
        {
          "server": "5e34cd9a9dbb744872192895",
          "roleName": "Pilot"
        }
      ],
      "createdAt": "2019-12-31T19:51:08.009Z",
      "updatedAt": "2020-11-07T01:33:31.617Z",
      "id": "5e0ba6ac14e2590f760a0f0f",
      "title": "LTT Supporter Plus",
      "enabled": true,
      "featured": false,
      "description": "You are the real MVP. \n\nYour support helps us continue to build out our team, drive up production values, run experiments that might lose money for a long time (*cough* LTX *cough*) and otherwise be the best content creators we can be.\n\nThis tier includes all the perks of the previous ones, but at floatplane's glorious high bitrate 4K!",
      "price": "10.00",
      "priceYearly": "100.00",
      "paymentID": 66,
      "currency": "usd",
      "trialPeriod": 0,
      "allowGrandfatheredAccess": false,
      "logo": null,
      "creator": "59f94c0bdd241b70349eb72b",
      "discordServers": [
        {
          "id": "5baa8838d9f3aa0a83acd429",
          "guildName": "LinusTechTips",
          "guildIcon": "a_528743a32b33b5eb227a8405d5593473",
          "inviteLink": "https://discord.gg/LTT",
          "inviteMode": "link"
        },
        {
          "id": "5e34cd9a9dbb744872192895",
          "guildName": "LTT Minecraft Network",
          "guildIcon": "4f7f812b49196b1646bdcdb84b948c84",
          "inviteLink": "https://discord.gg/VVpwBPXrMc",
          "inviteMode": "link"
        }
      ],
      "userIsSubscribed": false,
      "userIsGrandfathered": false,
      "enabledGlobal": true,
      "interval": "month"
    }
  ]
}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getcreatorsubinfopublic-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Information about the plans for the creator|[PlanInfoV2Response](#schemaplaninfov2response)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getcreatorsubinfopublic-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

<h1 id="floatplane-rest-api-edgesv2">EdgesV2</h1>

Get edge server information for media playback.

## getEdges

<a id="opIdgetEdges"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v2/edges \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v2/edges HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v2/edges',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v2/edges',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v2/edges', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v2/edges', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v2/edges");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v2/edges", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v2/edges`

*Get Edges*

Retrieve a list of edge servers from which to stream or download videos. This is deprecated, and using the CDN endpoint is recommended as a replacement.

> Example responses

> 200 Response

```json
{
  "edges": [
    {
      "hostname": "edge01-na.floatplane.com",
      "queryPort": 8090,
      "bandwidth": 1000000000,
      "allowDownload": true,
      "allowStreaming": true,
      "datacenter": {
        "countryCode": "CA",
        "regionCode": "QC",
        "latitude": 45.3168,
        "longitude": -73.8659
      }
    },
    {
      "hostname": "edge02-na.floatplane.com",
      "queryPort": 8090,
      "bandwidth": 500000000,
      "allowDownload": true,
      "allowStreaming": true,
      "datacenter": {
        "countryCode": "CA",
        "regionCode": "QC",
        "latitude": 45.3168,
        "longitude": -73.8659
      }
    },
    {
      "hostname": "edge01-eu.floatplane.com",
      "queryPort": 8090,
      "bandwidth": 1000000000,
      "allowDownload": false,
      "allowStreaming": true,
      "datacenter": {
        "countryCode": "FR",
        "regionCode": "A",
        "latitude": 48.5873,
        "longitude": 7.79821
      }
    },
    {
      "hostname": "edge01-au.floatplane.com",
      "queryPort": 8090,
      "bandwidth": 250000000,
      "allowDownload": false,
      "allowStreaming": true,
      "datacenter": {
        "countryCode": "AU",
        "regionCode": "NSW",
        "latitude": -33.8401,
        "longitude": 151.209
      }
    },
    {
      "hostname": "edge1-na-south.floatplane.com",
      "queryPort": 0,
      "bandwidth": 1000000000,
      "allowDownload": false,
      "allowStreaming": true,
      "datacenter": {
        "countryCode": "US",
        "regionCode": "FL",
        "latitude": 25.8124,
        "longitude": -80.2401
      }
    },
    {
      "hostname": "edge1-na-sv.floatplane.com",
      "queryPort": 0,
      "bandwidth": 1000000000,
      "allowDownload": false,
      "allowStreaming": true,
      "datacenter": {
        "countryCode": "US",
        "regionCode": "CA",
        "latitude": 37.3387,
        "longitude": -121.8914
      }
    },
    {
      "hostname": "edge03-na.floatplane.com",
      "queryPort": 0,
      "bandwidth": 3000000000,
      "allowDownload": false,
      "allowStreaming": true,
      "datacenter": {
        "countryCode": "CA",
        "regionCode": "QC",
        "latitude": 45.3168,
        "longitude": -73.8659
      }
    }
  ],
  "client": {}
}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getedges-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|[EdgesModel](#schemaedgesmodel)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getedges-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="success">
This operation does not require authentication
</aside>

<h1 id="floatplane-rest-api-faqv2">FAQV2</h1>

Get FAQs.

## getFaqSections

<a id="opIdgetFaqSections"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v2/faq/list \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v2/faq/list HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v2/faq/list',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v2/faq/list',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v2/faq/list', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v2/faq/list', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v2/faq/list");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v2/faq/list", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v2/faq/list`

*Get Faq Sections*

Retrieve a list of FAQ sections to display to the user. Each section contains one or more FAQ items. This is normally accessible from https://www.floatplane.com/support. Note that the answers to the FAQs will contain HTML.

> Example responses

> 200 Response

```json
[
  {
    "faqs": [
      {
        "createdAt": "2019-10-03T18:45:49.157Z",
        "updatedAt": "2019-12-19T22:06:01.843Z",
        "id": "5d9641ddbced315cc7d9135f",
        "question": "How do you get the PewPew emote? ",
        "answer": "<p><span style=\"background-color: rgb(255, 255, 255); color: rgb(0, 0, 0);\">The PewPew emote was as an account reward for users who had succesfully set up a payment method on Floatplane.com before January 1st 2019.</span></p>",
        "status": "public",
        "link": "g-pewpew-emote",
        "order": 1,
        "faqSection": "5d9641d0b3e3285cfffe44a9"
      }
    ],
    "createdAt": "2019-10-03T18:45:36.840Z",
    "updatedAt": "2019-12-19T22:08:50.481Z",
    "id": "5d9641d0b3e3285cfffe44a9",
    "name": "General",
    "description": "For general questions about Floatplane",
    "status": "public",
    "order": 1
  },
  {
    "faqs": [
      {
        "createdAt": "2019-10-03T18:26:28.413Z",
        "updatedAt": "2020-01-28T03:23:15.918Z",
        "id": "5d963d54221c575ce366b7e7",
        "question": "Can you upgrade me to the LTT supporter (1080p) subscription? ",
        "answer": "<p>At this time there is no difference between the two channels in regards to features that are unlocked. The only difference between the two is the price.</p>",
        "status": "public",
        "link": "sub-u-payment",
        "order": 1,
        "faqSection": "5d8d1be612c2535c9dc067d1"
      }
    ],
    "createdAt": "2019-09-26T20:13:26.431Z",
    "updatedAt": "2020-01-28T03:24:33.443Z",
    "id": "5d8d1be612c2535c9dc067d1",
    "name": "Subscription and Payment ",
    "description": "Life isn't always about money but this section is. If you have a payment or subscription issue look here.  ",
    "status": "public",
    "order": 2
  }
]
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getfaqsections-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getfaqsections-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|[[FaqSectionModel](#schemafaqsectionmodel)]|false|none|none|
|» faqs|[object]|true|none|none|
|»» createdAt|string(date-time)|true|none|none|
|»» updatedAt|string(date-time)¦null|true|none|none|
|»» id|string|true|none|none|
|»» question|string|true|none|none|
|»» answer|string|true|none|This field may contain HTML that should be rendered.|
|»» status|string|true|none|none|
|»» link|string|true|none|none|
|»» order|number|true|none|none|
|»» faqSection|string|true|none|none|
|» createdAt|string(date-time)|true|none|none|
|» updatedAt|string(date-time)¦null|true|none|none|
|» id|string|true|none|none|
|» name|string|true|none|none|
|» description|string|true|none|none|
|» status|string|true|none|none|
|» order|number|true|none|none|

#### Enumerated Values

|Property|Value|
|---|---|
|status|public|
|status|public|

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="success">
This operation does not require authentication
</aside>

<h1 id="floatplane-rest-api-paymentsv2">PaymentsV2</h1>

User payment method/address/invoice management.

## listPaymentMethods

<a id="opIdlistPaymentMethods"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v2/payment/method/list \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v2/payment/method/list HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v2/payment/method/list',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v2/payment/method/list',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v2/payment/method/list', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v2/payment/method/list', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v2/payment/method/list");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v2/payment/method/list", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v2/payment/method/list`

*List Payment Methods*

Retrieve a list of saved payment methods for the user's account. Payment methods are how the user can pay for their subscription to creators on the platform.

> Example responses

> 200 Response

```json
[
  {
    "id": 54715,
    "payment_processor": 1,
    "default": true,
    "card": {
      "brand": "Visa",
      "last4": "1234",
      "exp_month": 11,
      "exp_year": 2028,
      "name": "Firstname Lastname"
    }
  }
]
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="listpaymentmethods-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="listpaymentmethods-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|[[PaymentMethodModel](#schemapaymentmethodmodel)]|false|none|none|
|» id|integer|true|none|none|
|» payment_processor|integer|true|none|none|
|» default|boolean|true|none|none|
|» card|object|true|none|none|
|»» brand|string|true|none|none|
|»» last4|string|true|none|none|
|»» exp_month|integer|true|none|none|
|»» exp_year|integer|true|none|none|
|»» name|string|true|none|none|

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## listAddresses

<a id="opIdlistAddresses"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v2/payment/address/list \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v2/payment/address/list HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v2/payment/address/list',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v2/payment/address/list',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v2/payment/address/list', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v2/payment/address/list', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v2/payment/address/list");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v2/payment/address/list", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v2/payment/address/list`

*List Addresses*

Retrieve a list of billing addresses saved to the user's account, to be used in conjunction with a payment method when purchasing subscriptions to creators.

> Example responses

> 200 Response

```json
[
  {
    "id": 44739,
    "customerName": "Firstname Lastname",
    "postalCode": "12345",
    "line1": "123 Main St",
    "city": "Metropolis",
    "region": "NY",
    "country": "US",
    "default": true
  }
]
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="listaddresses-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="listaddresses-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|[[PaymentAddressModel](#schemapaymentaddressmodel)]|false|none|none|
|» id|integer|true|none|none|
|» customerName|string|true|none|none|
|» postalCode|string|true|none|none|
|» line1|string|true|none|none|
|» city|string|true|none|none|
|» region|string|true|none|none|
|» country|string|true|none|none|
|» default|boolean|true|none|none|

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## listInvoices

<a id="opIdlistInvoices"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v2/payment/invoice/list \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v2/payment/invoice/list HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v2/payment/invoice/list',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v2/payment/invoice/list',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v2/payment/invoice/list', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v2/payment/invoice/list', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v2/payment/invoice/list");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v2/payment/invoice/list", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v2/payment/invoice/list`

*List Invoices*

Retrieve a list of paid or unpaid subscription invoices for the user.

> Example responses

> 200 Response

```json
{
  "invoices": [
    {
      "id": 1234567,
      "amountDue": 50,
      "amountTax": 0,
      "attemptCount": 0,
      "currency": "usd",
      "date": "2020-11-19T16:23:33.000Z",
      "dateDue": null,
      "periodStart": "2020-09-25T07:35:04.273Z",
      "periodEnd": "2021-09-25T07:35:04.273Z",
      "nextPaymentAttempt": "2020-09-25T07:35:04.273Z",
      "paid": true,
      "forgiven": false,
      "refunded": false,
      "subscriptions": [
        {
          "id": 1234567,
          "subscription": 12345,
          "periodStart": "2020-09-25T07:35:04.273Z",
          "periodEnd": "2021-09-25T07:35:04.273Z",
          "value": 50,
          "amountSubtotal": 50,
          "amountTotal": 50,
          "amountTax": 0,
          "plan": {
            "id": "5d48d0306825b5780db93d07",
            "title": "LTT Supporter (1080p)",
            "creator": {
              "id": "59f94c0bdd241b70349eb72b",
              "title": "LinusTechTips",
              "urlname": "linustechtips",
              "icon": {
                "width": 600,
                "height": 600,
                "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205.jpeg",
                "childImages": [
                  {
                    "width": 250,
                    "height": 250,
                    "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_250x250.jpeg"
                  },
                  {
                    "width": 100,
                    "height": 100,
                    "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_100x100.jpeg"
                  }
                ]
              }
            }
          }
        }
      ]
    }
  ]
}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="listinvoices-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|[PaymentInvoiceListV2Response](#schemapaymentinvoicelistv2response)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="listinvoices-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

<h1 id="floatplane-rest-api-socketv3">SocketV3</h1>

Socket subscriptions and connections.

## socketConnect

<a id="opIdsocketConnect"></a>

> Code samples

```shell
# You can also use wget
curl -X POST https://www.floatplane.com/api/v3/socket/connect \
  -H 'Accept: application/json'

```

```http
POST https://www.floatplane.com/api/v3/socket/connect HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/socket/connect',
{
  method: 'POST',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.post 'https://www.floatplane.com/api/v3/socket/connect',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.post('https://www.floatplane.com/api/v3/socket/connect', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('POST','https://www.floatplane.com/api/v3/socket/connect', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/socket/connect");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("POST");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("POST", "https://www.floatplane.com/api/v3/socket/connect", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`POST /api/v3/socket/connect`

*Connect*

Used in Socket.IO/WebSocket connections. See the AsyncAPI documentation for more information. This should not be used on a raw HTTP connection.

> Example responses

> 200 Response

```json
{}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="socketconnect-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="socketconnect-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## disconnectSocket

<a id="opIddisconnectSocket"></a>

> Code samples

```shell
# You can also use wget
curl -X POST https://www.floatplane.com/api/v3/socket/disconnect \
  -H 'Accept: application/json'

```

```http
POST https://www.floatplane.com/api/v3/socket/disconnect HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/socket/disconnect',
{
  method: 'POST',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.post 'https://www.floatplane.com/api/v3/socket/disconnect',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.post('https://www.floatplane.com/api/v3/socket/disconnect', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('POST','https://www.floatplane.com/api/v3/socket/disconnect', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/socket/disconnect");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("POST");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("POST", "https://www.floatplane.com/api/v3/socket/disconnect", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`POST /api/v3/socket/disconnect`

*Disconnect*

Used in Socket.IO/WebSocket connections. See the AsyncAPI documentation for more information. This should not be used on a raw HTTP connection.

> Example responses

> 200 Response

```json
{}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="disconnectsocket-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="disconnectsocket-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

<h1 id="floatplane-rest-api-subscriptionsv3">SubscriptionsV3</h1>

Get user subscriptions.

## listUserSubscriptionsV3

<a id="opIdlistUserSubscriptionsV3"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v3/user/subscriptions \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v3/user/subscriptions HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/user/subscriptions',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v3/user/subscriptions',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v3/user/subscriptions', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v3/user/subscriptions', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/user/subscriptions");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v3/user/subscriptions", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v3/user/subscriptions`

*List User Subscriptions*

Retrieve a list of all active subscriptions for the user.

> Example responses

> 200 Response

```json
[
  {
    "startDate": "2020-09-25T07:35:04.273Z",
    "endDate": "2021-09-25T07:35:04.273Z",
    "paymentID": 12345,
    "interval": "year",
    "paymentCancelled": false,
    "plan": {
      "id": "5d48d0306825b5780db93d07",
      "title": "LTT Supporter (1080p)",
      "description": "Includes:\n- Early access (when possible)\n- Live Streaming\n- Behind-the-scenes, cutting room floor & exclusives\n\nNOTE: Tech Quickie and TechLinked are included for now, but will move to their own Floatplane pages in the future",
      "price": "5.00",
      "priceYearly": "50.00",
      "currency": "usd",
      "logo": null,
      "interval": "month",
      "featured": true,
      "allowGrandfatheredAccess": false,
      "discordServers": [],
      "discordRoles": []
    },
    "creator": "59f94c0bdd241b70349eb72b"
  }
]
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="listusersubscriptionsv3-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Subscriptions returned|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="listusersubscriptionsv3-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|[[UserSubscriptionModel](#schemausersubscriptionmodel)]|false|none|none|
|» startDate|string(date-time)¦null|true|none|none|
|» endDate|string(date-time)¦null|true|none|none|
|» paymentID|integer¦null|true|none|none|
|» interval|string|true|none|none|
|» paymentCancelled|boolean¦null|false|none|none|
|» plan|[SubscriptionPlanModel](#schemasubscriptionplanmodel)|true|none|none|
|»» id|string|true|none|none|
|»» title|string|true|none|none|
|»» description|string|true|none|none|
|»» price|string¦null|true|none|none|
|»» priceYearly|string¦null|false|none|none|
|»» currency|string|true|none|none|
|»» logo|string¦null|true|none|none|
|»» interval|string|true|none|none|
|»» featured|boolean|true|none|none|
|»» allowGrandfatheredAccess|boolean¦null|false|none|none|
|»» discordServers|[[DiscordServerModel](#schemadiscordservermodel)]|true|none|none|
|»»» id|string|true|none|none|
|»»» guildName|string|true|none|none|
|»»» guildIcon|string|true|none|none|
|»»» inviteLink|string(uri)¦null|true|none|none|
|»»» inviteMode|string|true|none|none|
|»» discordRoles|[[DiscordRoleModel](#schemadiscordrolemodel)]|true|none|none|
|»»» server|string|true|none|none|
|»»» roleName|string|true|none|none|
|» creator|string|true|none|none|

#### Links

**getContent** => <a href="#opIdgetMultiCreatorBlogPosts">getMultiCreatorBlogPosts</a>

|Parameter|Expression|
|---|---|
|ids|$response.body#/0/creator|

**getCreators** => <a href="#opIdgetInfo">getInfo</a>

|Parameter|Expression|
|---|---|
|creatorGUID|$response.body#/0/creator|

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

<h1 id="floatplane-rest-api-userv2">UserV2</h1>

User discovery and profile management.

## getUserInfo

<a id="opIdgetUserInfo"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v2/user/info?id=string \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v2/user/info?id=string HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v2/user/info?id=string',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v2/user/info',
  params: {
  'id' => 'array[string]'
}, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v2/user/info', params={
  'id': [
  "string"
]
}, headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v2/user/info', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v2/user/info?id=string");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v2/user/info", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v2/user/info`

*Info*

Retrieve more detailed information about one or more users from their identifiers.

<h3 id="getuserinfo-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id|query|array[string]|true|The GUID identifer(s) of the user(s) to be retrieved.|

> Example responses

> 200 Response

```json
{
  "users": [
    {
      "id": "59f94c0bdd241b70349eb723",
      "user": {
        "id": "59f94c0bdd241b70349eb723",
        "username": "Linus",
        "profileImage": {
          "width": 600,
          "height": 600,
          "path": "https://pbs.floatplane.com/profile_images/59f94c0bdd241b70349eb723/013264939123424_1535577174346.jpeg",
          "childImages": [
            {
              "width": 100,
              "height": 100,
              "path": "https://pbs.floatplane.com/profile_images/59f94c0bdd241b70349eb723/013264939123424_1535577174346_100x100.jpeg"
            },
            {
              "width": 250,
              "height": 250,
              "path": "https://pbs.floatplane.com/profile_images/59f94c0bdd241b70349eb723/013264939123424_1535577174346_250x250.jpeg"
            }
          ]
        }
      }
    }
  ]
}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getuserinfo-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Results of the user search|[UserInfoV2Response](#schemauserinfov2response)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getuserinfo-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## getUserInfoByName

<a id="opIdgetUserInfoByName"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v2/user/named?username=string \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v2/user/named?username=string HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v2/user/named?username=string',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v2/user/named',
  params: {
  'username' => 'array[string]'
}, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v2/user/named', params={
  'username': [
  "string"
]
}, headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v2/user/named', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v2/user/named?username=string");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v2/user/named", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v2/user/named`

*Get Info By Name*

Retrieve more detailed information about one or more users from their usernames.

<h3 id="getuserinfobyname-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|username|query|array[string]|true|The username(s) of the user(s) to be retrieved.|

> Example responses

> 200 Response

```json
{
  "users": [
    {
      "id": "0123456789abcdef01234567",
      "user": {
        "id": "0123456789abcdef01234567",
        "username": "my_username",
        "profileImage": {
          "width": 512,
          "height": 512,
          "path": "https://pbs.floatplane.com/profile_images/default/user12.png",
          "childImages": [
            {
              "width": 250,
              "height": 250,
              "path": "https://pbs.floatplane.com/profile_images/default/user12_250x250.png"
            },
            {
              "width": 100,
              "height": 100,
              "path": "https://pbs.floatplane.com/profile_images/default/user12_100x100.png"
            }
          ]
        },
        "email": "testemail@example.com",
        "displayName": "Firstname Lastname"
      }
    }
  ]
}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getuserinfobyname-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Results of the user search|[UserNamedV2Response](#schemausernamedv2response)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getuserinfobyname-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## getSecurity

<a id="opIdgetSecurity"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v2/user/security \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v2/user/security HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v2/user/security',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v2/user/security',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v2/user/security', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v2/user/security', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v2/user/security");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v2/user/security", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v2/user/security`

*Get Security*

Retrieve information about the current security configuration for the user.

> Example responses

> 200 Response

```json
{
  "twofactorEnabled": true,
  "twofactorBackupCodeEnabled": true
}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getsecurity-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Current security settings|[UserSecurityV2Response](#schemausersecurityv2response)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getsecurity-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## userCreatorBanStatus

<a id="opIduserCreatorBanStatus"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v2/user/ban/status?creator=string \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v2/user/ban/status?creator=string HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v2/user/ban/status?creator=string',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v2/user/ban/status',
  params: {
  'creator' => 'string'
}, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v2/user/ban/status', params={
  'creator': 'string'
}, headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v2/user/ban/status', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v2/user/ban/status?creator=string");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v2/user/ban/status", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v2/user/ban/status`

*User Creator Ban Status*

Determine whether or not the user is banned for a given creator.

<h3 id="usercreatorbanstatus-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|creator|query|string|true|The GUID of the creator being queried.|

> Example responses

> 200 Response

```json
true
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="usercreatorbanstatus-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Whether the user is banned or not|boolean|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="usercreatorbanstatus-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

<h1 id="floatplane-rest-api-userv3">UserV3</h1>

User discovery and profile management.

## getActivityFeedV3

<a id="opIdgetActivityFeedV3"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v3/user/activity?id=string \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v3/user/activity?id=string HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/user/activity?id=string',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v3/user/activity',
  params: {
  'id' => 'string'
}, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v3/user/activity', params={
  'id': 'string'
}, headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v3/user/activity', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/user/activity?id=string");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v3/user/activity", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v3/user/activity`

*Get Activity Feed*

Retrieve recent activity for a user, such as comments and other interactions they have made on posts for creators.

<h3 id="getactivityfeedv3-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id|query|string|true|The GUID of the user being queried.|

> Example responses

> 200 Response

```json
{
  "activity": [
    {
      "time": "2021-10-09T08:12:51.290Z",
      "comment": "This is the text of the comment being posted",
      "postTitle": "TL: Facebook Does Not Care.",
      "postId": "j7KjCaKrtV",
      "creatorTitle": "LinusTechTips",
      "creatorUrl": "linustechtips"
    }
  ],
  "visibility": "public"
}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getactivityfeedv3-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Activity returned|[UserActivityV3Response](#schemauseractivityv3response)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getactivityfeedv3-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## getSelf

<a id="opIdgetSelf"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v3/user/self \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v3/user/self HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/user/self',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v3/user/self',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v3/user/self', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v3/user/self', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/user/self");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v3/user/self", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v3/user/self`

*Get Self*

Retrieve more detailed information about the user, including their name and email.

> Example responses

> 200 Response

```json
{
  "id": "0123456789abcdef01234567",
  "username": "my_username",
  "profileImage": {
    "width": 512,
    "height": 512,
    "path": "https://pbs.floatplane.com/profile_images/default/user12.png",
    "childImages": [
      {
        "width": 250,
        "height": 250,
        "path": "https://pbs.floatplane.com/profile_images/default/user12_250x250.png"
      },
      {
        "width": 100,
        "height": 100,
        "path": "https://pbs.floatplane.com/profile_images/default/user12_100x100.png"
      }
    ]
  },
  "email": "testemail@example.com",
  "displayName": "Firstname Lastname",
  "creators": [],
  "scheduledDeletionDate": null
}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getself-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Information returned|[UserSelfV3Response](#schemauserselfv3response)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getself-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## getUserNotificationSettingsV3

<a id="opIdgetUserNotificationSettingsV3"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v3/user/notification/list \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v3/user/notification/list HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/user/notification/list',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v3/user/notification/list',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v3/user/notification/list', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v3/user/notification/list', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/user/notification/list");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v3/user/notification/list", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v3/user/notification/list`

*Get User Notification Settings*

Retrieve notification details for a user. The details are split into seperate settings for each subscribed creator.

> Example responses

> 200 Response

```json
[
  {
    "creator": {
      "id": "59f94c0bdd241b70349eb72b",
      "owner": "59f94c0bdd241b70349eb723",
      "title": "LinusTechTips",
      "urlname": "linustechtips",
      "description": "We make entertaining videos about technology, including tech reviews, showcases and other content.",
      "about": "# We're LinusTechTips\nWe make videos and stuff, cool eh?",
      "category": "59f94c0bdd241b70349eb727",
      "cover": null,
      "icon": {
        "width": 16,
        "height": 16,
        "path": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQAQMAAAAlPW0iAAAAA1BMVEUfLTy/Rf//AAAAC0lEQVQI12MgEQAAADAAAWV61nwAAAAASUVORK5CYII="
      },
      "liveStream": null,
      "subscriptionPlans": null,
      "discoverable": true,
      "subscriberCountDisplay": "total",
      "incomeDisplay": false
    },
    "userNotificationSetting": {
      "createdAt": "2020-09-25T07:35:04.273Z",
      "updatedAt": "2021-10-07T14:16:56.561Z",
      "id": "abcdef0123456789abcdef01",
      "contentEmail": false,
      "contentFirebase": true,
      "creatorMessageEmail": false,
      "user": "0123456789abcdef01234567",
      "creator": "59f94c0bdd241b70349eb72b"
    }
  }
]
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getusernotificationsettingsv3-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Notifications returned|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getusernotificationsettingsv3-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|[[UserNotificationModel](#schemausernotificationmodel)]|false|none|none|
|» creator|[CreatorModelV2](#schemacreatormodelv2)|true|none|none|
|»» id|string|true|none|none|
|»» owner|string|true|none|none|
|»» title|string|true|none|none|
|»» urlname|string|true|none|Shown in the browser URL, and used in `/creator/named` queries.|
|»» description|string|true|none|none|
|»» about|string|true|none|none|
|»» category|string|true|none|none|
|»» cover|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»» width|integer|true|none|none|
|»»» height|integer|true|none|none|
|»»» path|string(uri)|true|none|none|
|»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»»»» width|integer|true|none|none|
|»»»» height|integer|true|none|none|
|»»»» path|string(uri)|true|none|none|
|»» icon|[ImageModel](#schemaimagemodel)|true|none|none|
|»»» width|integer|true|none|none|
|»»» height|integer|true|none|none|
|»»» path|string(uri)|true|none|none|
|»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»» liveStream|[LiveStreamModel](#schemalivestreammodel)¦null|true|none|none|
|»»» id|string|true|none|none|
|»»» title|string|true|none|none|
|»»» description|string|true|none|none|
|»»» thumbnail|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»»» width|integer|true|none|none|
|»»»» height|integer|true|none|none|
|»»»» path|string(uri)|true|none|none|
|»»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»»» owner|string|true|none|none|
|»»» channel|string|false|none|The creator channel this livestream belongs to.|
|»»» streamPath|string|true|none|none|
|»»» offline|object|true|none|none|
|»»»» title|string¦null|true|none|none|
|»»»» description|string¦null|true|none|none|
|»»»» thumbnail|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»»»» width|integer|true|none|none|
|»»»»» height|integer|true|none|none|
|»»»»» path|string(uri)|true|none|none|
|»»»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»» subscriptionPlans|[[SubscriptionPlanModel](#schemasubscriptionplanmodel)]¦null|true|none|none|
|»»» id|string|true|none|none|
|»»» title|string|true|none|none|
|»»» description|string|true|none|none|
|»»» price|string¦null|true|none|none|
|»»» priceYearly|string¦null|false|none|none|
|»»» currency|string|true|none|none|
|»»» logo|string¦null|true|none|none|
|»»» interval|string|true|none|none|
|»»» featured|boolean|true|none|none|
|»»» allowGrandfatheredAccess|boolean¦null|false|none|none|
|»»» discordServers|[[DiscordServerModel](#schemadiscordservermodel)]|true|none|none|
|»»»» id|string|true|none|none|
|»»»» guildName|string|true|none|none|
|»»»» guildIcon|string|true|none|none|
|»»»» inviteLink|string(uri)¦null|true|none|none|
|»»»» inviteMode|string|true|none|none|
|»»» discordRoles|[[DiscordRoleModel](#schemadiscordrolemodel)]|true|none|none|
|»»»» server|string|true|none|none|
|»»»» roleName|string|true|none|none|
|»» discoverable|boolean|true|none|none|
|»» subscriberCountDisplay|string|true|none|none|
|»» incomeDisplay|boolean|true|none|none|
|»» defaultChannel|string|false|none|none|
|» userNotificationSetting|object|true|none|none|
|»» createdAt|string(date-time)|false|none|none|
|»» updatedAt|string(date-time)¦null|false|none|none|
|»» id|string|false|none|none|
|»» contentEmail|boolean|true|none|none|
|»» contentFirebase|boolean|true|none|none|
|»» creatorMessageEmail|boolean|true|none|none|
|»» user|string|true|none|none|
|»» creator|string|true|none|none|

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## updateUserNotificationSettingsV3

<a id="opIdupdateUserNotificationSettingsV3"></a>

> Code samples

```shell
# You can also use wget
curl -X POST https://www.floatplane.com/api/v3/user/notification/update \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json'

```

```http
POST https://www.floatplane.com/api/v3/user/notification/update HTTP/1.1
Host: www.floatplane.com
Content-Type: application/json
Accept: application/json

```

```javascript
const inputBody = '{
  "creator": "string",
  "property": "contentEmail",
  "newValue": true
}';
const headers = {
  'Content-Type':'application/json',
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/user/notification/update',
{
  method: 'POST',
  body: inputBody,
  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Content-Type' => 'application/json',
  'Accept' => 'application/json'
}

result = RestClient.post 'https://www.floatplane.com/api/v3/user/notification/update',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Content-Type': 'application/json',
  'Accept': 'application/json'
}

r = requests.post('https://www.floatplane.com/api/v3/user/notification/update', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Content-Type' => 'application/json',
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('POST','https://www.floatplane.com/api/v3/user/notification/update', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/user/notification/update");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("POST");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Content-Type": []string{"application/json"},
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("POST", "https://www.floatplane.com/api/v3/user/notification/update", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`POST /api/v3/user/notification/update`

*Update User Notification Settings*

Enable or disable email or push notifications for a specific creator.

> Body parameter

```json
{
  "creator": "string",
  "property": "contentEmail",
  "newValue": true
}
```

<h3 id="updateusernotificationsettingsv3-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|[UserNotificationUpdateV3PostRequest](#schemausernotificationupdatev3postrequest)|true|none|

> Example responses

> 200 Response

```json
true
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="updateusernotificationsettingsv3-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Whether or not the update was successful|boolean|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="updateusernotificationsettingsv3-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

<h1 id="floatplane-rest-api-commentv3">CommentV3</h1>

Comment retrieval, posting, and interacting.

## postComment

<a id="opIdpostComment"></a>

> Code samples

```shell
# You can also use wget
curl -X POST https://www.floatplane.com/api/v3/comment \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json'

```

```http
POST https://www.floatplane.com/api/v3/comment HTTP/1.1
Host: www.floatplane.com
Content-Type: application/json
Accept: application/json

```

```javascript
const inputBody = '{
  "blogPost": "string",
  "text": "string"
}';
const headers = {
  'Content-Type':'application/json',
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/comment',
{
  method: 'POST',
  body: inputBody,
  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Content-Type' => 'application/json',
  'Accept' => 'application/json'
}

result = RestClient.post 'https://www.floatplane.com/api/v3/comment',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Content-Type': 'application/json',
  'Accept': 'application/json'
}

r = requests.post('https://www.floatplane.com/api/v3/comment', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Content-Type' => 'application/json',
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('POST','https://www.floatplane.com/api/v3/comment', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/comment");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("POST");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Content-Type": []string{"application/json"},
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("POST", "https://www.floatplane.com/api/v3/comment", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`POST /api/v3/comment`

*Post Comment*

Post a new comment to a blog post object.

> Body parameter

```json
{
  "blogPost": "string",
  "text": "string"
}
```

<h3 id="postcomment-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|[CommentV3PostRequest](#schemacommentv3postrequest)|true|none|

> Example responses

> 200 Response

```json
{
  "id": "8d575af6834343b166d0562a",
  "blogPost": "j7KjCaKrtV",
  "user": {
    "id": "0123456789abcdef01234567",
    "username": "my_username",
    "profileImage": {
      "width": 512,
      "height": 512,
      "path": "https://pbs.floatplane.com/profile_images/default/user12.png",
      "childImages": [
        {
          "width": 250,
          "height": 250,
          "path": "https://pbs.floatplane.com/profile_images/default/user12_250x250.png"
        },
        {
          "width": 100,
          "height": 100,
          "path": "https://pbs.floatplane.com/profile_images/default/user12_100x100.png"
        }
      ]
    }
  },
  "text": "This is the text of the comment being posted",
  "replying": null,
  "postDate": "2021-10-09T08:12:51.290Z",
  "editDate": "2021-10-09T08:12:51.290Z",
  "likes": 0,
  "dislikes": 0,
  "score": 0,
  "interactionCounts": {
    "like": 0,
    "dislike": 0
  }
}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="postcomment-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Commented posted successfully, returning comment details|[CommentV3PostResponse](#schemacommentv3postresponse)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="postcomment-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## getComments

<a id="opIdgetComments"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v3/comment?blogPost=string&limit=0 \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v3/comment?blogPost=string&limit=0 HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/comment?blogPost=string&limit=0',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v3/comment',
  params: {
  'blogPost' => 'string',
'limit' => 'integer'
}, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v3/comment', params={
  'blogPost': 'string',  'limit': '0'
}, headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v3/comment', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/comment?blogPost=string&limit=0");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v3/comment", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v3/comment`

*Get Comments*

Get comments for a blog post object. Note that replies to each comment tend to be limited to 3. The extra replies can be retrieved via `getCommentReplies`. The difference in `$response.body#/0/totalReplies` and `$response.body#/0/replies`'s length can determine if more comments need to be loaded.

<h3 id="getcomments-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|blogPost|query|string|true|Which blog post to retrieve comments for.|
|limit|query|integer|true|The maximum number of comments to return. This should be set to 20 by default.|
|fetchAfter|query|string|false|When loading more comments on a blog post, this is used to determine which which comments to skip. This is a GUID of the last comment from the previous call to `getComments`.|

> Example responses

> 200 Response

```json
[
  {
    "id": "00c5ab7379e746b24a76634b",
    "blogPost": "Dw2ms0AgL8",
    "user": {
      "id": "ff0a479639c60f3a8cd18d8b",
      "username": "some_username",
      "profileImage": {
        "width": 512,
        "height": 512,
        "path": "https://pbs.floatplane.com/profile_images/default/user10.png",
        "childImages": [
          {
            "width": 250,
            "height": 250,
            "path": "https://pbs.floatplane.com/profile_images/default/user10_250x250.png"
          },
          {
            "width": 100,
            "height": 100,
            "path": "https://pbs.floatplane.com/profile_images/default/user10_100x100.png"
          }
        ]
      }
    },
    "text": "This is my comment text, I really like this video.",
    "replying": null,
    "postDate": "2021-10-09T14:58:34.829Z",
    "editDate": "2021-10-09T14:58:34.829Z",
    "likes": 0,
    "dislikes": 0,
    "score": 0,
    "interactionCounts": {
      "like": 0,
      "dislike": 0
    },
    "totalReplies": 0,
    "replies": [],
    "userInteraction": null
  }
]
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getcomments-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - All comments returned for the query parameters|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getcomments-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|[[CommentModel](#schemacommentmodel)]|false|none|none|
|» id|string|true|none|none|
|» blogPost|string|true|none|none|
|» user|[UserModel](#schemausermodel)|true|none|Represents some basic information of a user (id, username, and profile image).|
|»» id|string|true|none|none|
|»» username|string|true|none|none|
|»» profileImage|[ImageModel](#schemaimagemodel)|true|none|none|
|»»» width|integer|true|none|none|
|»»» height|integer|true|none|none|
|»»» path|string(uri)|true|none|none|
|»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»»»» width|integer|true|none|none|
|»»»» height|integer|true|none|none|
|»»»» path|string(uri)|true|none|none|
|» text|string|true|none|none|
|» replying|string¦null|true|none|none|
|» postDate|string(date-time)|true|none|none|
|» editDate|string(date-time)¦null|true|none|none|
|» pinDate|string(date-time)¦null|false|none|none|
|» editCount|integer|true|none|none|
|» isEdited|boolean|true|none|none|
|» likes|integer|true|none|none|
|» dislikes|integer|true|none|none|
|» score|integer|true|none|none|
|» interactionCounts|object|true|none|none|
|»» like|integer|true|none|none|
|»» dislike|integer|true|none|none|
|» totalReplies|integer|false|none|none|
|» replies|[[CommentModel](#schemacommentmodel)]|false|none|This is present (but possibly empty) for top-level comments. This is never present for reply comments.|
|» userInteraction|[string]¦null|true|none|none|

#### Links

**getMoreComments** => <a href="#opIdgetcomments">getcomments</a>

|Parameter|Expression|
|---|---|
|fetchAfter|$response.body#/19/id|

**getMoreReplies** => <a href="#opIdgetCommentReplies">getCommentReplies</a>

|Parameter|Expression|
|---|---|

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## getCommentReplies

<a id="opIdgetCommentReplies"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v3/comment/replies?comment=string&blogPost=string&limit=0&rid=string \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v3/comment/replies?comment=string&blogPost=string&limit=0&rid=string HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/comment/replies?comment=string&blogPost=string&limit=0&rid=string',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v3/comment/replies',
  params: {
  'comment' => 'string',
'blogPost' => 'string',
'limit' => 'integer',
'rid' => 'string'
}, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v3/comment/replies', params={
  'comment': 'string',  'blogPost': 'string',  'limit': '0',  'rid': 'string'
}, headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v3/comment/replies', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/comment/replies?comment=string&blogPost=string&limit=0&rid=string");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v3/comment/replies", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v3/comment/replies`

*Get Comment Replies*

Retrieve more replies from a comment.

<h3 id="getcommentreplies-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|comment|query|string|true|The identifer of the comment from which to retrieve replies.|
|blogPost|query|string|true|The identifer of the blog post the `comment` belongs to.|
|limit|query|integer|true|How many replies to retrieve.|
|rid|query|string|true|The identifer of the last reply in the reply chain.|

> Example responses

> 200 Response

```json
[
  {
    "id": "1234567890abcdef",
    "blogPost": "p3OSnFmsR3",
    "user": {
      "id": "abcdef1234567890",
      "username": "the_username",
      "profileImage": {
        "width": 225,
        "height": 225,
        "path": "https://pbs.floatplane.com/profile_images/default/user12.png",
        "childImages": [
          {
            "width": 100,
            "height": 100,
            "path": "https://pbs.floatplane.com/profile_images/default/user12.png"
          }
        ]
      }
    },
    "text": "This is my reply text",
    "replying": "1234567890abcdef0",
    "postDate": "2021-12-17T06:57:33.152Z",
    "editDate": "2021-12-17T06:57:33.152Z",
    "likes": 0,
    "dislikes": 0,
    "score": 0,
    "interactionCounts": {
      "like": 0,
      "dislike": 0
    },
    "userInteraction": null
  }
]
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getcommentreplies-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getcommentreplies-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|[[CommentModel](#schemacommentmodel)]|false|none|none|
|» id|string|true|none|none|
|» blogPost|string|true|none|none|
|» user|[UserModel](#schemausermodel)|true|none|Represents some basic information of a user (id, username, and profile image).|
|»» id|string|true|none|none|
|»» username|string|true|none|none|
|»» profileImage|[ImageModel](#schemaimagemodel)|true|none|none|
|»»» width|integer|true|none|none|
|»»» height|integer|true|none|none|
|»»» path|string(uri)|true|none|none|
|»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»»»» width|integer|true|none|none|
|»»»» height|integer|true|none|none|
|»»»» path|string(uri)|true|none|none|
|» text|string|true|none|none|
|» replying|string¦null|true|none|none|
|» postDate|string(date-time)|true|none|none|
|» editDate|string(date-time)¦null|true|none|none|
|» pinDate|string(date-time)¦null|false|none|none|
|» editCount|integer|true|none|none|
|» isEdited|boolean|true|none|none|
|» likes|integer|true|none|none|
|» dislikes|integer|true|none|none|
|» score|integer|true|none|none|
|» interactionCounts|object|true|none|none|
|»» like|integer|true|none|none|
|»» dislike|integer|true|none|none|
|» totalReplies|integer|false|none|none|
|» replies|[[CommentModel](#schemacommentmodel)]|false|none|This is present (but possibly empty) for top-level comments. This is never present for reply comments.|
|» userInteraction|[string]¦null|true|none|none|

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## likeComment

<a id="opIdlikeComment"></a>

> Code samples

```shell
# You can also use wget
curl -X POST https://www.floatplane.com/api/v3/comment/like \
  -H 'Content-Type: application/json' \
  -H 'Accept: text/plain'

```

```http
POST https://www.floatplane.com/api/v3/comment/like HTTP/1.1
Host: www.floatplane.com
Content-Type: application/json
Accept: text/plain

```

```javascript
const inputBody = '{
  "comment": "string",
  "blogPost": "string"
}';
const headers = {
  'Content-Type':'application/json',
  'Accept':'text/plain'
};

fetch('https://www.floatplane.com/api/v3/comment/like',
{
  method: 'POST',
  body: inputBody,
  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Content-Type' => 'application/json',
  'Accept' => 'text/plain'
}

result = RestClient.post 'https://www.floatplane.com/api/v3/comment/like',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Content-Type': 'application/json',
  'Accept': 'text/plain'
}

r = requests.post('https://www.floatplane.com/api/v3/comment/like', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Content-Type' => 'application/json',
    'Accept' => 'text/plain',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('POST','https://www.floatplane.com/api/v3/comment/like', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/comment/like");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("POST");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Content-Type": []string{"application/json"},
        "Accept": []string{"text/plain"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("POST", "https://www.floatplane.com/api/v3/comment/like", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`POST /api/v3/comment/like`

*Like Comment*

Like a comment on a blog post.

> Body parameter

```json
{
  "comment": "string",
  "blogPost": "string"
}
```

<h3 id="likecomment-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|[CommentLikeV3PostRequest](#schemacommentlikev3postrequest)|true|none|

> Example responses

> 200 Response

```
"like"
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="likecomment-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Comment successfully liked|string|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="likecomment-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## dislikeComment

<a id="opIddislikeComment"></a>

> Code samples

```shell
# You can also use wget
curl -X POST https://www.floatplane.com/api/v3/comment/dislike \
  -H 'Content-Type: application/json' \
  -H 'Accept: text/plain'

```

```http
POST https://www.floatplane.com/api/v3/comment/dislike HTTP/1.1
Host: www.floatplane.com
Content-Type: application/json
Accept: text/plain

```

```javascript
const inputBody = '{
  "comment": "string",
  "blogPost": "string"
}';
const headers = {
  'Content-Type':'application/json',
  'Accept':'text/plain'
};

fetch('https://www.floatplane.com/api/v3/comment/dislike',
{
  method: 'POST',
  body: inputBody,
  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Content-Type' => 'application/json',
  'Accept' => 'text/plain'
}

result = RestClient.post 'https://www.floatplane.com/api/v3/comment/dislike',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Content-Type': 'application/json',
  'Accept': 'text/plain'
}

r = requests.post('https://www.floatplane.com/api/v3/comment/dislike', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Content-Type' => 'application/json',
    'Accept' => 'text/plain',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('POST','https://www.floatplane.com/api/v3/comment/dislike', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/comment/dislike");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("POST");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Content-Type": []string{"application/json"},
        "Accept": []string{"text/plain"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("POST", "https://www.floatplane.com/api/v3/comment/dislike", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`POST /api/v3/comment/dislike`

*Dislike Comment*

Dislike a comment on a blog post.

> Body parameter

```json
{
  "comment": "string",
  "blogPost": "string"
}
```

<h3 id="dislikecomment-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|[CommentLikeV3PostRequest](#schemacommentlikev3postrequest)|true|none|

> Example responses

> 200 Response

```
"dislike"
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="dislikecomment-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Comment successfully disliked|string|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="dislikecomment-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

<h1 id="floatplane-rest-api-contentv3">ContentV3</h1>

Content retrieval and interacting.

## getCreatorBlogPosts

<a id="opIdgetCreatorBlogPosts"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v3/content/creator?id=string \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v3/content/creator?id=string HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/content/creator?id=string',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v3/content/creator',
  params: {
  'id' => 'string'
}, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v3/content/creator', params={
  'id': 'string'
}, headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v3/content/creator', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/content/creator?id=string");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v3/content/creator", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v3/content/creator`

*Get Creator Blog Posts*

Retrieve a paginated list of blog posts from a creator. Or search for blog posts from a creator.

Example query: https://www.floatplane.com/api/v3/content/creator?id=59f94c0bdd241b70349eb72b&fromDate=2021-07-24T07:00:00.001Z&toDate=2022-07-27T06:59:59.099Z&hasVideo=true&hasAudio=true&hasPicture=false&hasText=false&fromDuration=1020&toDuration=9900&sort=DESC&search=thor&tags[0]=tjm

<h3 id="getcreatorblogposts-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id|query|string|true|The GUID of the creator to retrieve posts from.|
|channel|query|string|false|The id of a creator's specific channel from which to retrieve posts.|
|limit|query|integer|false|The maximum number of posts to return.|
|fetchAfter|query|integer|false|The number of posts to skip. Usually a multiple of `limit`, to get the next "page" of results.|
|search|query|string|false|Search filter to look for specific posts.|
|tags|query|array[string]|false|An array of tags to search against, possibly in addition to `search`.|
|hasVideo|query|boolean|false|If true, include blog posts with video attachments.|
|hasAudio|query|boolean|false|If true, include blog posts with audio attachments.|
|hasPicture|query|boolean|false|If true, include blog posts with picture attachments.|
|hasText|query|boolean|false|If true, only include blog posts that are text-only. Text-only posts are ones without any attachments, such as video, audio, picture, and gallery.|
|sort|query|string|false|`DESC` = Newest First. `ASC` = Oldest First.|
|fromDuration|query|integer|false|Include video posts where the duration of the video is at minimum `fromDuration` seconds long. Usually in multiples of 60 seconds. Implies `hasVideo=true`.|
|toDuration|query|integer|false|Include video posts where the duration of the video is at maximum `toDuration` seconds long. Usually in multiples of 60 seconds. Implies `hasVideo=true`.|
|fromDate|query|string(date-time)|false|Include posts where the publication date is on or after this filter date.|
|toDate|query|string(date-time)|false|Include posts where the publication date is on or before this filter date.|

#### Detailed descriptions

**hasText**: If true, only include blog posts that are text-only. Text-only posts are ones without any attachments, such as video, audio, picture, and gallery.

This filter and `hasVideo`, `hasAudio`, and `hasPicture` should be mutually exclusive. That is, if `hasText` is true then the other three should all be false. Conversely, if any of the other three are true, then `hasText` should be false. Otherwise, the filter would produce no results.

#### Enumerated Values

|Parameter|Value|
|---|---|
|sort|ASC|
|sort|DESC|

> Example responses

> 200 Response

```json
[
  {
    "id": "Dw2ms0AgL8",
    "guid": "Dw2ms0AgL8",
    "title": "Livestream VOD – October 9, 2021 @ 07:18 – First Linux Stream",
    "text": "<p>chat on Twitch</p>",
    "type": "blogPost",
    "tags": [
      "test"
    ],
    "attachmentOrder": [
      "TViGzkuIic"
    ],
    "metadata": {
      "hasVideo": true,
      "videoCount": 1,
      "videoDuration": 5689,
      "hasAudio": false,
      "audioCount": 0,
      "audioDuration": 0,
      "hasPicture": false,
      "pictureCount": 0,
      "hasGallery": false,
      "galleryCount": 0,
      "isFeatured": false
    },
    "releaseDate": "2021-10-09T09:29:00.039Z",
    "likes": 41,
    "dislikes": 0,
    "score": 41,
    "comments": 28,
    "creator": {
      "id": "59f94c0bdd241b70349eb72b",
      "owner": {
        "id": "59f94c0bdd241b70349eb723",
        "username": "Linus"
      },
      "title": "LinusTechTips",
      "urlname": "linustechtips",
      "description": "We make entertaining videos about technology, including tech reviews, showcases and other content.",
      "about": "# We're LinusTechTips\nWe make videos and stuff, cool eh?",
      "category": {
        "title": "Technology"
      },
      "cover": {
        "width": 1990,
        "height": 519,
        "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867.jpeg",
        "childImages": [
          {
            "width": 1245,
            "height": 325,
            "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867_1245x325.jpeg"
          }
        ]
      },
      "icon": {
        "width": 600,
        "height": 600,
        "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205.jpeg",
        "childImages": [
          {
            "width": 250,
            "height": 250,
            "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_250x250.jpeg"
          },
          {
            "width": 100,
            "height": 100,
            "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_100x100.jpeg"
          }
        ]
      },
      "liveStream": {
        "id": "5c13f3c006f1be15e08e05c0",
        "title": "First Linux Stream",
        "description": "<p>chat on Twitch</p>",
        "thumbnail": {
          "width": 1200,
          "height": 675,
          "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412.jpeg",
          "childImages": [
            {
              "width": 400,
              "height": 225,
              "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412_400x225.jpeg"
            }
          ]
        },
        "owner": "59f94c0bdd241b70349eb72b",
        "streamPath": "/api/video/v1/us-east-1.758417551536.channel.yKkxur4ukc0B.m3u8",
        "offline": {
          "title": "Offline",
          "description": "We're offline for now – please check back later!",
          "thumbnail": {
            "width": 1920,
            "height": 1080,
            "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026.jpeg",
            "childImages": [
              {
                "width": 400,
                "height": 225,
                "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_400x225.jpeg"
              },
              {
                "width": 1200,
                "height": 675,
                "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_1200x675.jpeg"
              }
            ]
          }
        }
      },
      "subscriptionPlans": [
        {
          "id": "5d48d0306825b5780db93d07",
          "title": "LTT Supporter (1080p)",
          "description": "Includes:\n- Early access (when possible)\n- Live Streaming\n- Behind-the-scenes, cutting room floor & exclusives\n\nNOTE: Tech Quickie and TechLinked are included for now, but will move to their own Floatplane pages in the future",
          "price": "5.00",
          "priceYearly": "50.00",
          "currency": "usd",
          "logo": null,
          "interval": "month",
          "featured": true,
          "allowGrandfatheredAccess": false,
          "discordServers": [],
          "discordRoles": []
        },
        {
          "id": "5e0ba6ac14e2590f760a0f0f",
          "title": "LTT Supporter Plus",
          "description": "You are the real MVP. \n\nYour support helps us continue to build out our team, drive up production values, run experiments that might lose money for a long time (*cough* LTX *cough*) and otherwise be the best content creators we can be.\n\nThis tier includes all the perks of the previous ones, but at floatplane's glorious high bitrate 4K!",
          "price": "10.00",
          "priceYearly": "100.00",
          "currency": "usd",
          "logo": null,
          "interval": "month",
          "featured": false,
          "allowGrandfatheredAccess": false,
          "discordServers": [],
          "discordRoles": []
        }
      ],
      "discoverable": true,
      "subscriberCountDisplay": "total",
      "incomeDisplay": false,
      "card": {
        "width": 375,
        "height": 500,
        "path": "https://pbs.floatplane.com/creator_card/59f94c0bdd241b70349eb72b/281467946609369_1551250329871.jpeg",
        "childImages": [
          {
            "width": 300,
            "height": 400,
            "path": "https://pbs.floatplane.com/creator_card/59f94c0bdd241b70349eb72b/281467946609369_1551250329871_300x400.jpeg"
          }
        ]
      }
    },
    "wasReleasedSilently": true,
    "thumbnail": {
      "width": 1200,
      "height": 675,
      "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412.jpeg",
      "childImages": [
        {
          "width": 400,
          "height": 225,
          "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412_400x225.jpeg"
        }
      ]
    },
    "isAccessible": true,
    "videoAttachments": [
      "TViGzkuIic"
    ],
    "audioAttachments": [],
    "pictureAttachments": [],
    "galleryAttachments": []
  },
  {
    "id": "ge4gLGfXnz",
    "guid": "ge4gLGfXnz",
    "title": "Livestream VOD – October 8, 2021 @ 20:26 – I Have MORE to Say About Steam Deck - WAN Show October 8, 2021",
    "text": "<p>Honey automatically applies the best coupon codes to save you money at </p><p>different online checkouts, try it now at <a href=\"https://www.joinhoney.com/linus\">https://www.joinhoney.com/linus</a></p><p><br /></p><p>Buy a Seasonic Ultra Titanium PSU</p><p>On Amazon: <a href=\"https://geni.us/q4lnefC\">https://geni.us/q4lnefC</a></p><p>On NewEgg: <a href=\"https://lmg.gg/8KV3S\">https://lmg.gg/8KV3S</a></p><p><br /></p><p>Visit <a href=\"https://www.squarespace.com/WAN\">https://www.squarespace.com/WAN</a> and use offer code WAN for 10% off</p><p><br /></p><p>Podcast Download: TBD</p><p><br /></p><p>Check out our other Podcasts:</p><p>Carpool Critics Movie Podcast: <a href=\"https://www.youtube.com/channel/UCt-oJR5teQIjOAxCmIQvcgA\">https://www.youtube.com/channel/UCt-oJR5teQIjOAxCmIQvcgA</a></p><p><br /></p><p>Timestamps TBD</p>",
    "type": "blogPost",
    "attachmentOrder": [
      "psqoN3CgMH",
      "KijsTQP8Rr"
    ],
    "metadata": {
      "hasVideo": true,
      "videoCount": 2,
      "videoDuration": 9506,
      "hasAudio": false,
      "audioCount": 0,
      "audioDuration": 0,
      "hasPicture": false,
      "pictureCount": 0,
      "hasGallery": false,
      "galleryCount": 0,
      "isFeatured": false
    },
    "releaseDate": "2021-10-09T09:28:00.015Z",
    "likes": 43,
    "dislikes": 3,
    "score": 40,
    "comments": 24,
    "creator": {
      "id": "59f94c0bdd241b70349eb72b",
      "owner": {
        "id": "59f94c0bdd241b70349eb723",
        "username": "Linus"
      },
      "title": "LinusTechTips",
      "urlname": "linustechtips",
      "description": "We make entertaining videos about technology, including tech reviews, showcases and other content.",
      "about": "# We're LinusTechTips\nWe make videos and stuff, cool eh?",
      "category": {
        "title": "Technology"
      },
      "cover": {
        "width": 1990,
        "height": 519,
        "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867.jpeg",
        "childImages": [
          {
            "width": 1245,
            "height": 325,
            "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867_1245x325.jpeg"
          }
        ]
      },
      "icon": {
        "width": 600,
        "height": 600,
        "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205.jpeg",
        "childImages": [
          {
            "width": 250,
            "height": 250,
            "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_250x250.jpeg"
          },
          {
            "width": 100,
            "height": 100,
            "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_100x100.jpeg"
          }
        ]
      },
      "liveStream": {
        "id": "5c13f3c006f1be15e08e05c0",
        "title": "First Linux Stream",
        "description": "<p>chat on Twitch</p>",
        "thumbnail": {
          "width": 1200,
          "height": 675,
          "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412.jpeg",
          "childImages": [
            {
              "width": 400,
              "height": 225,
              "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412_400x225.jpeg"
            }
          ]
        },
        "owner": "59f94c0bdd241b70349eb72b",
        "streamPath": "/api/video/v1/us-east-1.758417551536.channel.yKkxur4ukc0B.m3u8",
        "offline": {
          "title": "Offline",
          "description": "We're offline for now – please check back later!",
          "thumbnail": {
            "width": 1920,
            "height": 1080,
            "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026.jpeg",
            "childImages": [
              {
                "width": 400,
                "height": 225,
                "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_400x225.jpeg"
              },
              {
                "width": 1200,
                "height": 675,
                "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_1200x675.jpeg"
              }
            ]
          }
        }
      },
      "subscriptionPlans": [
        {
          "id": "5d48d0306825b5780db93d07",
          "title": "LTT Supporter (1080p)",
          "description": "Includes:\n- Early access (when possible)\n- Live Streaming\n- Behind-the-scenes, cutting room floor & exclusives\n\nNOTE: Tech Quickie and TechLinked are included for now, but will move to their own Floatplane pages in the future",
          "price": "5.00",
          "priceYearly": "50.00",
          "currency": "usd",
          "logo": null,
          "interval": "month",
          "featured": true,
          "allowGrandfatheredAccess": false,
          "discordServers": [],
          "discordRoles": []
        },
        {
          "id": "5e0ba6ac14e2590f760a0f0f",
          "title": "LTT Supporter Plus",
          "description": "You are the real MVP. \n\nYour support helps us continue to build out our team, drive up production values, run experiments that might lose money for a long time (*cough* LTX *cough*) and otherwise be the best content creators we can be.\n\nThis tier includes all the perks of the previous ones, but at floatplane's glorious high bitrate 4K!",
          "price": "10.00",
          "priceYearly": "100.00",
          "currency": "usd",
          "logo": null,
          "interval": "month",
          "featured": false,
          "allowGrandfatheredAccess": false,
          "discordServers": [],
          "discordRoles": []
        }
      ],
      "discoverable": true,
      "subscriberCountDisplay": "total",
      "incomeDisplay": false,
      "card": {
        "width": 375,
        "height": 500,
        "path": "https://pbs.floatplane.com/creator_card/59f94c0bdd241b70349eb72b/281467946609369_1551250329871.jpeg",
        "childImages": [
          {
            "width": 300,
            "height": 400,
            "path": "https://pbs.floatplane.com/creator_card/59f94c0bdd241b70349eb72b/281467946609369_1551250329871_300x400.jpeg"
          }
        ]
      }
    },
    "wasReleasedSilently": false,
    "thumbnail": {
      "width": 640,
      "height": 360,
      "path": "https://pbs.floatplane.com/blogPost_thumbnails/ge4gLGfXnz/564833356017787_1633771544979.jpeg",
      "childImages": [
        {
          "width": 400,
          "height": 225,
          "path": "https://pbs.floatplane.com/blogPost_thumbnails/ge4gLGfXnz/564833356017787_1633771544979_400x225.jpeg"
        }
      ]
    },
    "isAccessible": true,
    "videoAttachments": [
      "KijsTQP8Rr",
      "psqoN3CgMH"
    ],
    "audioAttachments": [],
    "pictureAttachments": [],
    "galleryAttachments": []
  }
]
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getcreatorblogposts-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Creator posted returned|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getcreatorblogposts-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|[[BlogPostModelV3](#schemablogpostmodelv3)]|false|none|none|
|» id|string|true|none|none|
|» guid|string|true|none|none|
|» title|string|true|none|none|
|» text|string|true|none|Text description of the post. May have HTML paragraph (`<p>`) tags surrounding it, along with other HTML..|
|» type|string|true|none|none|
|» channel|any|true|none|none|

*oneOf*

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|»» *anonymous*|[ChannelModel](#schemachannelmodel)|false|none|none|
|»»» id|string|true|none|none|
|»»» creator|string|true|none|none|
|»»» title|string|true|none|none|
|»»» urlname|string|true|none|Shown in the browser URL.|
|»»» about|string|true|none|none|
|»»» order|integer|false|none|none|
|»»» cover|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»»» width|integer|true|none|none|
|»»»» height|integer|true|none|none|
|»»»» path|string(uri)|true|none|none|
|»»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»»»»» width|integer|true|none|none|
|»»»»» height|integer|true|none|none|
|»»»»» path|string(uri)|true|none|none|
|»»» card|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»»» width|integer|true|none|none|
|»»»» height|integer|true|none|none|
|»»»» path|string(uri)|true|none|none|
|»»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»»» icon|[ImageModel](#schemaimagemodel)|true|none|none|
|»»»» width|integer|true|none|none|
|»»»» height|integer|true|none|none|
|»»»» path|string(uri)|true|none|none|
|»»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»»» socialLinks|[SocialLinksModel](#schemasociallinksmodel)|false|none|none|
|»»»» **additionalProperties**|string(uri)|false|none|none|

*xor*

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|»» *anonymous*|string|false|none|Channel identifier|

*continued*

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» tags|[string]|true|none|none|
|» attachmentOrder|[string]|true|none|none|
|» metadata|[PostMetadataModel](#schemapostmetadatamodel)|true|none|none|
|»» hasVideo|boolean|true|none|none|
|»» videoCount|integer|false|none|none|
|»» videoDuration|number|true|none|none|
|»» hasAudio|boolean|true|none|none|
|»» audioCount|integer|false|none|none|
|»» audioDuration|number|true|none|none|
|»» hasPicture|boolean|true|none|none|
|»» pictureCount|integer|false|none|none|
|»» hasGallery|boolean|false|none|none|
|»» galleryCount|integer|false|none|none|
|»» isFeatured|boolean|true|none|none|
|» releaseDate|string(date-time)|true|none|none|
|» likes|integer|true|none|none|
|» dislikes|integer|true|none|none|
|» score|integer|true|none|none|
|» comments|integer|true|none|none|
|» creator|object|true|none|none|
|»» id|string|true|none|none|
|»» owner|object|true|none|none|
|»»» id|string|true|none|none|
|»»» username|string|true|none|none|
|»» title|string|true|none|none|
|»» urlname|string|true|none|Shown in the browser URL, and used in `/creator/named` queries.|
|»» description|string|true|none|none|
|»» about|string|true|none|none|
|»» category|object|true|none|none|
|»»» id|string|true|none|none|
|»»» title|string|true|none|none|
|»» cover|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»» width|integer|true|none|none|
|»»» height|integer|true|none|none|
|»»» path|string(uri)|true|none|none|
|»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»» icon|[ImageModel](#schemaimagemodel)|true|none|none|
|»» liveStream|[LiveStreamModel](#schemalivestreammodel)¦null|true|none|none|
|»»» id|string|true|none|none|
|»»» title|string|true|none|none|
|»»» description|string|true|none|none|
|»»» thumbnail|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»»» width|integer|true|none|none|
|»»»» height|integer|true|none|none|
|»»»» path|string(uri)|true|none|none|
|»»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»»» owner|string|true|none|none|
|»»» channel|string|false|none|The creator channel this livestream belongs to.|
|»»» streamPath|string|true|none|none|
|»»» offline|object|true|none|none|
|»»»» title|string¦null|true|none|none|
|»»»» description|string¦null|true|none|none|
|»»»» thumbnail|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»»»» width|integer|true|none|none|
|»»»»» height|integer|true|none|none|
|»»»»» path|string(uri)|true|none|none|
|»»»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»» subscriptionPlans|[[SubscriptionPlanModel](#schemasubscriptionplanmodel)]|true|none|none|
|»»» id|string|true|none|none|
|»»» title|string|true|none|none|
|»»» description|string|true|none|none|
|»»» price|string¦null|true|none|none|
|»»» priceYearly|string¦null|false|none|none|
|»»» currency|string|true|none|none|
|»»» logo|string¦null|true|none|none|
|»»» interval|string|true|none|none|
|»»» featured|boolean|true|none|none|
|»»» allowGrandfatheredAccess|boolean¦null|false|none|none|
|»»» discordServers|[[DiscordServerModel](#schemadiscordservermodel)]|true|none|none|
|»»»» id|string|true|none|none|
|»»»» guildName|string|true|none|none|
|»»»» guildIcon|string|true|none|none|
|»»»» inviteLink|string(uri)¦null|true|none|none|
|»»»» inviteMode|string|true|none|none|
|»»» discordRoles|[[DiscordRoleModel](#schemadiscordrolemodel)]|true|none|none|
|»»»» server|string|true|none|none|
|»»»» roleName|string|true|none|none|
|»» discoverable|boolean|true|none|none|
|»» subscriberCountDisplay|string|true|none|none|
|»» incomeDisplay|boolean|true|none|none|
|»» defaultChannel|string|false|none|none|
|»» channels|[string]|false|none|none|
|»» card|[ImageModel](#schemaimagemodel)¦null|false|none|none|
|»»» width|integer|true|none|none|
|»»» height|integer|true|none|none|
|»»» path|string(uri)|true|none|none|
|»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|» wasReleasedSilently|boolean|true|none|none|
|» thumbnail|[ImageModel](#schemaimagemodel)¦null|false|none|none|
|»» width|integer|true|none|none|
|»» height|integer|true|none|none|
|»» path|string(uri)|true|none|none|
|»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|» isAccessible|boolean|true|none|If false, the post should be marked as locked and not viewable by the user.|
|» videoAttachments|[string]|false|none|May be undefined, usually when `isAccessible` is `false`.|
|» audioAttachments|[string]|false|none|May be undefined, usually when `isAccessible` is `false`.|
|» pictureAttachments|[string]|false|none|May be undefined, usually when `isAccessible` is `false`.|
|» galleryAttachments|[string]|false|none|May be undefined, usually when `isAccessible` is `false`.|

#### Enumerated Values

|Property|Value|
|---|---|
|type|blogPost|

#### Links

**getBlogPost** => <a href="#opIdgetBlogPost">getBlogPost</a>

|Parameter|Expression|
|---|---|
|id|$response.body#/0/id|

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## getMultiCreatorBlogPosts

<a id="opIdgetMultiCreatorBlogPosts"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v3/content/creator/list?ids=string&limit=1 \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v3/content/creator/list?ids=string&limit=1 HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/content/creator/list?ids=string&limit=1',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v3/content/creator/list',
  params: {
  'ids' => 'array[string]',
'limit' => 'integer'
}, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v3/content/creator/list', params={
  'ids': [
  "string"
],  'limit': '1'
}, headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v3/content/creator/list', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/content/creator/list?ids=string&limit=1");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v3/content/creator/list", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v3/content/creator/list`

*Get Multi Creator Blog Posts*

Retrieve paginated blog posts from multiple creators for the home page.

Example query: https://www.floatplane.com/api/v3/content/creator/list?ids[0]=59f94c0bdd241b70349eb72b&limit=20&fetchAfter[0][creatorId]=59f94c0bdd241b70349eb72b&fetchAfter[0][blogPostId]=B4WsyLnybS&fetchAfter[0][moreFetchable]=true

<h3 id="getmulticreatorblogposts-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|ids|query|array[string]|true|The GUID(s) of the creator(s) to retrieve posts from.|
|limit|query|integer|true|The maximum number of posts to retrieve.|
|fetchAfter|query|array[object]|false|For pagination, this is used to determine which posts to skip. There should be one `fetchAfter` object for each creator in `ids`. The `moreFetchable` in the request, and all of the data, comes from the `ContentCreatorListV3Response`.|

> Example responses

> 200 Response

```json
{
  "blogPosts": [
    {
      "id": "Dw2ms0AgL8",
      "guid": "Dw2ms0AgL8",
      "title": "Livestream VOD – October 9, 2021 @ 07:18 – First Linux Stream",
      "text": "<p>chat on Twitch</p>",
      "type": "blogPost",
      "attachmentOrder": [
        "TViGzkuIic"
      ],
      "metadata": {
        "hasVideo": true,
        "videoCount": 1,
        "videoDuration": 5689,
        "hasAudio": false,
        "audioCount": 0,
        "audioDuration": 0,
        "hasPicture": false,
        "pictureCount": 0,
        "hasGallery": false,
        "galleryCount": 0,
        "isFeatured": false
      },
      "releaseDate": "2021-10-09T09:29:00.039Z",
      "likes": 40,
      "dislikes": 0,
      "score": 40,
      "comments": 28,
      "creator": {
        "id": "59f94c0bdd241b70349eb72b",
        "owner": {
          "id": "59f94c0bdd241b70349eb723",
          "username": "Linus"
        },
        "title": "LinusTechTips",
        "urlname": "linustechtips",
        "description": "We make entertaining videos about technology, including tech reviews, showcases and other content.",
        "about": "# We're LinusTechTips\nWe make videos and stuff, cool eh?",
        "category": {
          "title": "Technology"
        },
        "cover": {
          "width": 1990,
          "height": 519,
          "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867.jpeg",
          "childImages": [
            {
              "width": 1245,
              "height": 325,
              "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867_1245x325.jpeg"
            }
          ]
        },
        "icon": {
          "width": 600,
          "height": 600,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205.jpeg",
          "childImages": [
            {
              "width": 250,
              "height": 250,
              "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_250x250.jpeg"
            },
            {
              "width": 100,
              "height": 100,
              "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_100x100.jpeg"
            }
          ]
        },
        "liveStream": {
          "id": "5c13f3c006f1be15e08e05c0",
          "title": "First Linux Stream",
          "description": "<p>chat on Twitch</p>",
          "thumbnail": {
            "width": 1200,
            "height": 675,
            "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412.jpeg",
            "childImages": [
              {
                "width": 400,
                "height": 225,
                "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412_400x225.jpeg"
              }
            ]
          },
          "owner": "59f94c0bdd241b70349eb72b",
          "streamPath": "/api/video/v1/us-east-1.758417551536.channel.yKkxur4ukc0B.m3u8",
          "offline": {
            "title": "Offline",
            "description": "We're offline for now – please check back later!",
            "thumbnail": {
              "width": 1920,
              "height": 1080,
              "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026.jpeg",
              "childImages": [
                {
                  "width": 400,
                  "height": 225,
                  "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_400x225.jpeg"
                },
                {
                  "width": 1200,
                  "height": 675,
                  "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_1200x675.jpeg"
                }
              ]
            }
          }
        },
        "subscriptionPlans": [
          {
            "id": "5d48d0306825b5780db93d07",
            "title": "LTT Supporter (1080p)",
            "description": "Includes:\n- Early access (when possible)\n- Live Streaming\n- Behind-the-scenes, cutting room floor & exclusives\n\nNOTE: Tech Quickie and TechLinked are included for now, but will move to their own Floatplane pages in the future",
            "price": "5.00",
            "priceYearly": "50.00",
            "currency": "usd",
            "logo": null,
            "interval": "month",
            "featured": true,
            "allowGrandfatheredAccess": false,
            "discordServers": [],
            "discordRoles": []
          },
          {
            "id": "5e0ba6ac14e2590f760a0f0f",
            "title": "LTT Supporter Plus",
            "description": "You are the real MVP. \n\nYour support helps us continue to build out our team, drive up production values, run experiments that might lose money for a long time (*cough* LTX *cough*) and otherwise be the best content creators we can be.\n\nThis tier includes all the perks of the previous ones, but at floatplane's glorious high bitrate 4K!",
            "price": "10.00",
            "priceYearly": "100.00",
            "currency": "usd",
            "logo": null,
            "interval": "month",
            "featured": false,
            "allowGrandfatheredAccess": false,
            "discordServers": [],
            "discordRoles": []
          }
        ],
        "discoverable": true,
        "subscriberCountDisplay": "total",
        "incomeDisplay": false,
        "card": {
          "width": 375,
          "height": 500,
          "path": "https://pbs.floatplane.com/creator_card/59f94c0bdd241b70349eb72b/281467946609369_1551250329871.jpeg",
          "childImages": [
            {
              "width": 300,
              "height": 400,
              "path": "https://pbs.floatplane.com/creator_card/59f94c0bdd241b70349eb72b/281467946609369_1551250329871_300x400.jpeg"
            }
          ]
        }
      },
      "wasReleasedSilently": true,
      "thumbnail": {
        "width": 1200,
        "height": 675,
        "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412.jpeg",
        "childImages": [
          {
            "width": 400,
            "height": 225,
            "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412_400x225.jpeg"
          }
        ]
      },
      "isAccessible": true,
      "videoAttachments": [
        "TViGzkuIic"
      ],
      "audioAttachments": [],
      "pictureAttachments": [],
      "galleryAttachments": []
    },
    {
      "id": "ge4gLGfXnz",
      "guid": "ge4gLGfXnz",
      "title": "Livestream VOD – October 8, 2021 @ 20:26 – I Have MORE to Say About Steam Deck - WAN Show October 8, 2021",
      "text": "<p>Honey automatically applies the best coupon codes to save you money at </p><p>different online checkouts, try it now at <a href=\"https://www.joinhoney.com/linus\">https://www.joinhoney.com/linus</a></p><p><br /></p><p>Buy a Seasonic Ultra Titanium PSU</p><p>On Amazon: <a href=\"https://geni.us/q4lnefC\">https://geni.us/q4lnefC</a></p><p>On NewEgg: <a href=\"https://lmg.gg/8KV3S\">https://lmg.gg/8KV3S</a></p><p><br /></p><p>Visit <a href=\"https://www.squarespace.com/WAN\">https://www.squarespace.com/WAN</a> and use offer code WAN for 10% off</p><p><br /></p><p>Podcast Download: TBD</p><p><br /></p><p>Check out our other Podcasts:</p><p>Carpool Critics Movie Podcast: <a href=\"https://www.youtube.com/channel/UCt-oJR5teQIjOAxCmIQvcgA\">https://www.youtube.com/channel/UCt-oJR5teQIjOAxCmIQvcgA</a></p><p><br /></p><p>Timestamps TBD</p>",
      "type": "blogPost",
      "attachmentOrder": [
        "psqoN3CgMH",
        "KijsTQP8Rr"
      ],
      "metadata": {
        "hasVideo": true,
        "videoCount": 2,
        "videoDuration": 9506,
        "hasAudio": false,
        "audioCount": 0,
        "audioDuration": 0,
        "hasPicture": false,
        "pictureCount": 0,
        "hasGallery": false,
        "galleryCount": 0,
        "isFeatured": false
      },
      "releaseDate": "2021-10-09T09:28:00.015Z",
      "likes": 43,
      "dislikes": 3,
      "score": 40,
      "comments": 24,
      "creator": {
        "id": "59f94c0bdd241b70349eb72b",
        "owner": {
          "id": "59f94c0bdd241b70349eb723",
          "username": "Linus"
        },
        "title": "LinusTechTips",
        "urlname": "linustechtips",
        "description": "We make entertaining videos about technology, including tech reviews, showcases and other content.",
        "about": "# We're LinusTechTips\nWe make videos and stuff, cool eh?",
        "category": {
          "title": "Technology"
        },
        "cover": {
          "width": 1990,
          "height": 519,
          "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867.jpeg",
          "childImages": [
            {
              "width": 1245,
              "height": 325,
              "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867_1245x325.jpeg"
            }
          ]
        },
        "icon": {
          "width": 600,
          "height": 600,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205.jpeg",
          "childImages": [
            {
              "width": 250,
              "height": 250,
              "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_250x250.jpeg"
            },
            {
              "width": 100,
              "height": 100,
              "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_100x100.jpeg"
            }
          ]
        },
        "liveStream": {
          "id": "5c13f3c006f1be15e08e05c0",
          "title": "First Linux Stream",
          "description": "<p>chat on Twitch</p>",
          "thumbnail": {
            "width": 1200,
            "height": 675,
            "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412.jpeg",
            "childImages": [
              {
                "width": 400,
                "height": 225,
                "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412_400x225.jpeg"
              }
            ]
          },
          "owner": "59f94c0bdd241b70349eb72b",
          "streamPath": "/api/video/v1/us-east-1.758417551536.channel.yKkxur4ukc0B.m3u8",
          "offline": {
            "title": "Offline",
            "description": "We're offline for now – please check back later!",
            "thumbnail": {
              "width": 1920,
              "height": 1080,
              "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026.jpeg",
              "childImages": [
                {
                  "width": 400,
                  "height": 225,
                  "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_400x225.jpeg"
                },
                {
                  "width": 1200,
                  "height": 675,
                  "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_1200x675.jpeg"
                }
              ]
            }
          }
        },
        "subscriptionPlans": [
          {
            "id": "5d48d0306825b5780db93d07",
            "title": "LTT Supporter (1080p)",
            "description": "Includes:\n- Early access (when possible)\n- Live Streaming\n- Behind-the-scenes, cutting room floor & exclusives\n\nNOTE: Tech Quickie and TechLinked are included for now, but will move to their own Floatplane pages in the future",
            "price": "5.00",
            "priceYearly": "50.00",
            "currency": "usd",
            "logo": null,
            "interval": "month",
            "featured": true,
            "allowGrandfatheredAccess": false,
            "discordServers": [],
            "discordRoles": []
          },
          {
            "id": "5e0ba6ac14e2590f760a0f0f",
            "title": "LTT Supporter Plus",
            "description": "You are the real MVP. \n\nYour support helps us continue to build out our team, drive up production values, run experiments that might lose money for a long time (*cough* LTX *cough*) and otherwise be the best content creators we can be.\n\nThis tier includes all the perks of the previous ones, but at floatplane's glorious high bitrate 4K!",
            "price": "10.00",
            "priceYearly": "100.00",
            "currency": "usd",
            "logo": null,
            "interval": "month",
            "featured": false,
            "allowGrandfatheredAccess": false,
            "discordServers": [],
            "discordRoles": []
          }
        ],
        "discoverable": true,
        "subscriberCountDisplay": "total",
        "incomeDisplay": false,
        "card": {
          "width": 375,
          "height": 500,
          "path": "https://pbs.floatplane.com/creator_card/59f94c0bdd241b70349eb72b/281467946609369_1551250329871.jpeg",
          "childImages": [
            {
              "width": 300,
              "height": 400,
              "path": "https://pbs.floatplane.com/creator_card/59f94c0bdd241b70349eb72b/281467946609369_1551250329871_300x400.jpeg"
            }
          ]
        }
      },
      "wasReleasedSilently": false,
      "thumbnail": {
        "width": 640,
        "height": 360,
        "path": "https://pbs.floatplane.com/blogPost_thumbnails/ge4gLGfXnz/564833356017787_1633771544979.jpeg",
        "childImages": [
          {
            "width": 400,
            "height": 225,
            "path": "https://pbs.floatplane.com/blogPost_thumbnails/ge4gLGfXnz/564833356017787_1633771544979_400x225.jpeg"
          }
        ]
      },
      "isAccessible": true,
      "videoAttachments": [
        "KijsTQP8Rr",
        "psqoN3CgMH"
      ],
      "audioAttachments": [],
      "pictureAttachments": [],
      "galleryAttachments": []
    }
  ],
  "lastElements": [
    {
      "creatorId": "59f94c0bdd241b70349eb72b",
      "blogPostId": "l2wH2gXLiW",
      "moreFetchable": true
    }
  ]
}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getmulticreatorblogposts-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Posts returned|[ContentCreatorListV3Response](#schemacontentcreatorlistv3response)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getmulticreatorblogposts-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## getContentTags

<a id="opIdgetContentTags"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v3/content/tags?creatorIds=string \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v3/content/tags?creatorIds=string HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/content/tags?creatorIds=string',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v3/content/tags',
  params: {
  'creatorIds' => 'array[string]'
}, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v3/content/tags', params={
  'creatorIds': [
  "string"
]
}, headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v3/content/tags', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/content/tags?creatorIds=string");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v3/content/tags", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v3/content/tags`

*Get Content Tags*

Retrieve all tags and the number of times the tags have been used for the specified creator(s).

<h3 id="getcontenttags-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|creatorIds|query|array[string]|true|The creator(s) to search by.|

> Example responses

> 200 Response

```json
{
  "battery": 1,
  "server": 1,
  "Airpods": 1,
  "storage": 1,
  "tjm": 1,
  "Apple": 1,
  "swap": 1,
  "memory": 1,
  "ltt": 1
}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getcontenttags-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Creator tag information|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getcontenttags-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» **additionalProperties**|integer|false|none|none|

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## getBlogPost

<a id="opIdgetBlogPost"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v3/content/post?id=string \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v3/content/post?id=string HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/content/post?id=string',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v3/content/post',
  params: {
  'id' => 'string'
}, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v3/content/post', params={
  'id': 'string'
}, headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v3/content/post', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/content/post?id=string");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v3/content/post", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v3/content/post`

*Get Blog Post*

Retrieve more details on a specific blog post object for viewing.

<h3 id="getblogpost-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id|query|string|true|The ID of the post to be retrieved.|

> Example responses

> 200 Response

```json
{
  "id": "Dw2ms0AgL8",
  "guid": "Dw2ms0AgL8",
  "title": "Livestream VOD – October 9, 2021 @ 07:18 – First Linux Stream",
  "text": "<p>chat on Twitch</p>",
  "type": "blogPost",
  "tags": [
    "test"
  ],
  "attachmentOrder": [
    "TViGzkuIic"
  ],
  "metadata": {
    "hasVideo": true,
    "videoCount": 1,
    "videoDuration": 5689,
    "hasAudio": false,
    "audioCount": 0,
    "audioDuration": 0,
    "hasPicture": false,
    "pictureCount": 0,
    "hasGallery": false,
    "galleryCount": 0,
    "isFeatured": false
  },
  "releaseDate": "2021-10-09T09:29:00.039Z",
  "likes": 41,
  "dislikes": 0,
  "score": 41,
  "comments": 28,
  "creator": {
    "id": "59f94c0bdd241b70349eb72b",
    "owner": "59f94c0bdd241b70349eb723",
    "title": "LinusTechTips",
    "urlname": "linustechtips",
    "description": "We make entertaining videos about technology, including tech reviews, showcases and other content.",
    "about": "# We're LinusTechTips\nWe make videos and stuff, cool eh?",
    "category": "59f94c0bdd241b70349eb727",
    "cover": null,
    "icon": {
      "width": 600,
      "height": 600,
      "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205.jpeg",
      "childImages": [
        {
          "width": 250,
          "height": 250,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_250x250.jpeg"
        },
        {
          "width": 100,
          "height": 100,
          "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_100x100.jpeg"
        }
      ]
    },
    "liveStream": null,
    "subscriptionPlans": null,
    "discoverable": true,
    "subscriberCountDisplay": "total",
    "incomeDisplay": false
  },
  "wasReleasedSilently": true,
  "thumbnail": {
    "width": 1200,
    "height": 675,
    "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412.jpeg",
    "childImages": [
      {
        "width": 400,
        "height": 225,
        "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412_400x225.jpeg"
      }
    ]
  },
  "isAccessible": true,
  "userInteraction": [],
  "videoAttachments": [
    {
      "id": "TViGzkuIic",
      "guid": "TViGzkuIic",
      "title": "October 9, 2021 @ 07:18 – First Linux Stream",
      "type": "video",
      "description": "",
      "releaseDate": null,
      "duration": 5689,
      "creator": "59f94c0bdd241b70349eb72b",
      "likes": 0,
      "dislikes": 0,
      "score": 0,
      "isProcessing": false,
      "primaryBlogPost": "Dw2ms0AgL8",
      "thumbnail": {
        "width": 1920,
        "height": 1080,
        "path": "https://pbs.floatplane.com/content_thumbnails/TViGzkuIic/324783659287024_1633769709593.jpeg",
        "childImages": []
      },
      "isAccessible": true
    }
  ],
  "audioAttachments": [
    {
      "id": "iGssjNGPSD",
      "guid": "iGssjNGPSD",
      "title": "Robocop FP.mp3",
      "type": "audio",
      "description": "",
      "duration": 4165,
      "waveform": {
        "dataSetLength": 3,
        "highestValue": 71,
        "lowestValue": 50,
        "data": [
          71,
          50,
          69
        ]
      },
      "creator": "59f94c0bdd241b70349eb72b",
      "likes": 0,
      "dislikes": 0,
      "score": 0,
      "isProcessing": false,
      "primaryBlogPost": "jVU2y9PlnG",
      "isAccessible": true
    }
  ],
  "pictureAttachments": [],
  "galleryAttachments": []
}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getblogpost-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Detailed post information|[ContentPostV3Response](#schemacontentpostv3response)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getblogpost-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## getRelatedBlogPosts

<a id="opIdgetRelatedBlogPosts"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v3/content/related?id=string \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v3/content/related?id=string HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/content/related?id=string',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v3/content/related',
  params: {
  'id' => 'string'
}, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v3/content/related', params={
  'id': 'string'
}, headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v3/content/related', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/content/related?id=string");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v3/content/related", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v3/content/related`

*Get Related Blog Posts*

Retrieve a list of blog posts that are related to the post being viewed.

<h3 id="getrelatedblogposts-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id|query|string|true|The ID of the originating post.|

> Example responses

> 200 Response

```json
[
  {
    "id": "ge4gLGfXnz",
    "guid": "ge4gLGfXnz",
    "title": "Livestream VOD – October 8, 2021 @ 20:26 – I Have MORE to Say About Steam Deck - WAN Show October 8, 2021",
    "text": "<p>Honey automatically applies the best coupon codes to save you money at </p><p>different online checkouts, try it now at <a href=\"https://www.joinhoney.com/linus\">https://www.joinhoney.com/linus</a></p><p><br /></p><p>Buy a Seasonic Ultra Titanium PSU</p><p>On Amazon: <a href=\"https://geni.us/q4lnefC\">https://geni.us/q4lnefC</a></p><p>On NewEgg: <a href=\"https://lmg.gg/8KV3S\">https://lmg.gg/8KV3S</a></p><p><br /></p><p>Visit <a href=\"https://www.squarespace.com/WAN\">https://www.squarespace.com/WAN</a> and use offer code WAN for 10% off</p><p><br /></p><p>Podcast Download: TBD</p><p><br /></p><p>Check out our other Podcasts:</p><p>Carpool Critics Movie Podcast: <a href=\"https://www.youtube.com/channel/UCt-oJR5teQIjOAxCmIQvcgA\">https://www.youtube.com/channel/UCt-oJR5teQIjOAxCmIQvcgA</a></p><p><br /></p><p>Timestamps TBD</p>",
    "type": "blogPost",
    "tags": [
      "test"
    ],
    "attachmentOrder": [
      "psqoN3CgMH",
      "KijsTQP8Rr"
    ],
    "metadata": {
      "hasVideo": true,
      "videoCount": 2,
      "videoDuration": 9506,
      "hasAudio": false,
      "audioCount": 0,
      "audioDuration": 0,
      "hasPicture": false,
      "pictureCount": 0,
      "hasGallery": false,
      "galleryCount": 0,
      "isFeatured": false
    },
    "releaseDate": "2021-10-09T09:28:00.015Z",
    "likes": 43,
    "dislikes": 3,
    "score": 40,
    "comments": 24,
    "creator": {
      "id": "59f94c0bdd241b70349eb72b",
      "owner": {
        "id": "59f94c0bdd241b70349eb723",
        "username": "Linus"
      },
      "title": "LinusTechTips",
      "urlname": "linustechtips",
      "description": "We make entertaining videos about technology, including tech reviews, showcases and other content.",
      "about": "# We're LinusTechTips\nWe make videos and stuff, cool eh?",
      "category": {
        "title": "Technology"
      },
      "cover": {
        "width": 1990,
        "height": 519,
        "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867.jpeg",
        "childImages": [
          {
            "width": 1245,
            "height": 325,
            "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867_1245x325.jpeg"
          }
        ]
      },
      "icon": {
        "width": 600,
        "height": 600,
        "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205.jpeg",
        "childImages": [
          {
            "width": 250,
            "height": 250,
            "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_250x250.jpeg"
          },
          {
            "width": 100,
            "height": 100,
            "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_100x100.jpeg"
          }
        ]
      },
      "liveStream": {
        "id": "5c13f3c006f1be15e08e05c0",
        "title": "First Linux Stream",
        "description": "<p>chat on Twitch</p>",
        "thumbnail": {
          "width": 1200,
          "height": 675,
          "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412.jpeg",
          "childImages": [
            {
              "width": 400,
              "height": 225,
              "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412_400x225.jpeg"
            }
          ]
        },
        "owner": "59f94c0bdd241b70349eb72b",
        "streamPath": "/api/video/v1/us-east-1.758417551536.channel.yKkxur4ukc0B.m3u8",
        "offline": {
          "title": "Offline",
          "description": "We're offline for now – please check back later!",
          "thumbnail": {
            "width": 1920,
            "height": 1080,
            "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026.jpeg",
            "childImages": [
              {
                "width": 400,
                "height": 225,
                "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_400x225.jpeg"
              },
              {
                "width": 1200,
                "height": 675,
                "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_1200x675.jpeg"
              }
            ]
          }
        }
      },
      "subscriptionPlans": [
        {
          "id": "5d48d0306825b5780db93d07",
          "title": "LTT Supporter (1080p)",
          "description": "Includes:\n- Early access (when possible)\n- Live Streaming\n- Behind-the-scenes, cutting room floor & exclusives\n\nNOTE: Tech Quickie and TechLinked are included for now, but will move to their own Floatplane pages in the future",
          "price": "5.00",
          "priceYearly": "50.00",
          "currency": "usd",
          "logo": null,
          "interval": "month",
          "featured": true,
          "allowGrandfatheredAccess": false,
          "discordServers": [],
          "discordRoles": []
        },
        {
          "id": "5e0ba6ac14e2590f760a0f0f",
          "title": "LTT Supporter Plus",
          "description": "You are the real MVP. \n\nYour support helps us continue to build out our team, drive up production values, run experiments that might lose money for a long time (*cough* LTX *cough*) and otherwise be the best content creators we can be.\n\nThis tier includes all the perks of the previous ones, but at floatplane's glorious high bitrate 4K!",
          "price": "10.00",
          "priceYearly": "100.00",
          "currency": "usd",
          "logo": null,
          "interval": "month",
          "featured": false,
          "allowGrandfatheredAccess": false,
          "discordServers": [],
          "discordRoles": []
        }
      ],
      "discoverable": true,
      "subscriberCountDisplay": "total",
      "incomeDisplay": false,
      "card": {
        "width": 375,
        "height": 500,
        "path": "https://pbs.floatplane.com/creator_card/59f94c0bdd241b70349eb72b/281467946609369_1551250329871.jpeg",
        "childImages": [
          {
            "width": 300,
            "height": 400,
            "path": "https://pbs.floatplane.com/creator_card/59f94c0bdd241b70349eb72b/281467946609369_1551250329871_300x400.jpeg"
          }
        ]
      }
    },
    "wasReleasedSilently": false,
    "thumbnail": {
      "width": 640,
      "height": 360,
      "path": "https://pbs.floatplane.com/blogPost_thumbnails/ge4gLGfXnz/564833356017787_1633771544979.jpeg",
      "childImages": [
        {
          "width": 400,
          "height": 225,
          "path": "https://pbs.floatplane.com/blogPost_thumbnails/ge4gLGfXnz/564833356017787_1633771544979_400x225.jpeg"
        }
      ]
    },
    "isAccessible": true
  },
  {
    "id": "j7KjCaKrtV",
    "guid": "j7KjCaKrtV",
    "title": "TL: Facebook Does Not Care.",
    "text": "<p><strong>NEWS SOURCES:</strong></p><p><strong> </strong></p><p>MICRO-BOSS, WORD</p><p>Microsoft to make its devices more repairable following pressure</p><p><a href=\"https://linustechtips.com/topic/1379452-microsoft-agrees-to-independent-third-party-study-to-look-into-right-to-repair/\">https://linustechtips.com/topic/1379452-microsoft-agrees-to-independent-third-party-study-to-look-into-right-to-repair/</a></p><p><a href=\"https://grist.org/accountability/bowing-to-investors-microsoft-will-make-its-devices-easier-to-fix/\">https://grist.org/accountability/bowing-to-investors-microsoft-will-make-its-devices-easier-to-fix/</a></p><p><a href=\"https://www.asyousow.org/about-us#:~:text=As%20You%20Sow%20is%20the%20nation%E2%80%99s%20non-profit%20leader%20in%20shareholder%20advocacy\">https://www.asyousow.org/about-us#:~:text=As%20You%20Sow%20is%20the%20nation%E2%80%99s%20non-profit%20leader%20in%20shareholder%20advocacy</a>.</p><p>Louis is happy <a href=\"https://www.youtube.com/watch?v=TiMdvR99fBQ\">https://www.youtube.com/watch?v=TiMdvR99fBQ</a></p><p> </p><p>WHAT, THIS OLE’D THING?</p><p>Switch OLED launches <a href=\"https://arstechnica.com/gaming/2021/10/switch-oled-review-nintendos-nicest-most-nonessential-upgrade-yet/\">https://arstechnica.com/gaming/2021/10/switch-oled-review-nintendos-nicest-most-nonessential-upgrade-yet/</a></p><p><a href=\"https://www.youtube.com/watch?v=4mHq6Y7JSmg\">https://www.youtube.com/watch?v=4mHq6Y7JSmg</a></p><p>New Joy-Cons, but drift isn’t going away</p><p><a href=\"https://www.polygon.com/22688586/nintendo-switch-oled-joy-con-drift-controllers\">https://www.polygon.com/22688586/nintendo-switch-oled-joy-con-drift-controllers</a></p><p>screen protector <a href=\"https://www.nintendolife.com/news/2021/10/switch-oled-comes-with-a-screen-protector-installed-but-please-dont-remove-it-says-nintendo\">https://www.nintendolife.com/news/2021/10/switch-oled-comes-with-a-screen-protector-installed-but-please-dont-remove-it-says-nintendo</a></p><p> </p><p>SHAMEBOOK</p><p>Facebook has more outages</p><p><a href=\"https://www.engadget.com/facebook-and-instagram-are-down-for-the-second-time-this-week-193257623.html\">https://www.engadget.com/facebook-and-instagram-are-down-for-the-second-time-this-week-193257623.html</a></p><p><a href=\"https://twitter.com/Facebook/status/1446556732977778695\">https://twitter.com/Facebook/status/1446556732977778695</a></p><p>IG fixed <a href=\"https://twitter.com/InstagramComms/status/1446582114468597761\">https://twitter.com/InstagramComms/status/1446582114468597761</a></p><p>Unfollow everything developer banned <a href=\"https://www.theverge.com/2021/10/8/22716044/facebook-unfollow-everything-tool-louis-barclay-banned-for-life\">https://www.theverge.com/2021/10/8/22716044/facebook-unfollow-everything-tool-louis-barclay-banned-for-life</a></p><p><a href=\"https://slate.com/technology/2021/10/facebook-unfollow-everything-cease-desist.html\">https://slate.com/technology/2021/10/facebook-unfollow-everything-cease-desist.html</a></p><p><a href=\"https://louisbarclay.notion.site/Unfollow-Everything-cease-and-desist-letter-from-Facebook-ea219169421b457bb7ce010b7bf9ce1f\">https://louisbarclay.notion.site/Unfollow-Everything-cease-and-desist-letter-from-Facebook-ea219169421b457bb7ce010b7bf9ce1f</a></p><p> </p><p>QUICK BITS</p><p> </p><p>GET JEFF’D</p><p>Game backgrounds on Twitch replaced with Jeff Bezos</p><p><a href=\"https://twitter.com/AnEternalEnigma/status/1446421951883489281\">https://twitter.com/AnEternalEnigma/status/1446421951883489281</a></p><p>defaced <a href=\"https://www.cnet.com/tech/gaming/twitch-reportedly-defaced-with-pictures-of-jeff-bezos/\">https://www.cnet.com/tech/gaming/twitch-reportedly-defaced-with-pictures-of-jeff-bezos/</a></p><p> </p><p>ABOUT TO GET PADDLED</p><p>First In-App Purchasing alternative for iOS: Paddle</p><p><a href=\"https://twitter.com/PaddleHQ/status/1446050078301605890\">https://twitter.com/PaddleHQ/status/1446050078301605890</a></p><p><a href=\"https://paddle.com/platform/in-app-purchase/\">https://paddle.com/platform/in-app-purchase/</a></p><p>Sweeney</p><p><a href=\"https://twitter.com/TimSweeneyEpic/status/1446117510919585796\">https://twitter.com/TimSweeneyEpic/status/1446117510919585796</a></p><p> </p><p>STAY KINECTED</p><p>Sky Glass TV looks like a big iMac</p><p><a href=\"https://www.youtube.com/watch?v=GpGskL5PCKU\">https://www.youtube.com/watch?v=GpGskL5PCKU</a></p><p>Kinect motion controls <a href=\"https://www.theverge.com/2021/10/7/22714117/microsoft-kinect-is-back-sky-glass-tv-smart-camera-features\">https://www.theverge.com/2021/10/7/22714117/microsoft-kinect-is-back-sky-glass-tv-smart-camera-features</a> - full screen embedded video on this page and record</p><p><br /></p><p>UPGRADED TO CUMULONIMBUS</p><p>XCloud is now running Xbox Series X hardware</p><p><a href=\"https://www.kitguru.net/gaming/matthew-wilson/microsoft-has-upgraded-xcloud-to-xbox-series-x-hardware/\">https://www.kitguru.net/gaming/matthew-wilson/microsoft-has-upgraded-xcloud-to-xbox-series-x-hardware/</a></p><p><a href=\"https://www.eurogamer.net/articles/2021-10-07-xbox-cloud-gaming-now-runs-on-series-x-hardware\">https://www.eurogamer.net/articles/2021-10-07-xbox-cloud-gaming-now-runs-on-series-x-hardware</a></p><p> </p><p>NO PIXEL LEFT UN-LEAKED</p><p>Pixel 6 and 6 Pro teardowns leaked</p><p><a href=\"https://9to5google.com/2021/10/08/pixel-6-pro-teardown-leak/\">https://9to5google.com/2021/10/08/pixel-6-pro-teardown-leak/</a></p>",
    "type": "blogPost",
    "attachmentOrder": [
      "R3mVASdVGt"
    ],
    "metadata": {
      "hasVideo": true,
      "videoCount": 1,
      "videoDuration": 377,
      "hasAudio": false,
      "audioCount": 0,
      "audioDuration": 0,
      "hasPicture": false,
      "pictureCount": 0,
      "hasGallery": false,
      "galleryCount": 0,
      "isFeatured": false
    },
    "releaseDate": "2021-10-09T02:55:00.045Z",
    "likes": 101,
    "dislikes": 0,
    "score": 101,
    "comments": 19,
    "creator": {
      "id": "59f94c0bdd241b70349eb72b",
      "owner": {
        "id": "59f94c0bdd241b70349eb723",
        "username": "Linus"
      },
      "title": "LinusTechTips",
      "urlname": "linustechtips",
      "description": "We make entertaining videos about technology, including tech reviews, showcases and other content.",
      "about": "# We're LinusTechTips\nWe make videos and stuff, cool eh?",
      "category": {
        "title": "Technology"
      },
      "cover": {
        "width": 1990,
        "height": 519,
        "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867.jpeg",
        "childImages": [
          {
            "width": 1245,
            "height": 325,
            "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867_1245x325.jpeg"
          }
        ]
      },
      "icon": {
        "width": 600,
        "height": 600,
        "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205.jpeg",
        "childImages": [
          {
            "width": 250,
            "height": 250,
            "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_250x250.jpeg"
          },
          {
            "width": 100,
            "height": 100,
            "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_100x100.jpeg"
          }
        ]
      },
      "liveStream": {
        "id": "5c13f3c006f1be15e08e05c0",
        "title": "First Linux Stream",
        "description": "<p>chat on Twitch</p>",
        "thumbnail": {
          "width": 1200,
          "height": 675,
          "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412.jpeg",
          "childImages": [
            {
              "width": 400,
              "height": 225,
              "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412_400x225.jpeg"
            }
          ]
        },
        "owner": "59f94c0bdd241b70349eb72b",
        "streamPath": "/api/video/v1/us-east-1.758417551536.channel.yKkxur4ukc0B.m3u8",
        "offline": {
          "title": "Offline",
          "description": "We're offline for now – please check back later!",
          "thumbnail": {
            "width": 1920,
            "height": 1080,
            "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026.jpeg",
            "childImages": [
              {
                "width": 400,
                "height": 225,
                "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_400x225.jpeg"
              },
              {
                "width": 1200,
                "height": 675,
                "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_1200x675.jpeg"
              }
            ]
          }
        }
      },
      "subscriptionPlans": [
        {
          "id": "5d48d0306825b5780db93d07",
          "title": "LTT Supporter (1080p)",
          "description": "Includes:\n- Early access (when possible)\n- Live Streaming\n- Behind-the-scenes, cutting room floor & exclusives\n\nNOTE: Tech Quickie and TechLinked are included for now, but will move to their own Floatplane pages in the future",
          "price": "5.00",
          "priceYearly": "50.00",
          "currency": "usd",
          "logo": null,
          "interval": "month",
          "featured": true,
          "allowGrandfatheredAccess": false,
          "discordServers": [],
          "discordRoles": []
        },
        {
          "id": "5e0ba6ac14e2590f760a0f0f",
          "title": "LTT Supporter Plus",
          "description": "You are the real MVP. \n\nYour support helps us continue to build out our team, drive up production values, run experiments that might lose money for a long time (*cough* LTX *cough*) and otherwise be the best content creators we can be.\n\nThis tier includes all the perks of the previous ones, but at floatplane's glorious high bitrate 4K!",
          "price": "10.00",
          "priceYearly": "100.00",
          "currency": "usd",
          "logo": null,
          "interval": "month",
          "featured": false,
          "allowGrandfatheredAccess": false,
          "discordServers": [],
          "discordRoles": []
        }
      ],
      "discoverable": true,
      "subscriberCountDisplay": "total",
      "incomeDisplay": false,
      "card": {
        "width": 375,
        "height": 500,
        "path": "https://pbs.floatplane.com/creator_card/59f94c0bdd241b70349eb72b/281467946609369_1551250329871.jpeg",
        "childImages": [
          {
            "width": 300,
            "height": 400,
            "path": "https://pbs.floatplane.com/creator_card/59f94c0bdd241b70349eb72b/281467946609369_1551250329871_300x400.jpeg"
          }
        ]
      }
    },
    "wasReleasedSilently": false,
    "thumbnail": {
      "width": 1920,
      "height": 1080,
      "path": "https://pbs.floatplane.com/blogPost_thumbnails/j7KjCaKrtV/726584368653303_1633741254596.jpeg",
      "childImages": [
        {
          "width": 1200,
          "height": 675,
          "path": "https://pbs.floatplane.com/blogPost_thumbnails/j7KjCaKrtV/726584368653303_1633741254596_1200x675.jpeg"
        },
        {
          "width": 400,
          "height": 225,
          "path": "https://pbs.floatplane.com/blogPost_thumbnails/j7KjCaKrtV/726584368653303_1633741254596_400x225.jpeg"
        }
      ]
    },
    "isAccessible": true
  },
  {
    "id": "VkxlNv3k8j",
    "guid": "VkxlNv3k8j",
    "title": "TQ: Has USB-C WON Against Apple?",
    "text": "<p>Learn about the proposed USB-C mandate in the EU.</p>",
    "type": "blogPost",
    "attachmentOrder": [
      "7OWQQsxYYN"
    ],
    "metadata": {
      "hasVideo": true,
      "videoCount": 1,
      "videoDuration": 293,
      "hasAudio": false,
      "audioCount": 0,
      "audioDuration": 0,
      "hasPicture": false,
      "pictureCount": 0,
      "hasGallery": false,
      "galleryCount": 0,
      "isFeatured": false
    },
    "releaseDate": "2021-10-08T23:31:00.037Z",
    "likes": 106,
    "dislikes": 0,
    "score": 106,
    "comments": 15,
    "creator": {
      "id": "59f94c0bdd241b70349eb72b",
      "owner": {
        "id": "59f94c0bdd241b70349eb723",
        "username": "Linus"
      },
      "title": "LinusTechTips",
      "urlname": "linustechtips",
      "description": "We make entertaining videos about technology, including tech reviews, showcases and other content.",
      "about": "# We're LinusTechTips\nWe make videos and stuff, cool eh?",
      "category": {
        "title": "Technology"
      },
      "cover": {
        "width": 1990,
        "height": 519,
        "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867.jpeg",
        "childImages": [
          {
            "width": 1245,
            "height": 325,
            "path": "https://pbs.floatplane.com/cover_images/59f94c0bdd241b70349eb72b/696951209272749_1521668313867_1245x325.jpeg"
          }
        ]
      },
      "icon": {
        "width": 600,
        "height": 600,
        "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205.jpeg",
        "childImages": [
          {
            "width": 250,
            "height": 250,
            "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_250x250.jpeg"
          },
          {
            "width": 100,
            "height": 100,
            "path": "https://pbs.floatplane.com/creator_icons/59f94c0bdd241b70349eb72b/770551996990709_1551249357205_100x100.jpeg"
          }
        ]
      },
      "liveStream": {
        "id": "5c13f3c006f1be15e08e05c0",
        "title": "First Linux Stream",
        "description": "<p>chat on Twitch</p>",
        "thumbnail": {
          "width": 1200,
          "height": 675,
          "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412.jpeg",
          "childImages": [
            {
              "width": 400,
              "height": 225,
              "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/084281881402899_1633761128412_400x225.jpeg"
            }
          ]
        },
        "owner": "59f94c0bdd241b70349eb72b",
        "streamPath": "/api/video/v1/us-east-1.758417551536.channel.yKkxur4ukc0B.m3u8",
        "offline": {
          "title": "Offline",
          "description": "We're offline for now – please check back later!",
          "thumbnail": {
            "width": 1920,
            "height": 1080,
            "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026.jpeg",
            "childImages": [
              {
                "width": 400,
                "height": 225,
                "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_400x225.jpeg"
              },
              {
                "width": 1200,
                "height": 675,
                "path": "https://pbs.floatplane.com/stream_thumbnails/5c13f3c006f1be15e08e05c0/894654974252956_1549059179026_1200x675.jpeg"
              }
            ]
          }
        }
      },
      "subscriptionPlans": [
        {
          "id": "5d48d0306825b5780db93d07",
          "title": "LTT Supporter (1080p)",
          "description": "Includes:\n- Early access (when possible)\n- Live Streaming\n- Behind-the-scenes, cutting room floor & exclusives\n\nNOTE: Tech Quickie and TechLinked are included for now, but will move to their own Floatplane pages in the future",
          "price": "5.00",
          "priceYearly": "50.00",
          "currency": "usd",
          "logo": null,
          "interval": "month",
          "featured": true,
          "allowGrandfatheredAccess": false,
          "discordServers": [],
          "discordRoles": []
        },
        {
          "id": "5e0ba6ac14e2590f760a0f0f",
          "title": "LTT Supporter Plus",
          "description": "You are the real MVP. \n\nYour support helps us continue to build out our team, drive up production values, run experiments that might lose money for a long time (*cough* LTX *cough*) and otherwise be the best content creators we can be.\n\nThis tier includes all the perks of the previous ones, but at floatplane's glorious high bitrate 4K!",
          "price": "10.00",
          "priceYearly": "100.00",
          "currency": "usd",
          "logo": null,
          "interval": "month",
          "featured": false,
          "allowGrandfatheredAccess": false,
          "discordServers": [],
          "discordRoles": []
        }
      ],
      "discoverable": true,
      "subscriberCountDisplay": "total",
      "incomeDisplay": false,
      "card": {
        "width": 375,
        "height": 500,
        "path": "https://pbs.floatplane.com/creator_card/59f94c0bdd241b70349eb72b/281467946609369_1551250329871.jpeg",
        "childImages": [
          {
            "width": 300,
            "height": 400,
            "path": "https://pbs.floatplane.com/creator_card/59f94c0bdd241b70349eb72b/281467946609369_1551250329871_300x400.jpeg"
          }
        ]
      }
    },
    "wasReleasedSilently": false,
    "thumbnail": {
      "width": 1920,
      "height": 1080,
      "path": "https://pbs.floatplane.com/blogPost_thumbnails/VkxlNv3k8j/438666910492097_1633734872237.jpeg",
      "childImages": [
        {
          "width": 400,
          "height": 225,
          "path": "https://pbs.floatplane.com/blogPost_thumbnails/VkxlNv3k8j/438666910492097_1633734872237_400x225.jpeg"
        },
        {
          "width": 1200,
          "height": 675,
          "path": "https://pbs.floatplane.com/blogPost_thumbnails/VkxlNv3k8j/438666910492097_1633734872237_1200x675.jpeg"
        }
      ]
    },
    "isAccessible": true
  }
]
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getrelatedblogposts-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Related post details|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getrelatedblogposts-responseschema">Response Schema</h3>

Status Code **200**

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|[[BlogPostModelV3](#schemablogpostmodelv3)]|false|none|none|
|» id|string|true|none|none|
|» guid|string|true|none|none|
|» title|string|true|none|none|
|» text|string|true|none|Text description of the post. May have HTML paragraph (`<p>`) tags surrounding it, along with other HTML..|
|» type|string|true|none|none|
|» channel|any|true|none|none|

*oneOf*

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|»» *anonymous*|[ChannelModel](#schemachannelmodel)|false|none|none|
|»»» id|string|true|none|none|
|»»» creator|string|true|none|none|
|»»» title|string|true|none|none|
|»»» urlname|string|true|none|Shown in the browser URL.|
|»»» about|string|true|none|none|
|»»» order|integer|false|none|none|
|»»» cover|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»»» width|integer|true|none|none|
|»»»» height|integer|true|none|none|
|»»»» path|string(uri)|true|none|none|
|»»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»»»»» width|integer|true|none|none|
|»»»»» height|integer|true|none|none|
|»»»»» path|string(uri)|true|none|none|
|»»» card|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»»» width|integer|true|none|none|
|»»»» height|integer|true|none|none|
|»»»» path|string(uri)|true|none|none|
|»»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»»» icon|[ImageModel](#schemaimagemodel)|true|none|none|
|»»»» width|integer|true|none|none|
|»»»» height|integer|true|none|none|
|»»»» path|string(uri)|true|none|none|
|»»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»»» socialLinks|[SocialLinksModel](#schemasociallinksmodel)|false|none|none|
|»»»» **additionalProperties**|string(uri)|false|none|none|

*xor*

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|»» *anonymous*|string|false|none|Channel identifier|

*continued*

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» tags|[string]|true|none|none|
|» attachmentOrder|[string]|true|none|none|
|» metadata|[PostMetadataModel](#schemapostmetadatamodel)|true|none|none|
|»» hasVideo|boolean|true|none|none|
|»» videoCount|integer|false|none|none|
|»» videoDuration|number|true|none|none|
|»» hasAudio|boolean|true|none|none|
|»» audioCount|integer|false|none|none|
|»» audioDuration|number|true|none|none|
|»» hasPicture|boolean|true|none|none|
|»» pictureCount|integer|false|none|none|
|»» hasGallery|boolean|false|none|none|
|»» galleryCount|integer|false|none|none|
|»» isFeatured|boolean|true|none|none|
|» releaseDate|string(date-time)|true|none|none|
|» likes|integer|true|none|none|
|» dislikes|integer|true|none|none|
|» score|integer|true|none|none|
|» comments|integer|true|none|none|
|» creator|object|true|none|none|
|»» id|string|true|none|none|
|»» owner|object|true|none|none|
|»»» id|string|true|none|none|
|»»» username|string|true|none|none|
|»» title|string|true|none|none|
|»» urlname|string|true|none|Shown in the browser URL, and used in `/creator/named` queries.|
|»» description|string|true|none|none|
|»» about|string|true|none|none|
|»» category|object|true|none|none|
|»»» id|string|true|none|none|
|»»» title|string|true|none|none|
|»» cover|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»» width|integer|true|none|none|
|»»» height|integer|true|none|none|
|»»» path|string(uri)|true|none|none|
|»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»» icon|[ImageModel](#schemaimagemodel)|true|none|none|
|»» liveStream|[LiveStreamModel](#schemalivestreammodel)¦null|true|none|none|
|»»» id|string|true|none|none|
|»»» title|string|true|none|none|
|»»» description|string|true|none|none|
|»»» thumbnail|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»»» width|integer|true|none|none|
|»»»» height|integer|true|none|none|
|»»»» path|string(uri)|true|none|none|
|»»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»»» owner|string|true|none|none|
|»»» channel|string|false|none|The creator channel this livestream belongs to.|
|»»» streamPath|string|true|none|none|
|»»» offline|object|true|none|none|
|»»»» title|string¦null|true|none|none|
|»»»» description|string¦null|true|none|none|
|»»»» thumbnail|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|»»»»» width|integer|true|none|none|
|»»»»» height|integer|true|none|none|
|»»»»» path|string(uri)|true|none|none|
|»»»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|»» subscriptionPlans|[[SubscriptionPlanModel](#schemasubscriptionplanmodel)]|true|none|none|
|»»» id|string|true|none|none|
|»»» title|string|true|none|none|
|»»» description|string|true|none|none|
|»»» price|string¦null|true|none|none|
|»»» priceYearly|string¦null|false|none|none|
|»»» currency|string|true|none|none|
|»»» logo|string¦null|true|none|none|
|»»» interval|string|true|none|none|
|»»» featured|boolean|true|none|none|
|»»» allowGrandfatheredAccess|boolean¦null|false|none|none|
|»»» discordServers|[[DiscordServerModel](#schemadiscordservermodel)]|true|none|none|
|»»»» id|string|true|none|none|
|»»»» guildName|string|true|none|none|
|»»»» guildIcon|string|true|none|none|
|»»»» inviteLink|string(uri)¦null|true|none|none|
|»»»» inviteMode|string|true|none|none|
|»»» discordRoles|[[DiscordRoleModel](#schemadiscordrolemodel)]|true|none|none|
|»»»» server|string|true|none|none|
|»»»» roleName|string|true|none|none|
|»» discoverable|boolean|true|none|none|
|»» subscriberCountDisplay|string|true|none|none|
|»» incomeDisplay|boolean|true|none|none|
|»» defaultChannel|string|false|none|none|
|»» channels|[string]|false|none|none|
|»» card|[ImageModel](#schemaimagemodel)¦null|false|none|none|
|»»» width|integer|true|none|none|
|»»» height|integer|true|none|none|
|»»» path|string(uri)|true|none|none|
|»»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|» wasReleasedSilently|boolean|true|none|none|
|» thumbnail|[ImageModel](#schemaimagemodel)¦null|false|none|none|
|»» width|integer|true|none|none|
|»» height|integer|true|none|none|
|»» path|string(uri)|true|none|none|
|»» childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|
|» isAccessible|boolean|true|none|If false, the post should be marked as locked and not viewable by the user.|
|» videoAttachments|[string]|false|none|May be undefined, usually when `isAccessible` is `false`.|
|» audioAttachments|[string]|false|none|May be undefined, usually when `isAccessible` is `false`.|
|» pictureAttachments|[string]|false|none|May be undefined, usually when `isAccessible` is `false`.|
|» galleryAttachments|[string]|false|none|May be undefined, usually when `isAccessible` is `false`.|

#### Enumerated Values

|Property|Value|
|---|---|
|type|blogPost|

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## getVideoContent

<a id="opIdgetVideoContent"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v3/content/video?id=string \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v3/content/video?id=string HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/content/video?id=string',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v3/content/video',
  params: {
  'id' => 'string'
}, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v3/content/video', params={
  'id': 'string'
}, headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v3/content/video', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/content/video?id=string");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v3/content/video", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v3/content/video`

*Get Video Content*

Retrieve more information on a video attachment from a blog post in order to consume the video content.

<h3 id="getvideocontent-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id|query|string|true|The ID of the video attachment object, from the `BlogPostModelV3`.|

> Example responses

> 200 Response

```json
{
  "id": "TViGzkuIic",
  "guid": "TViGzkuIic",
  "title": "October 9, 2021 @ 07:18 – First Linux Stream",
  "type": "video",
  "description": "",
  "releaseDate": null,
  "duration": 5689,
  "creator": "59f94c0bdd241b70349eb72b",
  "likes": 0,
  "dislikes": 0,
  "score": 0,
  "isProcessing": false,
  "primaryBlogPost": "Dw2ms0AgL8",
  "thumbnail": {
    "width": 1920,
    "height": 1080,
    "path": "https://pbs.floatplane.com/content_thumbnails/TViGzkuIic/324783659287024_1633769709593.jpeg",
    "childImages": []
  },
  "isAccessible": true,
  "blogPosts": [
    "Dw2ms0AgL8"
  ],
  "timelineSprite": {
    "width": 4960,
    "height": 2610,
    "path": "https://pbs.floatplane.com/timeline_sprite/TViGzkuIic/142493855372807_1633769996492.jpeg",
    "childImages": []
  },
  "userInteraction": [],
  "levels": [
    {
      "name": "360",
      "width": 640,
      "height": 360,
      "label": "360p",
      "order": 0
    },
    {
      "name": "480",
      "width": 854,
      "height": 480,
      "label": "480p",
      "order": 1
    },
    {
      "name": "720",
      "width": 1280,
      "height": 720,
      "label": "720p",
      "order": 2
    }
  ],
  "textTracks": [
    {
      "id": "6823ca1cb3c97e3f8cefe6bb",
      "src": "https://941d59f1900d0a09025413971374845d.r2.cloudflarestorage.com/prod-texttrack/text_tracks/BX3VEUGz2v/AQsb2my1nC6x6XX.vtt?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=848bccaff51d5141475de2d2a180f979%2F20250607%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20250607T191449Z&X-Amz-Expires=900&X-Amz-Signature=4787e98e0d22cd06aec5a0d76fa6829138a95582e4d831964004471ff48fb1c1&X-Amz-SignedHeaders=host&response-expires=Sat%2C%2007%20Jun%202025%2019%3A14%3A50%20GMT&x-amz-checksum-mode=ENABLED&x-id=GetObject",
      "kind": "captions",
      "language": "en",
      "generated": false,
      "processing": false
    }
  ]
}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getvideocontent-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK - Video details returned|[ContentVideoV3Response](#schemacontentvideov3response)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getvideocontent-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## getPictureContent

<a id="opIdgetPictureContent"></a>

> Code samples

```shell
# You can also use wget
curl -X GET https://www.floatplane.com/api/v3/content/picture?id=string \
  -H 'Accept: application/json'

```

```http
GET https://www.floatplane.com/api/v3/content/picture?id=string HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/content/picture?id=string',
{
  method: 'GET',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.get 'https://www.floatplane.com/api/v3/content/picture',
  params: {
  'id' => 'string'
}, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.get('https://www.floatplane.com/api/v3/content/picture', params={
  'id': 'string'
}, headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('GET','https://www.floatplane.com/api/v3/content/picture', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/content/picture?id=string");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("GET");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("GET", "https://www.floatplane.com/api/v3/content/picture", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`GET /api/v3/content/picture`

*Get Picture Content*

Retrieve more information on a picture attachment from a blog post in order to consume the picture content.

<h3 id="getpicturecontent-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|id|query|string|true|The ID of the picture attachment object, from the `BlogPostModelV3`.|

> Example responses

> 200 Response

```json
{
  "id": "ZWKdCy8TMN",
  "guid": "ZWKdCy8TMN",
  "title": "\"I hate costumes\" Jonathan",
  "type": "picture",
  "description": "",
  "likes": 1,
  "dislikes": 0,
  "score": 1,
  "isProcessing": false,
  "creator": "59f94c0bdd241b70349eb72b",
  "primaryBlogPost": "PGZBzzRWpD",
  "userInteraction": [],
  "thumbnail": {
    "width": 1200,
    "height": 675,
    "path": "https://pbs.floatplane.com/picture_thumbnails/ZWKdCy8TMN/239212458322156_1634845035660.jpeg",
    "childImages": []
  },
  "isAccessible": true,
  "imageFiles": [
    {
      "path": "https://pbs.floatplane.com/content_images/59f94c0bdd241b70349eb72b/465975275316873_1634845031494_1164x675.jpeg?AWSAccessKeyId=...&Expires=...&Signature=...",
      "width": 1164,
      "height": 675,
      "size": 165390
    }
  ]
}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getpicturecontent-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|[ContentPictureV3Response](#schemacontentpicturev3response)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getpicturecontent-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## likeContent

<a id="opIdlikeContent"></a>

> Code samples

```shell
# You can also use wget
curl -X POST https://www.floatplane.com/api/v3/content/like \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json'

```

```http
POST https://www.floatplane.com/api/v3/content/like HTTP/1.1
Host: www.floatplane.com
Content-Type: application/json
Accept: application/json

```

```javascript
const inputBody = '{
  "contentType": "blogPost",
  "id": "string"
}';
const headers = {
  'Content-Type':'application/json',
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/content/like',
{
  method: 'POST',
  body: inputBody,
  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Content-Type' => 'application/json',
  'Accept' => 'application/json'
}

result = RestClient.post 'https://www.floatplane.com/api/v3/content/like',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Content-Type': 'application/json',
  'Accept': 'application/json'
}

r = requests.post('https://www.floatplane.com/api/v3/content/like', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Content-Type' => 'application/json',
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('POST','https://www.floatplane.com/api/v3/content/like', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/content/like");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("POST");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Content-Type": []string{"application/json"},
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("POST", "https://www.floatplane.com/api/v3/content/like", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`POST /api/v3/content/like`

*Like Content*

Toggles the like status on a piece of content. If disliked before, it will turn into a like. If liked before, the like will be removed.

> Body parameter

```json
{
  "contentType": "blogPost",
  "id": "string"
}
```

<h3 id="likecontent-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|[ContentLikeV3Request](#schemacontentlikev3request)|true|none|

> Example responses

> 200 Response

```json
[
  "like"
]
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="likecontent-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|[UserInteractionModel](#schemauserinteractionmodel)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="likecontent-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## dislikeContent

<a id="opIddislikeContent"></a>

> Code samples

```shell
# You can also use wget
curl -X POST https://www.floatplane.com/api/v3/content/dislike \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json'

```

```http
POST https://www.floatplane.com/api/v3/content/dislike HTTP/1.1
Host: www.floatplane.com
Content-Type: application/json
Accept: application/json

```

```javascript
const inputBody = '{
  "contentType": "blogPost",
  "id": "string"
}';
const headers = {
  'Content-Type':'application/json',
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/content/dislike',
{
  method: 'POST',
  body: inputBody,
  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Content-Type' => 'application/json',
  'Accept' => 'application/json'
}

result = RestClient.post 'https://www.floatplane.com/api/v3/content/dislike',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Content-Type': 'application/json',
  'Accept': 'application/json'
}

r = requests.post('https://www.floatplane.com/api/v3/content/dislike', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Content-Type' => 'application/json',
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('POST','https://www.floatplane.com/api/v3/content/dislike', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/content/dislike");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("POST");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Content-Type": []string{"application/json"},
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("POST", "https://www.floatplane.com/api/v3/content/dislike", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`POST /api/v3/content/dislike`

*Dislike Content*

Toggles the dislike status on a piece of content. If liked before, it will turn into a dislike. If disliked before, the dislike will be removed.

> Body parameter

```json
{
  "contentType": "blogPost",
  "id": "string"
}
```

<h3 id="dislikecontent-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|[ContentLikeV3Request](#schemacontentlikev3request)|true|none|

> Example responses

> 200 Response

```json
[
  "dislike"
]
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="dislikecontent-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|[UserInteractionModel](#schemauserinteractionmodel)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="dislikecontent-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## updateProgress

<a id="opIdupdateProgress"></a>

> Code samples

```shell
# You can also use wget
curl -X POST https://www.floatplane.com/api/v3/content/progress \
  -H 'Content-Type: application/json' \
  -H 'Accept: text/plain'

```

```http
POST https://www.floatplane.com/api/v3/content/progress HTTP/1.1
Host: www.floatplane.com
Content-Type: application/json
Accept: text/plain

```

```javascript
const inputBody = '{
  "id": "string",
  "contentType": "video",
  "progress": 0
}';
const headers = {
  'Content-Type':'application/json',
  'Accept':'text/plain'
};

fetch('https://www.floatplane.com/api/v3/content/progress',
{
  method: 'POST',
  body: inputBody,
  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Content-Type' => 'application/json',
  'Accept' => 'text/plain'
}

result = RestClient.post 'https://www.floatplane.com/api/v3/content/progress',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Content-Type': 'application/json',
  'Accept': 'text/plain'
}

r = requests.post('https://www.floatplane.com/api/v3/content/progress', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Content-Type' => 'application/json',
    'Accept' => 'text/plain',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('POST','https://www.floatplane.com/api/v3/content/progress', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/content/progress");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("POST");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Content-Type": []string{"application/json"},
        "Accept": []string{"text/plain"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("POST", "https://www.floatplane.com/api/v3/content/progress", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`POST /api/v3/content/progress`

*Update Progress*

Update the watch progress on a piece of media (usually video or audio), stored as the number of seconds in the media.

> Body parameter

```json
{
  "id": "string",
  "contentType": "video",
  "progress": 0
}
```

<h3 id="updateprogress-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|[UpdateProgressRequest](#schemaupdateprogressrequest)|true|none|

> Example responses

> 200 Response

```
"OK"
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="updateprogress-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|string|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="updateprogress-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## getProgress

<a id="opIdgetProgress"></a>

> Code samples

```shell
# You can also use wget
curl -X POST https://www.floatplane.com/api/v3/content/get/progress \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json'

```

```http
POST https://www.floatplane.com/api/v3/content/get/progress HTTP/1.1
Host: www.floatplane.com
Content-Type: application/json
Accept: application/json

```

```javascript
const inputBody = '{
  "ids": [
    "string"
  ],
  "contentType": "blogPost"
}';
const headers = {
  'Content-Type':'application/json',
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/content/get/progress',
{
  method: 'POST',
  body: inputBody,
  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Content-Type' => 'application/json',
  'Accept' => 'application/json'
}

result = RestClient.post 'https://www.floatplane.com/api/v3/content/get/progress',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Content-Type': 'application/json',
  'Accept': 'application/json'
}

r = requests.post('https://www.floatplane.com/api/v3/content/get/progress', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Content-Type' => 'application/json',
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('POST','https://www.floatplane.com/api/v3/content/get/progress', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/content/get/progress");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("POST");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Content-Type": []string{"application/json"},
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("POST", "https://www.floatplane.com/api/v3/content/get/progress", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`POST /api/v3/content/get/progress`

*Get Progress*

Batch retrieval of watch progress values for blog posts. This API is useful for showing progress of a list of blog posts shown on the screen to the user. When retrieving a list of blog posts, the media attachments only include the identifier; when retrieving full details of a blog post, the attachments include more information, but still fail to return the progress of the media. Only when pulling the full video/audio content does the progress get included in the response. Thus, the recommended approach is to pull paginated results of blog posts first, as usual, and then to call this endpoint to retrieve progress values for each blog post to show in some capacity, usually on the thumbnail as a progress bar on the bottom.

Note that the progress values returned in this endpoint are different from the update progress endpoint and the values returned in video/audio attachments. While the latter are measured in seconds, this endpoint returns progress as a percentage of the media's total duration. It is presumed that the progress returned is from the first attachment in the blog post's `attachmentOrder` that is either a video or audio attachment. Because this returns progress as an integer percentage (0 to 100), it is not recommended to use this particular value for jumping to a timestamp in the media when resuming playback, as the rounded number may be off by plus/minus several seconds in actual playback. Use the actual attachment progress, measured in seconds, instead.

> Body parameter

```json
{
  "ids": [
    "string"
  ],
  "contentType": "blogPost"
}
```

<h3 id="getprogress-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|[GetProgressRequest](#schemagetprogressrequest)|true|none|

> Example responses

> 200 Response

```json
[
  {
    "id": "3qZrz5ugsa",
    "progress": 0
  },
  {
    "id": "mhhrOfndeF",
    "progress": 100
  }
]
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="getprogress-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|[GetProgressResponse](#schemagetprogressresponse)|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="getprogress-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

<h1 id="floatplane-rest-api-pollv3">PollV3</h1>

Poll voting and management.

## joinLiveRoom

<a id="opIdjoinLiveRoom"></a>

> Code samples

```shell
# You can also use wget
curl -X POST https://www.floatplane.com/api/v3/poll/live/joinroom \
  -H 'Accept: application/json'

```

```http
POST https://www.floatplane.com/api/v3/poll/live/joinroom HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/poll/live/joinroom',
{
  method: 'POST',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.post 'https://www.floatplane.com/api/v3/poll/live/joinroom',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.post('https://www.floatplane.com/api/v3/poll/live/joinroom', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('POST','https://www.floatplane.com/api/v3/poll/live/joinroom', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/poll/live/joinroom");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("POST");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("POST", "https://www.floatplane.com/api/v3/poll/live/joinroom", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`POST /api/v3/poll/live/joinroom`

*Poll Join Live Room*

Used in Socket.IO/WebSocket connections. See the AsyncAPI documentation for more information. This should not be used on a raw HTTP connection.

> Example responses

> 200 Response

```json
{}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="joinliveroom-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="joinliveroom-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## leaveLiveRoom

<a id="opIdleaveLiveRoom"></a>

> Code samples

```shell
# You can also use wget
curl -X POST https://www.floatplane.com/api/v3/poll/live/leaveLiveRoom \
  -H 'Accept: application/json'

```

```http
POST https://www.floatplane.com/api/v3/poll/live/leaveLiveRoom HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/poll/live/leaveLiveRoom',
{
  method: 'POST',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.post 'https://www.floatplane.com/api/v3/poll/live/leaveLiveRoom',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.post('https://www.floatplane.com/api/v3/poll/live/leaveLiveRoom', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('POST','https://www.floatplane.com/api/v3/poll/live/leaveLiveRoom', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/poll/live/leaveLiveRoom");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("POST");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("POST", "https://www.floatplane.com/api/v3/poll/live/leaveLiveRoom", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`POST /api/v3/poll/live/leaveLiveRoom`

*Poll Leave Live Room*

Used in Socket.IO/WebSocket connections. See the AsyncAPI documentation for more information. This should not be used on a raw HTTP connection.

> Example responses

> 200 Response

```json
{}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="leaveliveroom-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="leaveliveroom-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

## votePoll

<a id="opIdvotePoll"></a>

> Code samples

```shell
# You can also use wget
curl -X POST https://www.floatplane.com/api/v3/poll/votePoll \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json'

```

```http
POST https://www.floatplane.com/api/v3/poll/votePoll HTTP/1.1
Host: www.floatplane.com
Content-Type: application/json
Accept: application/json

```

```javascript
const inputBody = '{
  "pollId": "string",
  "optionIndex": 0
}';
const headers = {
  'Content-Type':'application/json',
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/poll/votePoll',
{
  method: 'POST',
  body: inputBody,
  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Content-Type' => 'application/json',
  'Accept' => 'application/json'
}

result = RestClient.post 'https://www.floatplane.com/api/v3/poll/votePoll',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Content-Type': 'application/json',
  'Accept': 'application/json'
}

r = requests.post('https://www.floatplane.com/api/v3/poll/votePoll', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Content-Type' => 'application/json',
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('POST','https://www.floatplane.com/api/v3/poll/votePoll', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/poll/votePoll");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("POST");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Content-Type": []string{"application/json"},
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("POST", "https://www.floatplane.com/api/v3/poll/votePoll", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`POST /api/v3/poll/votePoll`

*Vote Poll*

Vote on an option of a poll. Voting a second time or attempting to change a choice may result in an error.

> Body parameter

```json
{
  "pollId": "string",
  "optionIndex": 0
}
```

<h3 id="votepoll-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|body|body|object|true|none|
|» pollId|body|string|true|The id of the poll to vote on.|
|» optionIndex|body|integer|true|The index of the options of the poll for which to vote. This should not be outside the bounds of the poll options.|

> Example responses

> 200 Response

```json
{}
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="votepoll-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="votepoll-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

<h1 id="floatplane-rest-api-redirectv3">RedirectV3</h1>

Channel-level redirection to Youtube or other sites for latest videos outside of Floatplane.

## redirectYTLatest

<a id="opIdredirectYTLatest"></a>

> Code samples

```shell
# You can also use wget
curl -X POST https://www.floatplane.com/api/v3/redirect-yt-latest/{channelKey} \
  -H 'Accept: application/json'

```

```http
POST https://www.floatplane.com/api/v3/redirect-yt-latest/{channelKey} HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/redirect-yt-latest/{channelKey}',
{
  method: 'POST',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.post 'https://www.floatplane.com/api/v3/redirect-yt-latest/{channelKey}',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.post('https://www.floatplane.com/api/v3/redirect-yt-latest/{channelKey}', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('POST','https://www.floatplane.com/api/v3/redirect-yt-latest/{channelKey}', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/redirect-yt-latest/{channelKey}");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("POST");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("POST", "https://www.floatplane.com/api/v3/redirect-yt-latest/{channelKey}", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`POST /api/v3/redirect-yt-latest/{channelKey}`

*Redirect to YouTube Latest Video*

Redirects (HTTP 302) the user to the latest LMG video for a given LMG channel key. For example, visiting this URL with a `channelKey` of `sc`, it will take you directly to the latest Short Circuit video on YouTube. Unknown if this works for non-LMG creators for their channels. Not used in Floatplane code.

<h3 id="redirectytlatest-parameters">Parameters</h3>

|Name|In|Type|Required|Description|
|---|---|---|---|---|
|channelKey|path|string|true|none|

> Example responses

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="redirectytlatest-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|302|[Found](https://tools.ietf.org/html/rfc7231#section-6.4.3)|Found|None|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="redirectytlatest-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|302|Location|string||A YouTube URL for a video.|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

<h1 id="floatplane-rest-api-loyaltyrewardsv3">LoyaltyRewardsV3</h1>

Loyalty rewards information and claiming.

## listCreatorLoyaltyReward

<a id="opIdlistCreatorLoyaltyReward"></a>

> Code samples

```shell
# You can also use wget
curl -X POST https://www.floatplane.com/api/v3/user/loyaltyreward/list \
  -H 'Accept: application/json'

```

```http
POST https://www.floatplane.com/api/v3/user/loyaltyreward/list HTTP/1.1
Host: www.floatplane.com
Accept: application/json

```

```javascript

const headers = {
  'Accept':'application/json'
};

fetch('https://www.floatplane.com/api/v3/user/loyaltyreward/list',
{
  method: 'POST',

  headers: headers
})
.then(function(res) {
    return res.json();
}).then(function(body) {
    console.log(body);
});

```

```ruby
require 'rest-client'
require 'json'

headers = {
  'Accept' => 'application/json'
}

result = RestClient.post 'https://www.floatplane.com/api/v3/user/loyaltyreward/list',
  params: {
  }, headers: headers

p JSON.parse(result)

```

```python
import requests
headers = {
  'Accept': 'application/json'
}

r = requests.post('https://www.floatplane.com/api/v3/user/loyaltyreward/list', headers = headers)

print(r.json())

```

```php
<?php

require 'vendor/autoload.php';

$headers = array(
    'Accept' => 'application/json',
);

$client = new \GuzzleHttp\Client();

// Define array of request body.
$request_body = array();

try {
    $response = $client->request('POST','https://www.floatplane.com/api/v3/user/loyaltyreward/list', array(
        'headers' => $headers,
        'json' => $request_body,
       )
    );
    print_r($response->getBody()->getContents());
 }
 catch (\GuzzleHttp\Exception\BadResponseException $e) {
    // handle exception or api errors.
    print_r($e->getMessage());
 }

 // ...

```

```java
URL obj = new URL("https://www.floatplane.com/api/v3/user/loyaltyreward/list");
HttpURLConnection con = (HttpURLConnection) obj.openConnection();
con.setRequestMethod("POST");
int responseCode = con.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer response = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
System.out.println(response.toString());

```

```go
package main

import (
       "bytes"
       "net/http"
)

func main() {

    headers := map[string][]string{
        "Accept": []string{"application/json"},
    }

    data := bytes.NewBuffer([]byte{jsonReq})
    req, err := http.NewRequest("POST", "https://www.floatplane.com/api/v3/user/loyaltyreward/list", data)
    req.Header = headers

    client := &http.Client{}
    resp, err := client.Do(req)
    // ...
}

```

`POST /api/v3/user/loyaltyreward/list`

*List Creator Loyalty Reward*

Retrieve a list of loyalty rewards for the user. The reason for why this is a POST and not a GET is unknown.

> Example responses

> 200 Response

```json
[]
```

> 400 Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

> 401 Response

```json
{
  "id": "erng-ah8e-n0d3",
  "errors": [
    {
      "id": "erng-ah8e-n0d3",
      "name": "notLoggedInError",
      "message": "You must be logged-in to access this resource."
    }
  ],
  "message": "You must be logged-in to access this resource."
}
```

> 403 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "missingAchievementError",
      "message": "You lack one or more of the required achievements needed to access the requested resource.",
      "data": {
        "requiresAllOfAchievement": [
          {
            "id": "6157853e479315db795f7296",
            "title": "FloatVPN Alpha",
            "startDate": null,
            "endDate": null,
            "icon": null
          }
        ]
      }
    }
  ],
  "message": "You lack one or more of the required achievements needed to access the requested resource."
}
```

```
"string"
```

> 404 Response

```json
{
  "id": "f4ec-orux-hds2",
  "errors": [
    {
      "id": "f4ec-orux-hds2",
      "name": "notFoundError"
    }
  ]
}
```

> default Response

```json
{
  "id": "awoz-3s5g-6amf",
  "errors": [
    {
      "id": "9edc-zejt-n3hb",
      "name": "paramValidationError",
      "message": "\"captchaToken\" must be an object",
      "data": {
        "rule": "object.base"
      }
    }
  ],
  "message": "\"captchaToken\" must be an object"
}
```

<h3 id="listcreatorloyaltyreward-responses">Responses</h3>

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|OK|Inline|
|400|[Bad Request](https://tools.ietf.org/html/rfc7231#section-6.5.1)|Bad Request - The request has errors and the server did not process it.|[ErrorModel](#schemaerrormodel)|
|401|[Unauthorized](https://tools.ietf.org/html/rfc7235#section-3.1)|Unauthenticated - The request was not authenticated to make the request.|[ErrorModel](#schemaerrormodel)|
|403|[Forbidden](https://tools.ietf.org/html/rfc7231#section-6.5.3)|Forbidden - The request was not authenticated to make the request.|string|
|404|[Not Found](https://tools.ietf.org/html/rfc7231#section-6.5.4)|Not Found - The resource was not found.|[ErrorModel](#schemaerrormodel)|
|429|[Too Many Requests](https://tools.ietf.org/html/rfc6585#section-4)|Too Many Requests - The resource was requested too many times|None|
|default|Default|Unexpected response code|[ErrorModel](#schemaerrormodel)|

<h3 id="listcreatorloyaltyreward-responseschema">Response Schema</h3>

### Response Headers

|Status|Header|Type|Format|Description|
|---|---|---|---|---|
|429|Retry-After|integer||The number of seconds the client must wait until future requests will respond normally.|

<aside class="warning">
To perform this operation, you must be authenticated by means of one of the following methods:
CookieAuth
</aside>

# Schemas

<h2 id="tocS_AuthLoginV2Request">AuthLoginV2Request</h2>
<!-- backwards compatibility -->
<a id="schemaauthloginv2request"></a>
<a id="schema_AuthLoginV2Request"></a>
<a id="tocSauthloginv2request"></a>
<a id="tocsauthloginv2request"></a>

```json
{
  "username": "string",
  "password": "string",
  "captchaToken": "string"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|username|string|true|none|none|
|password|string|true|none|none|
|captchaToken|string|false|none|The Google Recaptcha v2/v3 token to verify the request. On web browsers, this is required. For mobile or TV applications, this is not required only if the User-Agent indicates so (e.g., if the User-Agent contains "CFNetwork" in its value). Otherwise, the application would have to supply a valid captcha token, which can be difficult to obtain dynamically in some scenarios. In this case, this should be undefined (no key), not null.|

<h2 id="tocS_AuthLoginV2Response">AuthLoginV2Response</h2>
<!-- backwards compatibility -->
<a id="schemaauthloginv2response"></a>
<a id="schema_AuthLoginV2Response"></a>
<a id="tocSauthloginv2response"></a>
<a id="tocsauthloginv2response"></a>

```json
{
  "user": {
    "id": "string",
    "username": "string",
    "profileImage": {
      "width": 0,
      "height": 0,
      "path": "http://example.com",
      "childImages": [
        {
          "width": 0,
          "height": 0,
          "path": "http://example.com"
        }
      ]
    }
  },
  "needs2FA": true
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|user|[UserModel](#schemausermodel)|false|none|Identifying information about the new-logged-in user upon success. May be undefined when `needs2FA` is `true`.|
|needs2FA|boolean|true|none|If true, the user has not yet been authenticated, and will need to submit the 2FA token to complete authentication.|

<h2 id="tocS_CheckFor2faLoginRequest">CheckFor2faLoginRequest</h2>
<!-- backwards compatibility -->
<a id="schemacheckfor2faloginrequest"></a>
<a id="schema_CheckFor2faLoginRequest"></a>
<a id="tocScheckfor2faloginrequest"></a>
<a id="tocscheckfor2faloginrequest"></a>

```json
{
  "token": "string"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|token|string|true|none|The two-factor authentication token that the user inputs to complete the login process.|

<h2 id="tocS_CdnDeliveryV2QualityLevelModel">CdnDeliveryV2QualityLevelModel</h2>
<!-- backwards compatibility -->
<a id="schemacdndeliveryv2qualitylevelmodel"></a>
<a id="schema_CdnDeliveryV2QualityLevelModel"></a>
<a id="tocScdndeliveryv2qualitylevelmodel"></a>
<a id="tocscdndeliveryv2qualitylevelmodel"></a>

```json
{
  "name": "string",
  "width": 0,
  "height": 0,
  "label": "string",
  "order": 0,
  "mimeType": "string",
  "codecs": "string"
}

```

Represents a quality of video to download/stream.

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|name|string|true|none|Used to identify this level of quality, and to refer to the `qualityLevelParams` object below by the property key.|
|width|integer¦null|false|none|The video quality's resolution's width in pixels.|
|height|integer¦null|false|none|The video quality resolution's height in pixels.|
|label|string|true|none|The display-friendly version of `name`.|
|order|integer|true|none|The display order to be shown to the user.|
|mimeType|string¦null|false|none|none|
|codecs|string¦null|false|none|none|

<h2 id="tocS_CdnDeliveryV2ResourceModel">CdnDeliveryV2ResourceModel</h2>
<!-- backwards compatibility -->
<a id="schemacdndeliveryv2resourcemodel"></a>
<a id="schema_CdnDeliveryV2ResourceModel"></a>
<a id="tocScdndeliveryv2resourcemodel"></a>
<a id="tocscdndeliveryv2resourcemodel"></a>

```json
{
  "uri": "string",
  "data": {
    "qualityLevels": [
      {
        "name": "string",
        "width": 0,
        "height": 0,
        "label": "string",
        "order": 0,
        "mimeType": "string",
        "codecs": "string"
      }
    ],
    "qualityLevelParams": {}
  }
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|uri|string|true|none|The path to attach to the `cdn` property above. Replace the items surrounded by curly braces (`{`, `}`) with the appropriate values from the `data` property, depending on chosen resolution. First, choose the `qualityLevel`, then use the given token from the `qualityLevelParam` for that `qualityLevel`'s `name`.|
|data|object|true|none|none|
|» qualityLevels|[[CdnDeliveryV2QualityLevelModel](#schemacdndeliveryv2qualitylevelmodel)]|false|none|[Represents a quality of video to download/stream.]|
|» qualityLevelParams|object¦null|false|none|For each `qualityLevel` above, there will be an entry in this map where the property name matches the `qulityLevel[].name` containing a token to apply to the URL.|

<h2 id="tocS_CdnDeliveryV2VodLivestreamResponse">CdnDeliveryV2VodLivestreamResponse</h2>
<!-- backwards compatibility -->
<a id="schemacdndeliveryv2vodlivestreamresponse"></a>
<a id="schema_CdnDeliveryV2VodLivestreamResponse"></a>
<a id="tocScdndeliveryv2vodlivestreamresponse"></a>
<a id="tocscdndeliveryv2vodlivestreamresponse"></a>

```json
{
  "cdn": "http://example.com",
  "strategy": "cdn",
  "resource": {
    "uri": "string",
    "data": {
      "qualityLevels": [
        {
          "name": "string",
          "width": 0,
          "height": 0,
          "label": "string",
          "order": 0,
          "mimeType": "string",
          "codecs": "string"
        }
      ],
      "qualityLevelParams": {}
    }
  }
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|cdn|string(uri)|true|none|The domain of the CDN server to use. Combine with data from the `resource` object to create a full URL.|
|strategy|string|true|none|Which download/streaming strategy to use. If `cdn`, then a `cdn` property will be included with the response. Otherwise, if set to `client`, then a `client` property will be included with the response. The cdn or client property should be combined with the `resource` property to perform the download/stream.|
|resource|[CdnDeliveryV2ResourceModel](#schemacdndeliveryv2resourcemodel)|true|none|none|

#### Enumerated Values

|Property|Value|
|---|---|
|strategy|cdn|
|strategy|client|

<h2 id="tocS_CdnDeliveryV2DownloadResponse">CdnDeliveryV2DownloadResponse</h2>
<!-- backwards compatibility -->
<a id="schemacdndeliveryv2downloadresponse"></a>
<a id="schema_CdnDeliveryV2DownloadResponse"></a>
<a id="tocScdndeliveryv2downloadresponse"></a>
<a id="tocscdndeliveryv2downloadresponse"></a>

```json
{
  "edges": [
    {
      "hostname": "string",
      "queryPort": 0,
      "bandwidth": 0,
      "allowDownload": true,
      "allowStreaming": true,
      "datacenter": {
        "countryCode": "string",
        "regionCode": "string",
        "latitude": 0,
        "longitude": 0
      }
    }
  ],
  "client": {},
  "strategy": "cdn",
  "resource": {
    "uri": "string",
    "data": {
      "qualityLevels": [
        {
          "name": "string",
          "width": 0,
          "height": 0,
          "label": "string",
          "order": 0,
          "mimeType": "string",
          "codecs": "string"
        }
      ],
      "qualityLevelParams": {}
    }
  }
}

```

### Properties

allOf

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|[EdgesModel](#schemaedgesmodel)|false|none|none|

and

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|object|false|none|none|
|» strategy|string|true|none|Which download/streaming strategy to use. If `cdn`, then a `cdn` property will be included with the response. Otherwise, if set to `client`, then a `client` property will be included with the response. The cdn or client property should be combined with the `resource` property to perform the download/stream.|
|» resource|[CdnDeliveryV2ResourceModel](#schemacdndeliveryv2resourcemodel)|true|none|none|

#### Enumerated Values

|Property|Value|
|---|---|
|strategy|cdn|
|strategy|client|

<h2 id="tocS_CdnDeliveryV2Response">CdnDeliveryV2Response</h2>
<!-- backwards compatibility -->
<a id="schemacdndeliveryv2response"></a>
<a id="schema_CdnDeliveryV2Response"></a>
<a id="tocScdndeliveryv2response"></a>
<a id="tocscdndeliveryv2response"></a>

```json
{
  "cdn": "http://example.com",
  "strategy": "cdn",
  "resource": {
    "uri": "string",
    "data": {
      "qualityLevels": [
        {
          "name": "string",
          "width": 0,
          "height": 0,
          "label": "string",
          "order": 0,
          "mimeType": "string",
          "codecs": "string"
        }
      ],
      "qualityLevelParams": {}
    }
  }
}

```

### Properties

oneOf

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|[CdnDeliveryV2VodLivestreamResponse](#schemacdndeliveryv2vodlivestreamresponse)|false|none|none|

xor

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|[CdnDeliveryV2DownloadResponse](#schemacdndeliveryv2downloadresponse)|false|none|none|

<h2 id="tocS_CdnDeliveryV3Response">CdnDeliveryV3Response</h2>
<!-- backwards compatibility -->
<a id="schemacdndeliveryv3response"></a>
<a id="schema_CdnDeliveryV3Response"></a>
<a id="tocScdndeliveryv3response"></a>
<a id="tocscdndeliveryv3response"></a>

```json
{
  "groups": [
    {
      "origins": [
        {
          "url": "http://example.com",
          "queryUrl": "http://example.com",
          "datacenter": {
            "countryCode": "string",
            "regionCode": "string",
            "latitude": 0,
            "longitude": 0
          }
        }
      ],
      "variants": [
        {
          "name": "string",
          "label": "string",
          "url": "string",
          "origins": [
            {
              "url": "http://example.com",
              "queryUrl": "http://example.com",
              "datacenter": {
                "countryCode": "string",
                "regionCode": "string",
                "latitude": 0,
                "longitude": 0
              }
            }
          ],
          "order": 0,
          "enabled": true,
          "hidden": true,
          "meta": {
            "common": {
              "size": 0,
              "access": {
                "deniedReason": "isMissingPermission",
                "deniedMessage": "string"
              }
            },
            "video": {
              "codec": "string",
              "codecSimple": "string",
              "mimeType": "string",
              "width": 0,
              "height": 0,
              "isHdr": true,
              "fps": 0,
              "bitrate": {
                "maximum": 0,
                "average": 0
              }
            },
            "audio": {
              "codec": "string",
              "codecSimple": "string",
              "mimeType": "string",
              "channelCount": 0,
              "samplerate": 0,
              "bitrate": {
                "maximum": 0,
                "average": 0
              }
            },
            "image": {
              "codec": "string",
              "codecSimple": "string",
              "mimeType": "string",
              "width": 0,
              "height": 0,
              "isHdr": true
            },
            "live": {
              "lowLatencyExtension": "llhls"
            }
          },
          "mimeType": "string"
        }
      ]
    }
  ]
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|groups|[[CdnDeliveryV3Group](#schemacdndeliveryv3group)]|true|none|`groups` may consist of zero or more elements.|

<h2 id="tocS_CdnDeliveryV3Group">CdnDeliveryV3Group</h2>
<!-- backwards compatibility -->
<a id="schemacdndeliveryv3group"></a>
<a id="schema_CdnDeliveryV3Group"></a>
<a id="tocScdndeliveryv3group"></a>
<a id="tocscdndeliveryv3group"></a>

```json
{
  "origins": [
    {
      "url": "http://example.com",
      "queryUrl": "http://example.com",
      "datacenter": {
        "countryCode": "string",
        "regionCode": "string",
        "latitude": 0,
        "longitude": 0
      }
    }
  ],
  "variants": [
    {
      "name": "string",
      "label": "string",
      "url": "string",
      "origins": [
        {
          "url": "http://example.com",
          "queryUrl": "http://example.com",
          "datacenter": {
            "countryCode": "string",
            "regionCode": "string",
            "latitude": 0,
            "longitude": 0
          }
        }
      ],
      "order": 0,
      "enabled": true,
      "hidden": true,
      "meta": {
        "common": {
          "size": 0,
          "access": {
            "deniedReason": "isMissingPermission",
            "deniedMessage": "string"
          }
        },
        "video": {
          "codec": "string",
          "codecSimple": "string",
          "mimeType": "string",
          "width": 0,
          "height": 0,
          "isHdr": true,
          "fps": 0,
          "bitrate": {
            "maximum": 0,
            "average": 0
          }
        },
        "audio": {
          "codec": "string",
          "codecSimple": "string",
          "mimeType": "string",
          "channelCount": 0,
          "samplerate": 0,
          "bitrate": {
            "maximum": 0,
            "average": 0
          }
        },
        "image": {
          "codec": "string",
          "codecSimple": "string",
          "mimeType": "string",
          "width": 0,
          "height": 0,
          "isHdr": true
        },
        "live": {
          "lowLatencyExtension": "llhls"
        }
      },
      "mimeType": "string"
    }
  ]
}

```

A group is a logical grouping/separation of variants. At this time, there are no examples of more than one group in a response.

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|origins|[[CdnDeliveryV3Origin](#schemacdndeliveryv3origin)]|false|none|If `origins` is present, it will consist of one or more elements.|
|variants|[[CdnDeliveryV3Variant](#schemacdndeliveryv3variant)]|true|none|`variants` may consist of zero or more elements.|

<h2 id="tocS_CdnDeliveryV3Origin">CdnDeliveryV3Origin</h2>
<!-- backwards compatibility -->
<a id="schemacdndeliveryv3origin"></a>
<a id="schema_CdnDeliveryV3Origin"></a>
<a id="tocScdndeliveryv3origin"></a>
<a id="tocscdndeliveryv3origin"></a>

```json
{
  "url": "http://example.com",
  "queryUrl": "http://example.com",
  "datacenter": {
    "countryCode": "string",
    "regionCode": "string",
    "latitude": 0,
    "longitude": 0
  }
}

```

An `origin`, if present, is a choice of base URL or server from which to load a `variant`'s content. If origin(s) exists in a group or variant, then one must be chosen in combination with the variant's `url`.

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|url|string(uri)|true|none|An absolute URL (possibly with trailing slash) which acts as the base of a delivery resource URI. This is always present.|
|queryUrl|string(uri)|false|none|An absolute URL (possibly with trailing slash) which the client can use to query if the origin is active/working. This field may not be present. Perform an HTTP GET on this URL and expect an HTTP 200 in order to trust this origin.|
|datacenter|[EdgeDataCenter](#schemaedgedatacenter)|false|none|Location information for a datacenter. Not required.|

<h2 id="tocS_CdnDeliveryV3Variant">CdnDeliveryV3Variant</h2>
<!-- backwards compatibility -->
<a id="schemacdndeliveryv3variant"></a>
<a id="schema_CdnDeliveryV3Variant"></a>
<a id="tocScdndeliveryv3variant"></a>
<a id="tocscdndeliveryv3variant"></a>

```json
{
  "name": "string",
  "label": "string",
  "url": "string",
  "origins": [
    {
      "url": "http://example.com",
      "queryUrl": "http://example.com",
      "datacenter": {
        "countryCode": "string",
        "regionCode": "string",
        "latitude": 0,
        "longitude": 0
      }
    }
  ],
  "order": 0,
  "enabled": true,
  "hidden": true,
  "meta": {
    "common": {
      "size": 0,
      "access": {
        "deniedReason": "isMissingPermission",
        "deniedMessage": "string"
      }
    },
    "video": {
      "codec": "string",
      "codecSimple": "string",
      "mimeType": "string",
      "width": 0,
      "height": 0,
      "isHdr": true,
      "fps": 0,
      "bitrate": {
        "maximum": 0,
        "average": 0
      }
    },
    "audio": {
      "codec": "string",
      "codecSimple": "string",
      "mimeType": "string",
      "channelCount": 0,
      "samplerate": 0,
      "bitrate": {
        "maximum": 0,
        "average": 0
      }
    },
    "image": {
      "codec": "string",
      "codecSimple": "string",
      "mimeType": "string",
      "width": 0,
      "height": 0,
      "isHdr": true
    },
    "live": {
      "lowLatencyExtension": "llhls"
    }
  },
  "mimeType": "string"
}

```

A `variant` represents one variant of a source of media. The most common differenitating factor between variants is video resolution, but there may be more variations based on `isHdr`, codecs, FPS, etc. It's possible that groups of variants may be divided into separate `groups` elements.

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|name|string|true|none|A programmatic name for this variant, for use with uniquely identifying this variant.|
|label|string|true|none|A display-friendly label for this variant, for use in the UI.|
|url|string|true|none|A relative *or* absolute URL containing resource information for this variant. Compared to the V2 API, this URL does not contain template information and will not need to be modified before use, other than optionally combining with an origin. This value may contain a trailing slash.<br><br>If this URL is absolute, it may be used as-is in order to load the media content. If this URL is relative, then it should be combined with an origin base URL. In order of preference: 1) use an origin from this variant object, 2) use an origin from this variant's group object, 3) use `https://floatplane.com`.<br><br>Do not use an origin from a different group, or from a different variant, as this may result in errors.|
|origins|[[CdnDeliveryV3Origin](#schemacdndeliveryv3origin)]|false|none|If `origins` is present, it will consist of one or more elements.|
|order|integer(int64)|false|none|An optional field prescribing this variant's order in relation to other variants. No guarantees other than being greater than or less than the order of other variants within this group (e.g., order may not be consecutive).|
|enabled|boolean|false|none|An optional field indicating if this variant is enabled. If this is not enabled, it may be visible to the user, but not selectable. If this field is not present, assume a default value of `false`, for safety.|
|hidden|boolean|false|none|An optional field indicating if this variant should be hidden. If hidden, it should not be shown to the user nor considered in any code logic. If this field is not present, assume a default value of `false`. Only truthy values should hide a variant.|
|meta|[CdnDeliveryV3Meta](#schemacdndeliveryv3meta)|false|none|Metadata information for this variant. Note that most/all child and grandchild properties are not required on purpose.|
|mimeType|string|false|none|An optional string describing the MIME Type of this media source.|

<h2 id="tocS_CdnDeliveryV3Meta">CdnDeliveryV3Meta</h2>
<!-- backwards compatibility -->
<a id="schemacdndeliveryv3meta"></a>
<a id="schema_CdnDeliveryV3Meta"></a>
<a id="tocScdndeliveryv3meta"></a>
<a id="tocscdndeliveryv3meta"></a>

```json
{
  "common": {
    "size": 0,
    "access": {
      "deniedReason": "isMissingPermission",
      "deniedMessage": "string"
    }
  },
  "video": {
    "codec": "string",
    "codecSimple": "string",
    "mimeType": "string",
    "width": 0,
    "height": 0,
    "isHdr": true,
    "fps": 0,
    "bitrate": {
      "maximum": 0,
      "average": 0
    }
  },
  "audio": {
    "codec": "string",
    "codecSimple": "string",
    "mimeType": "string",
    "channelCount": 0,
    "samplerate": 0,
    "bitrate": {
      "maximum": 0,
      "average": 0
    }
  },
  "image": {
    "codec": "string",
    "codecSimple": "string",
    "mimeType": "string",
    "width": 0,
    "height": 0,
    "isHdr": true
  },
  "live": {
    "lowLatencyExtension": "llhls"
  }
}

```

Metadata information for this variant. Note that most/all child and grandchild properties are not required on purpose.

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|common|object|false|none|none|
|» size|number|false|none|Size of the corresponding media file, measured in bytes.|
|» access|object|false|none|none|
|»» deniedReason|string|false|none|- `isMissingPermission`: Indicates that the requester is lacking a required plan or other form of permission entitling on to access the corresponding resource.<br>- `isProcessing`: Indicates that the corresponding resource is processing. Clients may choose to periodically refetch an asset's info when it has reported this state.<br>- `isBroken`: Indicates that the corresponding resource is defective in some manner which has rendered it currently inaccessible. It is possible that the asset will be repaired at some later point in time. Clients may choose to periodically refetch an asset's info when it has reported this state.|
|»» deniedMessage|string|false|none|Message describing in human-readable terms why access has been witheld for a resource.|
|video|any|false|none|none|

allOf

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» *anonymous*|[CdnDeliveryV3MediaIdentityCharacteristics](#schemacdndeliveryv3mediaidentitycharacteristics)|false|none|none|

and

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» *anonymous*|[CdnDeliveryV3ImagePresentationCharacteristics](#schemacdndeliveryv3imagepresentationcharacteristics)|false|none|none|

and

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» *anonymous*|object|false|none|none|
|»» fps|number|false|none|Maximum count of frames presented per second for the video.|

and

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» *anonymous*|[CdnDeliveryV3MediaBitrateInfo](#schemacdndeliveryv3mediabitrateinfo)|false|none|none|

continued

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|audio|any|false|none|none|

allOf

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» *anonymous*|[CdnDeliveryV3MediaIdentityCharacteristics](#schemacdndeliveryv3mediaidentitycharacteristics)|false|none|none|

and

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» *anonymous*|object|false|none|none|
|»» channelCount|integer|false|none|Count of channels carried by the audio stream.|
|»» samplerate|integer|false|none|Count of samples recorded per second.|

and

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» *anonymous*|[CdnDeliveryV3MediaBitrateInfo](#schemacdndeliveryv3mediabitrateinfo)|false|none|none|

continued

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|image|any|false|none|none|

allOf

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» *anonymous*|[CdnDeliveryV3MediaIdentityCharacteristics](#schemacdndeliveryv3mediaidentitycharacteristics)|false|none|none|

and

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» *anonymous*|[CdnDeliveryV3ImagePresentationCharacteristics](#schemacdndeliveryv3imagepresentationcharacteristics)|false|none|none|

continued

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|live|object|false|none|none|
|» lowLatencyExtension|string|false|none|- `llhls`: 🍎-backed low-latency HLS extension.<br>- `clhls`: Community-backed low-latency HLS extension.<br>- `ivshls`: IVS custom low-latency HLS extension.<br>- `lldash`: DASH-IF-backed low-Latency DASH extension.|

#### Enumerated Values

|Property|Value|
|---|---|
|deniedReason|isMissingPermission|
|deniedReason|isProcessing|
|deniedReason|isBroken|
|lowLatencyExtension|llhls|
|lowLatencyExtension|clhls|
|lowLatencyExtension|ivshls|
|lowLatencyExtension|lldash|

<h2 id="tocS_CdnDeliveryV3MediaIdentityCharacteristics">CdnDeliveryV3MediaIdentityCharacteristics</h2>
<!-- backwards compatibility -->
<a id="schemacdndeliveryv3mediaidentitycharacteristics"></a>
<a id="schema_CdnDeliveryV3MediaIdentityCharacteristics"></a>
<a id="tocScdndeliveryv3mediaidentitycharacteristics"></a>
<a id="tocscdndeliveryv3mediaidentitycharacteristics"></a>

```json
{
  "codec": "string",
  "codecSimple": "string",
  "mimeType": "string"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|codec|string|false|none|RFC 6381 codec string indicating stream data chunk format.|
|codecSimple|string|false|none|RFC 6381 codec string indicating stream format on the most basic level, without the addition of profile/level/etc. information.|
|mimeType|string|false|none|MIME-type for individual stream data chunks (as opposed to a containing playlist).|

<h2 id="tocS_CdnDeliveryV3ImagePresentationCharacteristics">CdnDeliveryV3ImagePresentationCharacteristics</h2>
<!-- backwards compatibility -->
<a id="schemacdndeliveryv3imagepresentationcharacteristics"></a>
<a id="schema_CdnDeliveryV3ImagePresentationCharacteristics"></a>
<a id="tocScdndeliveryv3imagepresentationcharacteristics"></a>
<a id="tocscdndeliveryv3imagepresentationcharacteristics"></a>

```json
{
  "width": 0,
  "height": 0,
  "isHdr": true
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|width|integer|false|none|Count of horizontal pixels presented.|
|height|integer|false|none|Count of vertical pixels presented.|
|isHdr|boolean|false|none|Whether or not this data stream carries HDR content.|

<h2 id="tocS_CdnDeliveryV3MediaBitrateInfo">CdnDeliveryV3MediaBitrateInfo</h2>
<!-- backwards compatibility -->
<a id="schemacdndeliveryv3mediabitrateinfo"></a>
<a id="schema_CdnDeliveryV3MediaBitrateInfo"></a>
<a id="tocScdndeliveryv3mediabitrateinfo"></a>
<a id="tocscdndeliveryv3mediabitrateinfo"></a>

```json
{
  "bitrate": {
    "maximum": 0,
    "average": 0
  }
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|bitrate|object|false|none|none|
|» maximum|number|false|none|Maximum bitrate observed for the data stream.|
|» average|number|false|none|Average bitrate observed for the data stream.|

<h2 id="tocS_PaymentInvoiceListV2Response">PaymentInvoiceListV2Response</h2>
<!-- backwards compatibility -->
<a id="schemapaymentinvoicelistv2response"></a>
<a id="schema_PaymentInvoiceListV2Response"></a>
<a id="tocSpaymentinvoicelistv2response"></a>
<a id="tocspaymentinvoicelistv2response"></a>

```json
{
  "invoices": [
    {
      "id": 0,
      "amountDue": 0,
      "amountTax": 0,
      "attemptCount": 0,
      "currency": "string",
      "date": "2019-08-24T14:15:22Z",
      "dateDue": "2019-08-24T14:15:22Z",
      "periodStart": "2019-08-24T14:15:22Z",
      "periodEnd": "2019-08-24T14:15:22Z",
      "nextPaymentAttempt": "2019-08-24T14:15:22Z",
      "paid": true,
      "forgiven": true,
      "refunded": true,
      "subscriptions": [
        {
          "id": 0,
          "subscription": 0,
          "periodStart": "2019-08-24T14:15:22Z",
          "periodEnd": "2019-08-24T14:15:22Z",
          "value": 0,
          "amountSubtotal": 0,
          "amountTotal": 0,
          "amountTax": 0,
          "plan": {
            "id": "string",
            "title": "string",
            "creator": {
              "id": "string",
              "title": "string",
              "urlname": "string",
              "icon": {
                "width": 0,
                "height": 0,
                "path": "http://example.com",
                "childImages": [
                  {
                    "width": 0,
                    "height": 0,
                    "path": "http://example.com"
                  }
                ]
              }
            }
          }
        }
      ]
    }
  ]
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|invoices|[object]|true|none|none|
|» id|integer|true|none|none|
|» amountDue|number|true|none|none|
|» amountTax|number|true|none|none|
|» attemptCount|integer|true|none|none|
|» currency|string|true|none|none|
|» date|string(date-time)|true|none|none|
|» dateDue|string(date-time)¦null|true|none|none|
|» periodStart|string(date-time)|true|none|none|
|» periodEnd|string(date-time)|true|none|none|
|» nextPaymentAttempt|string(date-time)¦null|true|none|none|
|» paid|boolean|true|none|none|
|» forgiven|boolean|true|none|none|
|» refunded|boolean|true|none|none|
|» subscriptions|[object]¦null|true|none|The subscriptions this invoice is in reference to.|
|»» id|integer|true|none|none|
|»» subscription|number|true|none|none|
|»» periodStart|string(date-time)¦null|true|none|none|
|»» periodEnd|string(date-time)¦null|true|none|none|
|»» value|number|true|none|none|
|»» amountSubtotal|number|true|none|none|
|»» amountTotal|number|true|none|none|
|»» amountTax|number|true|none|none|
|»» plan|object|true|none|none|
|»»» id|string|true|none|none|
|»»» title|string|true|none|none|
|»»» creator|object|true|none|none|
|»»»» id|string|true|none|none|
|»»»» title|string|true|none|none|
|»»»» urlname|string|true|none|none|
|»»»» icon|[ImageModel](#schemaimagemodel)|true|none|none|

<h2 id="tocS_PlanInfoV2Response">PlanInfoV2Response</h2>
<!-- backwards compatibility -->
<a id="schemaplaninfov2response"></a>
<a id="schema_PlanInfoV2Response"></a>
<a id="tocSplaninfov2response"></a>
<a id="tocsplaninfov2response"></a>

```json
{
  "totalSubscriberCount": 0,
  "totalIncome": 0,
  "plans": [
    {
      "id": "string",
      "title": "string",
      "description": "string",
      "price": "string",
      "priceYearly": "string",
      "currency": "string",
      "logo": "string",
      "interval": "string",
      "featured": true,
      "allowGrandfatheredAccess": true,
      "discordServers": [
        {
          "id": "string",
          "guildName": "string",
          "guildIcon": "string",
          "inviteLink": "http://example.com",
          "inviteMode": "string"
        }
      ],
      "discordRoles": [
        {
          "server": "string",
          "roleName": "string"
        }
      ],
      "createdAt": "2019-08-24T14:15:22Z",
      "updatedAt": "2019-08-24T14:15:22Z",
      "enabled": true,
      "paymentID": 0,
      "trialPeriod": 0,
      "creator": "string",
      "userIsSubscribed": true,
      "userIsGrandfathered": true,
      "enabledGlobal": true
    }
  ]
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|totalSubscriberCount|integer¦null|true|none|The total number of subscribers for this creator.|
|totalIncome|number¦null|true|none|The total amount of monthly income for this creator. This field tends to always be $0 for regular users.|
|plans|[allOf]|true|none|none|

allOf

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» *anonymous*|[SubscriptionPlanModel](#schemasubscriptionplanmodel)|false|none|none|

and

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» *anonymous*|object|false|none|none|
|»» createdAt|string(date-time)|true|none|none|
|»» updatedAt|string(date-time)¦null|true|none|none|
|»» enabled|boolean|true|none|none|
|»» paymentID|integer¦null|true|none|none|
|»» trialPeriod|number|true|none|none|
|»» creator|string|true|none|none|
|»» userIsSubscribed|boolean|true|none|none|
|»» userIsGrandfathered|boolean|false|none|none|
|»» enabledGlobal|boolean|true|none|none|

<h2 id="tocS_UserInfoV2Response">UserInfoV2Response</h2>
<!-- backwards compatibility -->
<a id="schemauserinfov2response"></a>
<a id="schema_UserInfoV2Response"></a>
<a id="tocSuserinfov2response"></a>
<a id="tocsuserinfov2response"></a>

```json
{
  "users": [
    {
      "id": "string",
      "user": {
        "id": "string",
        "username": "string",
        "profileImage": {
          "width": 0,
          "height": 0,
          "path": "http://example.com",
          "childImages": [
            {
              "width": 0,
              "height": 0,
              "path": "http://example.com"
            }
          ]
        }
      }
    }
  ]
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|users|[object]|true|none|none|
|» id|string|true|none|none|
|» user|any|true|none|none|

oneOf

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|»» *anonymous*|[UserModel](#schemausermodel)|false|none|Represents some basic information of a user (id, username, and profile image).|

xor

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|»» *anonymous*|[UserSelfModel](#schemauserselfmodel)|false|none|none|

<h2 id="tocS_UserNamedV2Response">UserNamedV2Response</h2>
<!-- backwards compatibility -->
<a id="schemausernamedv2response"></a>
<a id="schema_UserNamedV2Response"></a>
<a id="tocSusernamedv2response"></a>
<a id="tocsusernamedv2response"></a>

```json
{
  "users": [
    {
      "id": "string",
      "user": {
        "id": "string",
        "username": "string",
        "profileImage": {
          "width": 0,
          "height": 0,
          "path": "http://example.com",
          "childImages": [
            {
              "width": 0,
              "height": 0,
              "path": "http://example.com"
            }
          ]
        }
      }
    }
  ]
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|users|[object]|true|none|none|
|» id|string|true|none|none|
|» user|any|true|none|none|

oneOf

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|»» *anonymous*|[UserModel](#schemausermodel)|false|none|Represents some basic information of a user (id, username, and profile image).|

xor

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|»» *anonymous*|[UserSelfModel](#schemauserselfmodel)|false|none|none|

<h2 id="tocS_UserSecurityV2Response">UserSecurityV2Response</h2>
<!-- backwards compatibility -->
<a id="schemausersecurityv2response"></a>
<a id="schema_UserSecurityV2Response"></a>
<a id="tocSusersecurityv2response"></a>
<a id="tocsusersecurityv2response"></a>

```json
{
  "twofactorEnabled": true,
  "twofactorBackupCodeEnabled": true
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|twofactorEnabled|boolean|true|none|none|
|twofactorBackupCodeEnabled|boolean|true|none|none|

<h2 id="tocS_CommentV3PostRequest">CommentV3PostRequest</h2>
<!-- backwards compatibility -->
<a id="schemacommentv3postrequest"></a>
<a id="schema_CommentV3PostRequest"></a>
<a id="tocScommentv3postrequest"></a>
<a id="tocscommentv3postrequest"></a>

```json
{
  "blogPost": "string",
  "text": "string"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|blogPost|string|true|none|The GUID of the blogPost the comment should be posted to.|
|text|string|true|none|The text of the comment being posted.|

<h2 id="tocS_CommentV3PostResponse">CommentV3PostResponse</h2>
<!-- backwards compatibility -->
<a id="schemacommentv3postresponse"></a>
<a id="schema_CommentV3PostResponse"></a>
<a id="tocScommentv3postresponse"></a>
<a id="tocscommentv3postresponse"></a>

```json
{
  "id": "string",
  "blogPost": "string",
  "user": {
    "id": "string",
    "username": "string",
    "profileImage": {
      "width": 0,
      "height": 0,
      "path": "http://example.com",
      "childImages": [
        {
          "width": 0,
          "height": 0,
          "path": "http://example.com"
        }
      ]
    }
  },
  "text": "string",
  "replying": "string",
  "postDate": "string",
  "editDate": "string",
  "editCount": 0,
  "isEdited": true,
  "likes": 0,
  "dislikes": 0,
  "score": 0,
  "interactionCounts": {
    "like": 0,
    "dislike": 0
  }
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|string|true|none|none|
|blogPost|string|true|none|none|
|user|[UserModel](#schemausermodel)|true|none|Represents some basic information of a user (id, username, and profile image).|
|text|string|true|none|none|
|replying|string|true|none|none|
|postDate|string|true|none|none|
|editDate|string|true|none|none|
|editCount|integer|true|none|none|
|isEdited|boolean|true|none|none|
|likes|integer|true|none|none|
|dislikes|integer|true|none|none|
|score|integer|true|none|none|
|interactionCounts|object|true|none|none|
|» like|integer|true|none|none|
|» dislike|integer|true|none|none|

<h2 id="tocS_CommentLikeV3PostRequest">CommentLikeV3PostRequest</h2>
<!-- backwards compatibility -->
<a id="schemacommentlikev3postrequest"></a>
<a id="schema_CommentLikeV3PostRequest"></a>
<a id="tocScommentlikev3postrequest"></a>
<a id="tocscommentlikev3postrequest"></a>

```json
{
  "comment": "string",
  "blogPost": "string"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|comment|string|true|none|The GUID of the comment being liked.|
|blogPost|string|true|none|The GUID of the post the comment is on.|

<h2 id="tocS_ContentCreatorListV3Response">ContentCreatorListV3Response</h2>
<!-- backwards compatibility -->
<a id="schemacontentcreatorlistv3response"></a>
<a id="schema_ContentCreatorListV3Response"></a>
<a id="tocScontentcreatorlistv3response"></a>
<a id="tocscontentcreatorlistv3response"></a>

```json
{
  "blogPosts": [
    {
      "id": "string",
      "guid": "string",
      "title": "string",
      "text": "string",
      "type": "blogPost",
      "channel": {
        "id": "string",
        "creator": "string",
        "title": "string",
        "urlname": "string",
        "about": "string",
        "order": 0,
        "cover": {
          "width": 0,
          "height": 0,
          "path": "http://example.com",
          "childImages": [
            {
              "width": 0,
              "height": 0,
              "path": "http://example.com"
            }
          ]
        },
        "card": {
          "width": 0,
          "height": 0,
          "path": "http://example.com",
          "childImages": [
            {
              "width": 0,
              "height": 0,
              "path": "http://example.com"
            }
          ]
        },
        "icon": {
          "width": 0,
          "height": 0,
          "path": "http://example.com",
          "childImages": [
            {
              "width": 0,
              "height": 0,
              "path": "http://example.com"
            }
          ]
        },
        "socialLinks": {
          "property1": "http://example.com",
          "property2": "http://example.com"
        }
      },
      "tags": [
        "string"
      ],
      "attachmentOrder": [
        "string"
      ],
      "metadata": {
        "hasVideo": true,
        "videoCount": 0,
        "videoDuration": 0,
        "hasAudio": true,
        "audioCount": 0,
        "audioDuration": 0,
        "hasPicture": true,
        "pictureCount": 0,
        "hasGallery": true,
        "galleryCount": 0,
        "isFeatured": true
      },
      "releaseDate": "2019-08-24T14:15:22Z",
      "likes": 0,
      "dislikes": 0,
      "score": 0,
      "comments": 0,
      "creator": {
        "id": "string",
        "owner": {
          "id": "string",
          "username": "string"
        },
        "title": "string",
        "urlname": "string",
        "description": "string",
        "about": "string",
        "category": {
          "id": "string",
          "title": "string"
        },
        "cover": {
          "width": 0,
          "height": 0,
          "path": "http://example.com",
          "childImages": [
            {
              "width": 0,
              "height": 0,
              "path": "http://example.com"
            }
          ]
        },
        "icon": {
          "width": 0,
          "height": 0,
          "path": "http://example.com",
          "childImages": [
            {
              "width": 0,
              "height": 0,
              "path": "http://example.com"
            }
          ]
        },
        "liveStream": {
          "id": "string",
          "title": "string",
          "description": "string",
          "thumbnail": {
            "width": 0,
            "height": 0,
            "path": "http://example.com",
            "childImages": [
              {
                "width": 0,
                "height": 0,
                "path": "http://example.com"
              }
            ]
          },
          "owner": "string",
          "channel": "string",
          "streamPath": "string",
          "offline": {
            "title": "string",
            "description": "string",
            "thumbnail": {
              "width": 0,
              "height": 0,
              "path": "http://example.com",
              "childImages": [
                {
                  "width": 0,
                  "height": 0,
                  "path": "http://example.com"
                }
              ]
            }
          }
        },
        "subscriptionPlans": [
          {
            "id": "string",
            "title": "string",
            "description": "string",
            "price": "string",
            "priceYearly": "string",
            "currency": "string",
            "logo": "string",
            "interval": "string",
            "featured": true,
            "allowGrandfatheredAccess": true,
            "discordServers": [
              {
                "id": "string",
                "guildName": "string",
                "guildIcon": "string",
                "inviteLink": "http://example.com",
                "inviteMode": "string"
              }
            ],
            "discordRoles": [
              {
                "server": "string",
                "roleName": "string"
              }
            ]
          }
        ],
        "discoverable": true,
        "subscriberCountDisplay": "string",
        "incomeDisplay": true,
        "defaultChannel": "string",
        "channels": [
          "string"
        ],
        "card": {
          "width": 0,
          "height": 0,
          "path": "http://example.com",
          "childImages": [
            {
              "width": 0,
              "height": 0,
              "path": "http://example.com"
            }
          ]
        }
      },
      "wasReleasedSilently": true,
      "thumbnail": {
        "width": 0,
        "height": 0,
        "path": "http://example.com",
        "childImages": [
          {
            "width": 0,
            "height": 0,
            "path": "http://example.com"
          }
        ]
      },
      "isAccessible": true,
      "videoAttachments": [
        "string"
      ],
      "audioAttachments": [
        "string"
      ],
      "pictureAttachments": [
        "string"
      ],
      "galleryAttachments": [
        "string"
      ]
    }
  ],
  "lastElements": [
    {
      "creatorId": "string",
      "blogPostId": "string",
      "moreFetchable": true
    }
  ]
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|blogPosts|[[BlogPostModelV3](#schemablogpostmodelv3)]|true|none|none|
|lastElements|[[ContentCreatorListLastItems](#schemacontentcreatorlistlastitems)]|true|none|Information about paging: what the last ID retrieve is and if more posts can be retrieved afterward for subsequent requests.|

<h2 id="tocS_ContentCreatorListLastItems">ContentCreatorListLastItems</h2>
<!-- backwards compatibility -->
<a id="schemacontentcreatorlistlastitems"></a>
<a id="schema_ContentCreatorListLastItems"></a>
<a id="tocScontentcreatorlistlastitems"></a>
<a id="tocscontentcreatorlistlastitems"></a>

```json
{
  "creatorId": "string",
  "blogPostId": "string",
  "moreFetchable": true
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|creatorId|string|true|none|none|
|blogPostId|string¦null|true|none|This may be returned as `null` if no blog posts for this creator appeared yet on this page of blog posts. However, Floatplane will complain if this is sent with a `null` value.|
|moreFetchable|boolean|true|none|none|

<h2 id="tocS_ContentPostV3Response">ContentPostV3Response</h2>
<!-- backwards compatibility -->
<a id="schemacontentpostv3response"></a>
<a id="schema_ContentPostV3Response"></a>
<a id="tocScontentpostv3response"></a>
<a id="tocscontentpostv3response"></a>

```json
{
  "id": "string",
  "guid": "string",
  "title": "string",
  "text": "string",
  "type": "blogPost",
  "channel": {
    "id": "string",
    "creator": "string",
    "title": "string",
    "urlname": "string",
    "about": "string",
    "order": 0,
    "cover": {
      "width": 0,
      "height": 0,
      "path": "http://example.com",
      "childImages": [
        {
          "width": 0,
          "height": 0,
          "path": "http://example.com"
        }
      ]
    },
    "card": {
      "width": 0,
      "height": 0,
      "path": "http://example.com",
      "childImages": [
        {
          "width": 0,
          "height": 0,
          "path": "http://example.com"
        }
      ]
    },
    "icon": {
      "width": 0,
      "height": 0,
      "path": "http://example.com",
      "childImages": [
        {
          "width": 0,
          "height": 0,
          "path": "http://example.com"
        }
      ]
    },
    "socialLinks": {
      "property1": "http://example.com",
      "property2": "http://example.com"
    }
  },
  "tags": [
    "string"
  ],
  "attachmentOrder": [
    "string"
  ],
  "metadata": {
    "hasVideo": true,
    "videoCount": 0,
    "videoDuration": 0,
    "hasAudio": true,
    "audioCount": 0,
    "audioDuration": 0,
    "hasPicture": true,
    "pictureCount": 0,
    "hasGallery": true,
    "galleryCount": 0,
    "isFeatured": true
  },
  "releaseDate": "2019-08-24T14:15:22Z",
  "likes": 0,
  "dislikes": 0,
  "score": 0,
  "comments": 0,
  "creator": {
    "id": "string",
    "owner": "string",
    "title": "string",
    "urlname": "string",
    "description": "string",
    "about": "string",
    "category": "string",
    "cover": {
      "width": 0,
      "height": 0,
      "path": "http://example.com",
      "childImages": [
        {
          "width": 0,
          "height": 0,
          "path": "http://example.com"
        }
      ]
    },
    "icon": {
      "width": 0,
      "height": 0,
      "path": "http://example.com",
      "childImages": [
        {
          "width": 0,
          "height": 0,
          "path": "http://example.com"
        }
      ]
    },
    "liveStream": {
      "id": "string",
      "title": "string",
      "description": "string",
      "thumbnail": {
        "width": 0,
        "height": 0,
        "path": "http://example.com",
        "childImages": [
          {
            "width": 0,
            "height": 0,
            "path": "http://example.com"
          }
        ]
      },
      "owner": "string",
      "channel": "string",
      "streamPath": "string",
      "offline": {
        "title": "string",
        "description": "string",
        "thumbnail": {
          "width": 0,
          "height": 0,
          "path": "http://example.com",
          "childImages": [
            {
              "width": 0,
              "height": 0,
              "path": "http://example.com"
            }
          ]
        }
      }
    },
    "subscriptionPlans": [
      {
        "id": "string",
        "title": "string",
        "description": "string",
        "price": "string",
        "priceYearly": "string",
        "currency": "string",
        "logo": "string",
        "interval": "string",
        "featured": true,
        "allowGrandfatheredAccess": true,
        "discordServers": [
          {
            "id": "string",
            "guildName": "string",
            "guildIcon": "string",
            "inviteLink": "http://example.com",
            "inviteMode": "string"
          }
        ],
        "discordRoles": [
          {
            "server": "string",
            "roleName": "string"
          }
        ]
      }
    ],
    "discoverable": true,
    "subscriberCountDisplay": "string",
    "incomeDisplay": true,
    "defaultChannel": "string"
  },
  "wasReleasedSilently": true,
  "thumbnail": {
    "width": 0,
    "height": 0,
    "path": "http://example.com",
    "childImages": [
      {
        "width": 0,
        "height": 0,
        "path": "http://example.com"
      }
    ]
  },
  "isAccessible": true,
  "userInteraction": [
    "like"
  ],
  "videoAttachments": [
    {
      "id": "string",
      "guid": "string",
      "title": "string",
      "type": "string",
      "description": "string",
      "releaseDate": "2019-08-24T14:15:22Z",
      "duration": 0,
      "creator": "string",
      "likes": 0,
      "dislikes": 0,
      "score": 0,
      "isProcessing": true,
      "primaryBlogPost": "string",
      "thumbnail": {
        "width": 0,
        "height": 0,
        "path": "http://example.com",
        "childImages": [
          {
            "width": 0,
            "height": 0,
            "path": "http://example.com"
          }
        ]
      },
      "isAccessible": true
    }
  ],
  "audioAttachments": [
    {
      "id": "string",
      "guid": "string",
      "title": "string",
      "type": "string",
      "description": "string",
      "duration": 0,
      "waveform": {
        "dataSetLength": 0,
        "highestValue": 0,
        "lowestValue": 0,
        "data": [
          0
        ]
      },
      "creator": "string",
      "likes": 0,
      "dislikes": 0,
      "score": 0,
      "isProcessing": true,
      "primaryBlogPost": "string",
      "isAccessible": true
    }
  ],
  "pictureAttachments": [
    {
      "id": "string",
      "guid": "string",
      "title": "string",
      "type": "string",
      "description": "string",
      "likes": 0,
      "dislikes": 0,
      "score": 0,
      "isProcessing": true,
      "creator": "string",
      "primaryBlogPost": "string",
      "thumbnail": {
        "width": 0,
        "height": 0,
        "path": "http://example.com",
        "childImages": [
          {
            "width": 0,
            "height": 0,
            "path": "http://example.com"
          }
        ]
      },
      "isAccessible": true
    }
  ],
  "galleryAttachments": [
    null
  ]
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|string|true|none|none|
|guid|string|true|none|none|
|title|string|true|none|none|
|text|string|true|none|Text description of the post. May have HTML paragraph (`<p>`) tags surrounding it, along with other HTML.|
|type|string|true|none|none|
|channel|[ChannelModel](#schemachannelmodel)|true|none|none|
|tags|[string]|true|none|none|
|attachmentOrder|[string]|true|none|none|
|metadata|[PostMetadataModel](#schemapostmetadatamodel)|true|none|none|
|releaseDate|string(date-time)|true|none|none|
|likes|integer|true|none|none|
|dislikes|integer|true|none|none|
|score|integer|true|none|none|
|comments|integer|true|none|none|
|creator|[CreatorModelV2](#schemacreatormodelv2)|true|none|none|
|wasReleasedSilently|boolean|true|none|none|
|thumbnail|[ImageModel](#schemaimagemodel)¦null|false|none|none|
|isAccessible|boolean|true|none|If false, the post should be marked as locked and not viewable by the user.|
|userInteraction|[UserInteractionModel](#schemauserinteractionmodel)|true|none|none|
|videoAttachments|[[VideoAttachmentModel](#schemavideoattachmentmodel)]|false|none|May be undefined when the post is locked.|
|audioAttachments|[[AudioAttachmentModel](#schemaaudioattachmentmodel)]|false|none|May be undefined when the post is locked.|
|pictureAttachments|[[PictureAttachmentModel](#schemapictureattachmentmodel)]|false|none|May be undefined when the post is locked.|
|galleryAttachments|[any]|false|none|May be undefined when the post is locked.|

#### Enumerated Values

|Property|Value|
|---|---|
|type|blogPost|

<h2 id="tocS_ContentVideoV3Response">ContentVideoV3Response</h2>
<!-- backwards compatibility -->
<a id="schemacontentvideov3response"></a>
<a id="schema_ContentVideoV3Response"></a>
<a id="tocScontentvideov3response"></a>
<a id="tocscontentvideov3response"></a>

```json
{
  "id": "string",
  "guid": "string",
  "title": "string",
  "type": "string",
  "description": "string",
  "releaseDate": "2019-08-24T14:15:22Z",
  "duration": 0,
  "creator": "string",
  "likes": 0,
  "dislikes": 0,
  "score": 0,
  "isProcessing": true,
  "primaryBlogPost": "string",
  "thumbnail": {
    "width": 0,
    "height": 0,
    "path": "http://example.com",
    "childImages": [
      {
        "width": 0,
        "height": 0,
        "path": "http://example.com"
      }
    ]
  },
  "isAccessible": true,
  "blogPosts": [
    "string"
  ],
  "timelineSprite": {
    "width": 0,
    "height": 0,
    "path": "http://example.com",
    "childImages": [
      {
        "width": 0,
        "height": 0,
        "path": "http://example.com"
      }
    ]
  },
  "progress": 0,
  "userInteraction": [
    "like"
  ],
  "levels": [
    {
      "name": "string",
      "width": 0,
      "height": 0,
      "label": "string",
      "order": 0
    }
  ],
  "textTracks": [
    {
      "id": "string",
      "src": "string",
      "kind": "string",
      "language": "string",
      "generated": true,
      "processing": true
    }
  ]
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|string|true|none|none|
|guid|string|true|none|none|
|title|string|true|none|none|
|type|string|true|none|none|
|description|string|true|none|none|
|releaseDate|string(date-time)¦null|true|none|none|
|duration|number|true|none|Unit: seconds.|
|creator|string|true|none|none|
|likes|integer|true|none|none|
|dislikes|integer|true|none|none|
|score|integer|true|none|none|
|isProcessing|boolean|true|none|none|
|primaryBlogPost|string|true|none|none|
|thumbnail|[ImageModel](#schemaimagemodel)|true|none|none|
|isAccessible|boolean|true|none|If false, the post should be marked as locked and not viewable by the user.|
|blogPosts|[string]|true|none|none|
|timelineSprite|[ImageModel](#schemaimagemodel)|true|none|none|
|progress|integer|false|none|The watch progress of the video, in seconds. If no progress has yet been posted to the video, then this field may not appear.|
|userInteraction|[UserInteractionModel](#schemauserinteractionmodel)|true|none|none|
|levels|[object]|true|none|none|
|» name|string|true|none|none|
|» width|integer|true|none|none|
|» height|integer|true|none|none|
|» label|string|true|none|none|
|» order|integer|true|none|none|
|textTracks|[object]|false|none|none|
|» id|string|true|none|Text track identifier.|
|» src|string(url)|true|none|URL to a text track file, usually .vtt files.|
|» kind|string|true|none|Usually 'caption'.|
|» language|string|true|none|Language code, e.g. 'en'.|
|» generated|boolean|true|none|Whether the text track has been generated.|
|» processing|boolean|true|none|Whether the text track is processing.|

<h2 id="tocS_ContentPictureV3Response">ContentPictureV3Response</h2>
<!-- backwards compatibility -->
<a id="schemacontentpicturev3response"></a>
<a id="schema_ContentPictureV3Response"></a>
<a id="tocScontentpicturev3response"></a>
<a id="tocscontentpicturev3response"></a>

```json
{
  "id": "string",
  "guid": "string",
  "title": "string",
  "type": "string",
  "description": "string",
  "likes": 0,
  "dislikes": 0,
  "score": 0,
  "isProcessing": true,
  "creator": "string",
  "primaryBlogPost": "string",
  "userInteraction": [
    "like"
  ],
  "thumbnail": {
    "width": 0,
    "height": 0,
    "path": "http://example.com",
    "childImages": [
      {
        "width": 0,
        "height": 0,
        "path": "http://example.com"
      }
    ]
  },
  "isAccessible": true,
  "imageFiles": [
    {
      "path": "http://example.com",
      "width": 0,
      "height": 0,
      "size": 0
    }
  ]
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|string|true|none|none|
|guid|string|true|none|none|
|title|string|true|none|none|
|type|string|true|none|none|
|description|string|true|none|none|
|likes|integer|true|none|none|
|dislikes|integer|true|none|none|
|score|integer|true|none|none|
|isProcessing|boolean|true|none|none|
|creator|string|true|none|none|
|primaryBlogPost|string|true|none|none|
|userInteraction|[UserInteractionModel](#schemauserinteractionmodel)|true|none|none|
|thumbnail|[ImageModel](#schemaimagemodel)|true|none|none|
|isAccessible|boolean|true|none|If false, the post should be marked as locked and not viewable by the user.|
|imageFiles|[[ImageFileModel](#schemaimagefilemodel)]|true|none|none|

<h2 id="tocS_UserActivityV3Response">UserActivityV3Response</h2>
<!-- backwards compatibility -->
<a id="schemauseractivityv3response"></a>
<a id="schema_UserActivityV3Response"></a>
<a id="tocSuseractivityv3response"></a>
<a id="tocsuseractivityv3response"></a>

```json
{
  "activity": [
    {
      "time": "2019-08-24T14:15:22Z",
      "comment": "string",
      "postTitle": "string",
      "postId": "string",
      "creatorTitle": "string",
      "creatorUrl": "string"
    }
  ],
  "visibility": "string"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|activity|[object]|true|none|none|
|» time|string(date-time)|true|none|none|
|» comment|string|true|none|none|
|» postTitle|string|true|none|none|
|» postId|string|true|none|none|
|» creatorTitle|string|true|none|none|
|» creatorUrl|string|true|none|none|
|visibility|string|true|none|none|

<h2 id="tocS_UserLinksV3Response">UserLinksV3Response</h2>
<!-- backwards compatibility -->
<a id="schemauserlinksv3response"></a>
<a id="schema_UserLinksV3Response"></a>
<a id="tocSuserlinksv3response"></a>
<a id="tocsuserlinksv3response"></a>

```json
{
  "property1": {
    "url": "http://example.com",
    "type": {
      "name": "string",
      "displayName": "string",
      "hostName": "string"
    }
  },
  "property2": {
    "url": "http://example.com",
    "type": {
      "name": "string",
      "displayName": "string",
      "hostName": "string"
    }
  }
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|**additionalProperties**|object|false|none|none|
|» url|string(uri)|true|none|The URL the user has configured for this link.|
|» type|object|true|none|none|
|»» name|string|true|none|The code name of this link type.|
|»» displayName|string|true|none|The display-friendly name of this link type.|
|»» hostName|string|true|none|The hostname that should be a part of the URL.|

<h2 id="tocS_UserNotificationUpdateV3PostRequest">UserNotificationUpdateV3PostRequest</h2>
<!-- backwards compatibility -->
<a id="schemausernotificationupdatev3postrequest"></a>
<a id="schema_UserNotificationUpdateV3PostRequest"></a>
<a id="tocSusernotificationupdatev3postrequest"></a>
<a id="tocsusernotificationupdatev3postrequest"></a>

```json
{
  "creator": "string",
  "property": "contentEmail",
  "newValue": true
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|creator|string|true|none|none|
|property|string|true|none|Use `contentEmail` for email notifications, and `contentFirebase` for push notifications.|
|newValue|boolean|true|none|none|

#### Enumerated Values

|Property|Value|
|---|---|
|property|contentEmail|
|property|contentFirebase|

<h2 id="tocS_UserSelfV3Response">UserSelfV3Response</h2>
<!-- backwards compatibility -->
<a id="schemauserselfv3response"></a>
<a id="schema_UserSelfV3Response"></a>
<a id="tocSuserselfv3response"></a>
<a id="tocsuserselfv3response"></a>

```json
{
  "id": "string",
  "username": "string",
  "profileImage": {
    "width": 0,
    "height": 0,
    "path": "http://example.com",
    "childImages": [
      {
        "width": 0,
        "height": 0,
        "path": "http://example.com"
      }
    ]
  },
  "email": "string",
  "displayName": "string",
  "creators": [
    null
  ],
  "scheduledDeletionDate": "2019-08-24T14:15:22Z"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|string|true|none|none|
|username|string|true|none|none|
|profileImage|[ImageModel](#schemaimagemodel)|true|none|none|
|email|string|true|none|none|
|displayName|string|true|none|none|
|creators|[any]|true|none|none|
|scheduledDeletionDate|string(date-time)¦null|true|none|none|

<h2 id="tocS_ContentLikeV3Request">ContentLikeV3Request</h2>
<!-- backwards compatibility -->
<a id="schemacontentlikev3request"></a>
<a id="schema_ContentLikeV3Request"></a>
<a id="tocScontentlikev3request"></a>
<a id="tocscontentlikev3request"></a>

```json
{
  "contentType": "blogPost",
  "id": "string"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|contentType|string|true|none|none|
|id|string|true|none|none|

#### Enumerated Values

|Property|Value|
|---|---|
|contentType|blogPost|

<h2 id="tocS_GetCaptchaInfoResponse">GetCaptchaInfoResponse</h2>
<!-- backwards compatibility -->
<a id="schemagetcaptchainforesponse"></a>
<a id="schema_GetCaptchaInfoResponse"></a>
<a id="tocSgetcaptchainforesponse"></a>
<a id="tocsgetcaptchainforesponse"></a>

```json
{
  "v2": {
    "variants": {
      "android": {
        "siteKey": "string"
      },
      "checkbox": {
        "siteKey": "string"
      },
      "invisible": {
        "siteKey": "string"
      }
    }
  },
  "v3": {
    "variants": {
      "invisible": {
        "siteKey": "string"
      }
    }
  }
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|v2|object|true|none|none|
|» variants|object|true|none|none|
|»» android|object|true|none|none|
|»»» siteKey|string|true|none|none|
|»» checkbox|object|true|none|none|
|»»» siteKey|string|true|none|none|
|»» invisible|object|true|none|none|
|»»» siteKey|string|true|none|none|
|v3|object|true|none|none|
|» variants|object|true|none|none|
|»» invisible|object|true|none|none|
|»»» siteKey|string|true|none|none|

<h2 id="tocS_ErrorModel">ErrorModel</h2>
<!-- backwards compatibility -->
<a id="schemaerrormodel"></a>
<a id="schema_ErrorModel"></a>
<a id="tocSerrormodel"></a>
<a id="tocserrormodel"></a>

```json
{
  "id": "string",
  "errors": [
    {
      "id": "string",
      "name": "string",
      "message": "string",
      "data": {}
    }
  ],
  "message": "string"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|string|true|none|none|
|errors|[object]|true|none|none|
|» id|string|true|none|none|
|» name|string|true|none|none|
|» message|string¦null|false|none|May be undefined.|
|» data|object¦null|false|none|May be undefined.|
|message|string|false|none|May be undefined.|

<h2 id="tocS_PaymentAddressModel">PaymentAddressModel</h2>
<!-- backwards compatibility -->
<a id="schemapaymentaddressmodel"></a>
<a id="schema_PaymentAddressModel"></a>
<a id="tocSpaymentaddressmodel"></a>
<a id="tocspaymentaddressmodel"></a>

```json
{
  "id": 0,
  "customerName": "string",
  "postalCode": "string",
  "line1": "string",
  "city": "string",
  "region": "string",
  "country": "string",
  "default": true
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|none|
|customerName|string|true|none|none|
|postalCode|string|true|none|none|
|line1|string|true|none|none|
|city|string|true|none|none|
|region|string|true|none|none|
|country|string|true|none|none|
|default|boolean|true|none|none|

<h2 id="tocS_PaymentMethodModel">PaymentMethodModel</h2>
<!-- backwards compatibility -->
<a id="schemapaymentmethodmodel"></a>
<a id="schema_PaymentMethodModel"></a>
<a id="tocSpaymentmethodmodel"></a>
<a id="tocspaymentmethodmodel"></a>

```json
{
  "id": 0,
  "payment_processor": 0,
  "default": true,
  "card": {
    "brand": "string",
    "last4": "string",
    "exp_month": 0,
    "exp_year": 0,
    "name": "string"
  }
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|integer|true|none|none|
|payment_processor|integer|true|none|none|
|default|boolean|true|none|none|
|card|object|true|none|none|
|» brand|string|true|none|none|
|» last4|string|true|none|none|
|» exp_month|integer|true|none|none|
|» exp_year|integer|true|none|none|
|» name|string|true|none|none|

<h2 id="tocS_CreatorModelV2">CreatorModelV2</h2>
<!-- backwards compatibility -->
<a id="schemacreatormodelv2"></a>
<a id="schema_CreatorModelV2"></a>
<a id="tocScreatormodelv2"></a>
<a id="tocscreatormodelv2"></a>

```json
{
  "id": "string",
  "owner": "string",
  "title": "string",
  "urlname": "string",
  "description": "string",
  "about": "string",
  "category": "string",
  "cover": {
    "width": 0,
    "height": 0,
    "path": "http://example.com",
    "childImages": [
      {
        "width": 0,
        "height": 0,
        "path": "http://example.com"
      }
    ]
  },
  "icon": {
    "width": 0,
    "height": 0,
    "path": "http://example.com",
    "childImages": [
      {
        "width": 0,
        "height": 0,
        "path": "http://example.com"
      }
    ]
  },
  "liveStream": {
    "id": "string",
    "title": "string",
    "description": "string",
    "thumbnail": {
      "width": 0,
      "height": 0,
      "path": "http://example.com",
      "childImages": [
        {
          "width": 0,
          "height": 0,
          "path": "http://example.com"
        }
      ]
    },
    "owner": "string",
    "channel": "string",
    "streamPath": "string",
    "offline": {
      "title": "string",
      "description": "string",
      "thumbnail": {
        "width": 0,
        "height": 0,
        "path": "http://example.com",
        "childImages": [
          {
            "width": 0,
            "height": 0,
            "path": "http://example.com"
          }
        ]
      }
    }
  },
  "subscriptionPlans": [
    {
      "id": "string",
      "title": "string",
      "description": "string",
      "price": "string",
      "priceYearly": "string",
      "currency": "string",
      "logo": "string",
      "interval": "string",
      "featured": true,
      "allowGrandfatheredAccess": true,
      "discordServers": [
        {
          "id": "string",
          "guildName": "string",
          "guildIcon": "string",
          "inviteLink": "http://example.com",
          "inviteMode": "string"
        }
      ],
      "discordRoles": [
        {
          "server": "string",
          "roleName": "string"
        }
      ]
    }
  ],
  "discoverable": true,
  "subscriberCountDisplay": "string",
  "incomeDisplay": true,
  "defaultChannel": "string"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|string|true|none|none|
|owner|string|true|none|none|
|title|string|true|none|none|
|urlname|string|true|none|Shown in the browser URL, and used in `/creator/named` queries.|
|description|string|true|none|none|
|about|string|true|none|none|
|category|string|true|none|none|
|cover|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|icon|[ImageModel](#schemaimagemodel)|true|none|none|
|liveStream|[LiveStreamModel](#schemalivestreammodel)¦null|true|none|none|
|subscriptionPlans|[[SubscriptionPlanModel](#schemasubscriptionplanmodel)]¦null|true|none|none|
|discoverable|boolean|true|none|none|
|subscriberCountDisplay|string|true|none|none|
|incomeDisplay|boolean|true|none|none|
|defaultChannel|string|false|none|none|

<h2 id="tocS_CreatorModelV2Extended">CreatorModelV2Extended</h2>
<!-- backwards compatibility -->
<a id="schemacreatormodelv2extended"></a>
<a id="schema_CreatorModelV2Extended"></a>
<a id="tocScreatormodelv2extended"></a>
<a id="tocscreatormodelv2extended"></a>

```json
{
  "id": "string",
  "owner": "string",
  "title": "string",
  "urlname": "string",
  "description": "string",
  "about": "string",
  "category": "string",
  "cover": {
    "width": 0,
    "height": 0,
    "path": "http://example.com",
    "childImages": [
      {
        "width": 0,
        "height": 0,
        "path": "http://example.com"
      }
    ]
  },
  "icon": {
    "width": 0,
    "height": 0,
    "path": "http://example.com",
    "childImages": [
      {
        "width": 0,
        "height": 0,
        "path": "http://example.com"
      }
    ]
  },
  "liveStream": {
    "id": "string",
    "title": "string",
    "description": "string",
    "thumbnail": {
      "width": 0,
      "height": 0,
      "path": "http://example.com",
      "childImages": [
        {
          "width": 0,
          "height": 0,
          "path": "http://example.com"
        }
      ]
    },
    "owner": "string",
    "channel": "string",
    "streamPath": "string",
    "offline": {
      "title": "string",
      "description": "string",
      "thumbnail": {
        "width": 0,
        "height": 0,
        "path": "http://example.com",
        "childImages": [
          {
            "width": 0,
            "height": 0,
            "path": "http://example.com"
          }
        ]
      }
    }
  },
  "subscriptionPlans": [
    {
      "id": "string",
      "title": "string",
      "description": "string",
      "price": "string",
      "priceYearly": "string",
      "currency": "string",
      "logo": "string",
      "interval": "string",
      "featured": true,
      "allowGrandfatheredAccess": true,
      "discordServers": [
        {
          "id": "string",
          "guildName": "string",
          "guildIcon": "string",
          "inviteLink": "http://example.com",
          "inviteMode": "string"
        }
      ],
      "discordRoles": [
        {
          "server": "string",
          "roleName": "string"
        }
      ]
    }
  ],
  "discoverable": true,
  "subscriberCountDisplay": "string",
  "incomeDisplay": true,
  "defaultChannel": "string",
  "socialLinks": {
    "property1": "http://example.com",
    "property2": "http://example.com"
  },
  "discordServers": [
    {
      "id": "string",
      "guildName": "string",
      "guildIcon": "string",
      "inviteLink": "http://example.com",
      "inviteMode": "string"
    }
  ]
}

```

### Properties

allOf

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|[CreatorModelV2](#schemacreatormodelv2)|false|none|none|

and

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|*anonymous*|object|false|none|none|
|» socialLinks|[SocialLinksModel](#schemasociallinksmodel)|true|none|none|
|» discordServers|[[DiscordServerModel](#schemadiscordservermodel)]|true|none|none|

<h2 id="tocS_CreatorModelV3">CreatorModelV3</h2>
<!-- backwards compatibility -->
<a id="schemacreatormodelv3"></a>
<a id="schema_CreatorModelV3"></a>
<a id="tocScreatormodelv3"></a>
<a id="tocscreatormodelv3"></a>

```json
{
  "id": "string",
  "owner": {
    "id": "string",
    "username": "string"
  },
  "title": "string",
  "urlname": "string",
  "description": "string",
  "about": "string",
  "category": {
    "id": "string",
    "title": "string"
  },
  "cover": {
    "width": 0,
    "height": 0,
    "path": "http://example.com",
    "childImages": [
      {
        "width": 0,
        "height": 0,
        "path": "http://example.com"
      }
    ]
  },
  "icon": {
    "width": 0,
    "height": 0,
    "path": "http://example.com",
    "childImages": [
      {
        "width": 0,
        "height": 0,
        "path": "http://example.com"
      }
    ]
  },
  "liveStream": {
    "id": "string",
    "title": "string",
    "description": "string",
    "thumbnail": {
      "width": 0,
      "height": 0,
      "path": "http://example.com",
      "childImages": [
        {
          "width": 0,
          "height": 0,
          "path": "http://example.com"
        }
      ]
    },
    "owner": "string",
    "channel": "string",
    "streamPath": "string",
    "offline": {
      "title": "string",
      "description": "string",
      "thumbnail": {
        "width": 0,
        "height": 0,
        "path": "http://example.com",
        "childImages": [
          {
            "width": 0,
            "height": 0,
            "path": "http://example.com"
          }
        ]
      }
    }
  },
  "subscriptionPlans": [
    {
      "id": "string",
      "title": "string",
      "description": "string",
      "price": "string",
      "priceYearly": "string",
      "currency": "string",
      "logo": "string",
      "interval": "string",
      "featured": true,
      "allowGrandfatheredAccess": true,
      "discordServers": [
        {
          "id": "string",
          "guildName": "string",
          "guildIcon": "string",
          "inviteLink": "http://example.com",
          "inviteMode": "string"
        }
      ],
      "discordRoles": [
        {
          "server": "string",
          "roleName": "string"
        }
      ]
    }
  ],
  "discoverable": true,
  "subscriberCountDisplay": "string",
  "incomeDisplay": true,
  "defaultChannel": "string",
  "socialLinks": {
    "property1": "http://example.com",
    "property2": "http://example.com"
  },
  "channels": [
    {
      "id": "string",
      "creator": "string",
      "title": "string",
      "urlname": "string",
      "about": "string",
      "order": 0,
      "cover": {
        "width": 0,
        "height": 0,
        "path": "http://example.com",
        "childImages": [
          {
            "width": 0,
            "height": 0,
            "path": "http://example.com"
          }
        ]
      },
      "card": {
        "width": 0,
        "height": 0,
        "path": "http://example.com",
        "childImages": [
          {
            "width": 0,
            "height": 0,
            "path": "http://example.com"
          }
        ]
      },
      "icon": {
        "width": 0,
        "height": 0,
        "path": "http://example.com",
        "childImages": [
          {
            "width": 0,
            "height": 0,
            "path": "http://example.com"
          }
        ]
      },
      "socialLinks": {
        "property1": "http://example.com",
        "property2": "http://example.com"
      }
    }
  ],
  "discordServers": [
    {
      "id": "string",
      "guildName": "string",
      "guildIcon": "string",
      "inviteLink": "http://example.com",
      "inviteMode": "string"
    }
  ],
  "card": {
    "width": 0,
    "height": 0,
    "path": "http://example.com",
    "childImages": [
      {
        "width": 0,
        "height": 0,
        "path": "http://example.com"
      }
    ]
  }
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|string|true|none|none|
|owner|object|true|none|none|
|» id|string|true|none|none|
|» username|string|true|none|none|
|title|string|true|none|none|
|urlname|string|true|none|Shown in the browser URL, and used in `/creator/named` queries.|
|description|string|true|none|none|
|about|string|true|none|none|
|category|object|true|none|none|
|» id|string|true|none|none|
|» title|string|true|none|none|
|cover|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|icon|[ImageModel](#schemaimagemodel)|true|none|none|
|liveStream|[LiveStreamModel](#schemalivestreammodel)¦null|true|none|none|
|subscriptionPlans|[[SubscriptionPlanModel](#schemasubscriptionplanmodel)]¦null|true|none|none|
|discoverable|boolean|true|none|none|
|subscriberCountDisplay|string|true|none|none|
|incomeDisplay|boolean|true|none|none|
|defaultChannel|string|true|none|none|
|socialLinks|[SocialLinksModel](#schemasociallinksmodel)|true|none|none|
|channels|[[ChannelModel](#schemachannelmodel)]|true|none|none|
|discordServers|[[DiscordServerModel](#schemadiscordservermodel)]|false|none|Present in `/creator/named` queries|
|card|[ImageModel](#schemaimagemodel)|false|none|none|

<h2 id="tocS_ChannelModel">ChannelModel</h2>
<!-- backwards compatibility -->
<a id="schemachannelmodel"></a>
<a id="schema_ChannelModel"></a>
<a id="tocSchannelmodel"></a>
<a id="tocschannelmodel"></a>

```json
{
  "id": "string",
  "creator": "string",
  "title": "string",
  "urlname": "string",
  "about": "string",
  "order": 0,
  "cover": {
    "width": 0,
    "height": 0,
    "path": "http://example.com",
    "childImages": [
      {
        "width": 0,
        "height": 0,
        "path": "http://example.com"
      }
    ]
  },
  "card": {
    "width": 0,
    "height": 0,
    "path": "http://example.com",
    "childImages": [
      {
        "width": 0,
        "height": 0,
        "path": "http://example.com"
      }
    ]
  },
  "icon": {
    "width": 0,
    "height": 0,
    "path": "http://example.com",
    "childImages": [
      {
        "width": 0,
        "height": 0,
        "path": "http://example.com"
      }
    ]
  },
  "socialLinks": {
    "property1": "http://example.com",
    "property2": "http://example.com"
  }
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|string|true|none|none|
|creator|string|true|none|none|
|title|string|true|none|none|
|urlname|string|true|none|Shown in the browser URL.|
|about|string|true|none|none|
|order|integer|false|none|none|
|cover|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|card|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|icon|[ImageModel](#schemaimagemodel)|true|none|none|
|socialLinks|[SocialLinksModel](#schemasociallinksmodel)|false|none|none|

<h2 id="tocS_BlogPostModelV3">BlogPostModelV3</h2>
<!-- backwards compatibility -->
<a id="schemablogpostmodelv3"></a>
<a id="schema_BlogPostModelV3"></a>
<a id="tocSblogpostmodelv3"></a>
<a id="tocsblogpostmodelv3"></a>

```json
{
  "id": "string",
  "guid": "string",
  "title": "string",
  "text": "string",
  "type": "blogPost",
  "channel": {
    "id": "string",
    "creator": "string",
    "title": "string",
    "urlname": "string",
    "about": "string",
    "order": 0,
    "cover": {
      "width": 0,
      "height": 0,
      "path": "http://example.com",
      "childImages": [
        {
          "width": 0,
          "height": 0,
          "path": "http://example.com"
        }
      ]
    },
    "card": {
      "width": 0,
      "height": 0,
      "path": "http://example.com",
      "childImages": [
        {
          "width": 0,
          "height": 0,
          "path": "http://example.com"
        }
      ]
    },
    "icon": {
      "width": 0,
      "height": 0,
      "path": "http://example.com",
      "childImages": [
        {
          "width": 0,
          "height": 0,
          "path": "http://example.com"
        }
      ]
    },
    "socialLinks": {
      "property1": "http://example.com",
      "property2": "http://example.com"
    }
  },
  "tags": [
    "string"
  ],
  "attachmentOrder": [
    "string"
  ],
  "metadata": {
    "hasVideo": true,
    "videoCount": 0,
    "videoDuration": 0,
    "hasAudio": true,
    "audioCount": 0,
    "audioDuration": 0,
    "hasPicture": true,
    "pictureCount": 0,
    "hasGallery": true,
    "galleryCount": 0,
    "isFeatured": true
  },
  "releaseDate": "2019-08-24T14:15:22Z",
  "likes": 0,
  "dislikes": 0,
  "score": 0,
  "comments": 0,
  "creator": {
    "id": "string",
    "owner": {
      "id": "string",
      "username": "string"
    },
    "title": "string",
    "urlname": "string",
    "description": "string",
    "about": "string",
    "category": {
      "id": "string",
      "title": "string"
    },
    "cover": {
      "width": 0,
      "height": 0,
      "path": "http://example.com",
      "childImages": [
        {
          "width": 0,
          "height": 0,
          "path": "http://example.com"
        }
      ]
    },
    "icon": {
      "width": 0,
      "height": 0,
      "path": "http://example.com",
      "childImages": [
        {
          "width": 0,
          "height": 0,
          "path": "http://example.com"
        }
      ]
    },
    "liveStream": {
      "id": "string",
      "title": "string",
      "description": "string",
      "thumbnail": {
        "width": 0,
        "height": 0,
        "path": "http://example.com",
        "childImages": [
          {
            "width": 0,
            "height": 0,
            "path": "http://example.com"
          }
        ]
      },
      "owner": "string",
      "channel": "string",
      "streamPath": "string",
      "offline": {
        "title": "string",
        "description": "string",
        "thumbnail": {
          "width": 0,
          "height": 0,
          "path": "http://example.com",
          "childImages": [
            {
              "width": 0,
              "height": 0,
              "path": "http://example.com"
            }
          ]
        }
      }
    },
    "subscriptionPlans": [
      {
        "id": "string",
        "title": "string",
        "description": "string",
        "price": "string",
        "priceYearly": "string",
        "currency": "string",
        "logo": "string",
        "interval": "string",
        "featured": true,
        "allowGrandfatheredAccess": true,
        "discordServers": [
          {
            "id": "string",
            "guildName": "string",
            "guildIcon": "string",
            "inviteLink": "http://example.com",
            "inviteMode": "string"
          }
        ],
        "discordRoles": [
          {
            "server": "string",
            "roleName": "string"
          }
        ]
      }
    ],
    "discoverable": true,
    "subscriberCountDisplay": "string",
    "incomeDisplay": true,
    "defaultChannel": "string",
    "channels": [
      "string"
    ],
    "card": {
      "width": 0,
      "height": 0,
      "path": "http://example.com",
      "childImages": [
        {
          "width": 0,
          "height": 0,
          "path": "http://example.com"
        }
      ]
    }
  },
  "wasReleasedSilently": true,
  "thumbnail": {
    "width": 0,
    "height": 0,
    "path": "http://example.com",
    "childImages": [
      {
        "width": 0,
        "height": 0,
        "path": "http://example.com"
      }
    ]
  },
  "isAccessible": true,
  "videoAttachments": [
    "string"
  ],
  "audioAttachments": [
    "string"
  ],
  "pictureAttachments": [
    "string"
  ],
  "galleryAttachments": [
    "string"
  ]
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|string|true|none|none|
|guid|string|true|none|none|
|title|string|true|none|none|
|text|string|true|none|Text description of the post. May have HTML paragraph (`<p>`) tags surrounding it, along with other HTML..|
|type|string|true|none|none|
|channel|any|true|none|none|

oneOf

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» *anonymous*|[ChannelModel](#schemachannelmodel)|false|none|none|

xor

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|» *anonymous*|string|false|none|Channel identifier|

continued

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|tags|[string]|true|none|none|
|attachmentOrder|[string]|true|none|none|
|metadata|[PostMetadataModel](#schemapostmetadatamodel)|true|none|none|
|releaseDate|string(date-time)|true|none|none|
|likes|integer|true|none|none|
|dislikes|integer|true|none|none|
|score|integer|true|none|none|
|comments|integer|true|none|none|
|creator|object|true|none|none|
|» id|string|true|none|none|
|» owner|object|true|none|none|
|»» id|string|true|none|none|
|»» username|string|true|none|none|
|» title|string|true|none|none|
|» urlname|string|true|none|Shown in the browser URL, and used in `/creator/named` queries.|
|» description|string|true|none|none|
|» about|string|true|none|none|
|» category|object|true|none|none|
|»» id|string|true|none|none|
|»» title|string|true|none|none|
|» cover|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|» icon|[ImageModel](#schemaimagemodel)|true|none|none|
|» liveStream|[LiveStreamModel](#schemalivestreammodel)¦null|true|none|none|
|» subscriptionPlans|[[SubscriptionPlanModel](#schemasubscriptionplanmodel)]|true|none|none|
|» discoverable|boolean|true|none|none|
|» subscriberCountDisplay|string|true|none|none|
|» incomeDisplay|boolean|true|none|none|
|» defaultChannel|string|false|none|none|
|» channels|[string]|false|none|none|
|» card|[ImageModel](#schemaimagemodel)¦null|false|none|none|
|wasReleasedSilently|boolean|true|none|none|
|thumbnail|[ImageModel](#schemaimagemodel)¦null|false|none|none|
|isAccessible|boolean|true|none|If false, the post should be marked as locked and not viewable by the user.|
|videoAttachments|[string]|false|none|May be undefined, usually when `isAccessible` is `false`.|
|audioAttachments|[string]|false|none|May be undefined, usually when `isAccessible` is `false`.|
|pictureAttachments|[string]|false|none|May be undefined, usually when `isAccessible` is `false`.|
|galleryAttachments|[string]|false|none|May be undefined, usually when `isAccessible` is `false`.|

#### Enumerated Values

|Property|Value|
|---|---|
|type|blogPost|

<h2 id="tocS_SubscriptionPlanModel">SubscriptionPlanModel</h2>
<!-- backwards compatibility -->
<a id="schemasubscriptionplanmodel"></a>
<a id="schema_SubscriptionPlanModel"></a>
<a id="tocSsubscriptionplanmodel"></a>
<a id="tocssubscriptionplanmodel"></a>

```json
{
  "id": "string",
  "title": "string",
  "description": "string",
  "price": "string",
  "priceYearly": "string",
  "currency": "string",
  "logo": "string",
  "interval": "string",
  "featured": true,
  "allowGrandfatheredAccess": true,
  "discordServers": [
    {
      "id": "string",
      "guildName": "string",
      "guildIcon": "string",
      "inviteLink": "http://example.com",
      "inviteMode": "string"
    }
  ],
  "discordRoles": [
    {
      "server": "string",
      "roleName": "string"
    }
  ]
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|string|true|none|none|
|title|string|true|none|none|
|description|string|true|none|none|
|price|string¦null|true|none|none|
|priceYearly|string¦null|false|none|none|
|currency|string|true|none|none|
|logo|string¦null|true|none|none|
|interval|string|true|none|none|
|featured|boolean|true|none|none|
|allowGrandfatheredAccess|boolean¦null|false|none|none|
|discordServers|[[DiscordServerModel](#schemadiscordservermodel)]|true|none|none|
|discordRoles|[[DiscordRoleModel](#schemadiscordrolemodel)]|true|none|none|

<h2 id="tocS_PostMetadataModel">PostMetadataModel</h2>
<!-- backwards compatibility -->
<a id="schemapostmetadatamodel"></a>
<a id="schema_PostMetadataModel"></a>
<a id="tocSpostmetadatamodel"></a>
<a id="tocspostmetadatamodel"></a>

```json
{
  "hasVideo": true,
  "videoCount": 0,
  "videoDuration": 0,
  "hasAudio": true,
  "audioCount": 0,
  "audioDuration": 0,
  "hasPicture": true,
  "pictureCount": 0,
  "hasGallery": true,
  "galleryCount": 0,
  "isFeatured": true
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|hasVideo|boolean|true|none|none|
|videoCount|integer|false|none|none|
|videoDuration|number|true|none|none|
|hasAudio|boolean|true|none|none|
|audioCount|integer|false|none|none|
|audioDuration|number|true|none|none|
|hasPicture|boolean|true|none|none|
|pictureCount|integer|false|none|none|
|hasGallery|boolean|false|none|none|
|galleryCount|integer|false|none|none|
|isFeatured|boolean|true|none|none|

<h2 id="tocS_VideoAttachmentModel">VideoAttachmentModel</h2>
<!-- backwards compatibility -->
<a id="schemavideoattachmentmodel"></a>
<a id="schema_VideoAttachmentModel"></a>
<a id="tocSvideoattachmentmodel"></a>
<a id="tocsvideoattachmentmodel"></a>

```json
{
  "id": "string",
  "guid": "string",
  "title": "string",
  "type": "string",
  "description": "string",
  "releaseDate": "2019-08-24T14:15:22Z",
  "duration": 0,
  "creator": "string",
  "likes": 0,
  "dislikes": 0,
  "score": 0,
  "isProcessing": true,
  "primaryBlogPost": "string",
  "thumbnail": {
    "width": 0,
    "height": 0,
    "path": "http://example.com",
    "childImages": [
      {
        "width": 0,
        "height": 0,
        "path": "http://example.com"
      }
    ]
  },
  "isAccessible": true
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|string|true|none|none|
|guid|string|true|none|none|
|title|string|true|none|none|
|type|string|true|none|none|
|description|string|true|none|none|
|releaseDate|string(date-time)¦null|true|none|none|
|duration|number|true|none|none|
|creator|string|true|none|none|
|likes|integer|true|none|none|
|dislikes|integer|true|none|none|
|score|integer|true|none|none|
|isProcessing|boolean|true|none|none|
|primaryBlogPost|string|true|none|none|
|thumbnail|[ImageModel](#schemaimagemodel)|true|none|none|
|isAccessible|boolean|true|none|If false, the post should be marked as locked and not viewable by the user.|

<h2 id="tocS_PictureAttachmentModel">PictureAttachmentModel</h2>
<!-- backwards compatibility -->
<a id="schemapictureattachmentmodel"></a>
<a id="schema_PictureAttachmentModel"></a>
<a id="tocSpictureattachmentmodel"></a>
<a id="tocspictureattachmentmodel"></a>

```json
{
  "id": "string",
  "guid": "string",
  "title": "string",
  "type": "string",
  "description": "string",
  "likes": 0,
  "dislikes": 0,
  "score": 0,
  "isProcessing": true,
  "creator": "string",
  "primaryBlogPost": "string",
  "thumbnail": {
    "width": 0,
    "height": 0,
    "path": "http://example.com",
    "childImages": [
      {
        "width": 0,
        "height": 0,
        "path": "http://example.com"
      }
    ]
  },
  "isAccessible": true
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|string|true|none|none|
|guid|string|true|none|none|
|title|string|true|none|none|
|type|string|true|none|none|
|description|string|true|none|none|
|likes|integer|true|none|none|
|dislikes|integer|true|none|none|
|score|integer|true|none|none|
|isProcessing|boolean|true|none|none|
|creator|string|true|none|none|
|primaryBlogPost|string|true|none|none|
|thumbnail|[ImageModel](#schemaimagemodel)|true|none|none|
|isAccessible|boolean|true|none|If false, the post should be marked as locked and not viewable by the user.|

<h2 id="tocS_AudioAttachmentModel">AudioAttachmentModel</h2>
<!-- backwards compatibility -->
<a id="schemaaudioattachmentmodel"></a>
<a id="schema_AudioAttachmentModel"></a>
<a id="tocSaudioattachmentmodel"></a>
<a id="tocsaudioattachmentmodel"></a>

```json
{
  "id": "string",
  "guid": "string",
  "title": "string",
  "type": "string",
  "description": "string",
  "duration": 0,
  "waveform": {
    "dataSetLength": 0,
    "highestValue": 0,
    "lowestValue": 0,
    "data": [
      0
    ]
  },
  "creator": "string",
  "likes": 0,
  "dislikes": 0,
  "score": 0,
  "isProcessing": true,
  "primaryBlogPost": "string",
  "isAccessible": true
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|string|true|none|none|
|guid|string|true|none|none|
|title|string|true|none|none|
|type|string|true|none|none|
|description|string|true|none|none|
|duration|integer|true|none|none|
|waveform|object|true|none|none|
|» dataSetLength|integer|true|none|none|
|» highestValue|integer|true|none|none|
|» lowestValue|integer|true|none|none|
|» data|[integer]|true|none|none|
|creator|string|true|none|none|
|likes|integer|true|none|none|
|dislikes|integer|true|none|none|
|score|integer|true|none|none|
|isProcessing|boolean|true|none|none|
|primaryBlogPost|string|true|none|none|
|isAccessible|boolean|true|none|If false, the post should be marked as locked and not viewable by the user.|

<h2 id="tocS_ImageModel">ImageModel</h2>
<!-- backwards compatibility -->
<a id="schemaimagemodel"></a>
<a id="schema_ImageModel"></a>
<a id="tocSimagemodel"></a>
<a id="tocsimagemodel"></a>

```json
{
  "width": 0,
  "height": 0,
  "path": "http://example.com",
  "childImages": [
    {
      "width": 0,
      "height": 0,
      "path": "http://example.com"
    }
  ]
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|width|integer|true|none|none|
|height|integer|true|none|none|
|path|string(uri)|true|none|none|
|childImages|[[ChildImageModel](#schemachildimagemodel)]¦null|true|none|none|

<h2 id="tocS_ChildImageModel">ChildImageModel</h2>
<!-- backwards compatibility -->
<a id="schemachildimagemodel"></a>
<a id="schema_ChildImageModel"></a>
<a id="tocSchildimagemodel"></a>
<a id="tocschildimagemodel"></a>

```json
{
  "width": 0,
  "height": 0,
  "path": "http://example.com"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|width|integer|true|none|none|
|height|integer|true|none|none|
|path|string(uri)|true|none|none|

<h2 id="tocS_ImageFileModel">ImageFileModel</h2>
<!-- backwards compatibility -->
<a id="schemaimagefilemodel"></a>
<a id="schema_ImageFileModel"></a>
<a id="tocSimagefilemodel"></a>
<a id="tocsimagefilemodel"></a>

```json
{
  "path": "http://example.com",
  "width": 0,
  "height": 0,
  "size": 0
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|path|string(uri)|true|none|none|
|width|integer|true|none|none|
|height|integer|true|none|none|
|size|integer|true|none|none|

<h2 id="tocS_LiveStreamModel">LiveStreamModel</h2>
<!-- backwards compatibility -->
<a id="schemalivestreammodel"></a>
<a id="schema_LiveStreamModel"></a>
<a id="tocSlivestreammodel"></a>
<a id="tocslivestreammodel"></a>

```json
{
  "id": "string",
  "title": "string",
  "description": "string",
  "thumbnail": {
    "width": 0,
    "height": 0,
    "path": "http://example.com",
    "childImages": [
      {
        "width": 0,
        "height": 0,
        "path": "http://example.com"
      }
    ]
  },
  "owner": "string",
  "channel": "string",
  "streamPath": "string",
  "offline": {
    "title": "string",
    "description": "string",
    "thumbnail": {
      "width": 0,
      "height": 0,
      "path": "http://example.com",
      "childImages": [
        {
          "width": 0,
          "height": 0,
          "path": "http://example.com"
        }
      ]
    }
  }
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|string|true|none|none|
|title|string|true|none|none|
|description|string|true|none|none|
|thumbnail|[ImageModel](#schemaimagemodel)¦null|true|none|none|
|owner|string|true|none|none|
|channel|string|false|none|The creator channel this livestream belongs to.|
|streamPath|string|true|none|none|
|offline|object|true|none|none|
|» title|string¦null|true|none|none|
|» description|string¦null|true|none|none|
|» thumbnail|[ImageModel](#schemaimagemodel)¦null|true|none|none|

<h2 id="tocS_SocialLinksModel">SocialLinksModel</h2>
<!-- backwards compatibility -->
<a id="schemasociallinksmodel"></a>
<a id="schema_SocialLinksModel"></a>
<a id="tocSsociallinksmodel"></a>
<a id="tocssociallinksmodel"></a>

```json
{
  "property1": "http://example.com",
  "property2": "http://example.com"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|**additionalProperties**|string(uri)|false|none|none|

<h2 id="tocS_DiscordServerModel">DiscordServerModel</h2>
<!-- backwards compatibility -->
<a id="schemadiscordservermodel"></a>
<a id="schema_DiscordServerModel"></a>
<a id="tocSdiscordservermodel"></a>
<a id="tocsdiscordservermodel"></a>

```json
{
  "id": "string",
  "guildName": "string",
  "guildIcon": "string",
  "inviteLink": "http://example.com",
  "inviteMode": "string"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|string|true|none|none|
|guildName|string|true|none|none|
|guildIcon|string|true|none|none|
|inviteLink|string(uri)¦null|true|none|none|
|inviteMode|string|true|none|none|

<h2 id="tocS_DiscordRoleModel">DiscordRoleModel</h2>
<!-- backwards compatibility -->
<a id="schemadiscordrolemodel"></a>
<a id="schema_DiscordRoleModel"></a>
<a id="tocSdiscordrolemodel"></a>
<a id="tocsdiscordrolemodel"></a>

```json
{
  "server": "string",
  "roleName": "string"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|server|string|true|none|none|
|roleName|string|true|none|none|

<h2 id="tocS_UserModel">UserModel</h2>
<!-- backwards compatibility -->
<a id="schemausermodel"></a>
<a id="schema_UserModel"></a>
<a id="tocSusermodel"></a>
<a id="tocsusermodel"></a>

```json
{
  "id": "string",
  "username": "string",
  "profileImage": {
    "width": 0,
    "height": 0,
    "path": "http://example.com",
    "childImages": [
      {
        "width": 0,
        "height": 0,
        "path": "http://example.com"
      }
    ]
  }
}

```

Represents some basic information of a user (id, username, and profile image).

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|string|true|none|none|
|username|string|true|none|none|
|profileImage|[ImageModel](#schemaimagemodel)|true|none|none|

<h2 id="tocS_UserSelfModel">UserSelfModel</h2>
<!-- backwards compatibility -->
<a id="schemauserselfmodel"></a>
<a id="schema_UserSelfModel"></a>
<a id="tocSuserselfmodel"></a>
<a id="tocsuserselfmodel"></a>

```json
{
  "id": "string",
  "username": "string",
  "profileImage": {
    "width": 0,
    "height": 0,
    "path": "http://example.com",
    "childImages": [
      {
        "width": 0,
        "height": 0,
        "path": "http://example.com"
      }
    ]
  },
  "email": "string",
  "displayName": "string"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|string|true|none|none|
|username|string|true|none|none|
|profileImage|[ImageModel](#schemaimagemodel)|true|none|none|
|email|string|true|none|none|
|displayName|string|true|none|none|

<h2 id="tocS_ConnectedAccountModel">ConnectedAccountModel</h2>
<!-- backwards compatibility -->
<a id="schemaconnectedaccountmodel"></a>
<a id="schema_ConnectedAccountModel"></a>
<a id="tocSconnectedaccountmodel"></a>
<a id="tocsconnectedaccountmodel"></a>

```json
{
  "key": "string",
  "name": "string",
  "enabled": true,
  "iconWhite": "string",
  "connectedAccount": {
    "id": "string",
    "remoteUserId": "string",
    "remoteUserName": "string",
    "data": {
      "canJoinGuilds": true
    }
  },
  "connected": true,
  "isAccountProvider": true
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|key|string|true|none|Unique identifier for the account type.|
|name|string|true|none|Display-friendly label for the `key`.|
|enabled|boolean|true|none|Determines if the system allows this account to be connected to.|
|iconWhite|string|true|none|none|
|connectedAccount|object¦null|true|none|none|
|» id|string|true|none|none|
|» remoteUserId|string|true|none|none|
|» remoteUserName|string|true|none|none|
|» data|object¦null|true|none|none|
|»» canJoinGuilds|boolean|true|none|none|
|connected|boolean|true|none|If true, the user is connected and the `connectedAccount` will have data about the account.|
|isAccountProvider|boolean|true|none|none|

<h2 id="tocS_CommentModel">CommentModel</h2>
<!-- backwards compatibility -->
<a id="schemacommentmodel"></a>
<a id="schema_CommentModel"></a>
<a id="tocScommentmodel"></a>
<a id="tocscommentmodel"></a>

```json
{
  "id": "string",
  "blogPost": "string",
  "user": {
    "id": "string",
    "username": "string",
    "profileImage": {
      "width": 0,
      "height": 0,
      "path": "http://example.com",
      "childImages": [
        {
          "width": 0,
          "height": 0,
          "path": "http://example.com"
        }
      ]
    }
  },
  "text": "string",
  "replying": "string",
  "postDate": "2019-08-24T14:15:22Z",
  "editDate": "2019-08-24T14:15:22Z",
  "pinDate": "2019-08-24T14:15:22Z",
  "editCount": 0,
  "isEdited": true,
  "likes": 0,
  "dislikes": 0,
  "score": 0,
  "interactionCounts": {
    "like": 0,
    "dislike": 0
  },
  "totalReplies": 0,
  "replies": [
    {
      "id": "string",
      "blogPost": "string",
      "user": {
        "id": "string",
        "username": "string",
        "profileImage": {
          "width": 0,
          "height": 0,
          "path": "http://example.com",
          "childImages": [
            {
              "width": 0,
              "height": 0,
              "path": "http://example.com"
            }
          ]
        }
      },
      "text": "string",
      "replying": "string",
      "postDate": "2019-08-24T14:15:22Z",
      "editDate": "2019-08-24T14:15:22Z",
      "pinDate": "2019-08-24T14:15:22Z",
      "editCount": 0,
      "isEdited": true,
      "likes": 0,
      "dislikes": 0,
      "score": 0,
      "interactionCounts": {
        "like": 0,
        "dislike": 0
      },
      "totalReplies": 0,
      "replies": [],
      "userInteraction": [
        "like"
      ]
    }
  ],
  "userInteraction": [
    "like"
  ]
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|string|true|none|none|
|blogPost|string|true|none|none|
|user|[UserModel](#schemausermodel)|true|none|Represents some basic information of a user (id, username, and profile image).|
|text|string|true|none|none|
|replying|string¦null|true|none|none|
|postDate|string(date-time)|true|none|none|
|editDate|string(date-time)¦null|true|none|none|
|pinDate|string(date-time)¦null|false|none|none|
|editCount|integer|true|none|none|
|isEdited|boolean|true|none|none|
|likes|integer|true|none|none|
|dislikes|integer|true|none|none|
|score|integer|true|none|none|
|interactionCounts|object|true|none|none|
|» like|integer|true|none|none|
|» dislike|integer|true|none|none|
|totalReplies|integer|false|none|none|
|replies|[[CommentModel](#schemacommentmodel)]|false|none|This is present (but possibly empty) for top-level comments. This is never present for reply comments.|
|userInteraction|[UserInteractionModel](#schemauserinteractionmodel)|true|none|none|

<h2 id="tocS_UserNotificationModel">UserNotificationModel</h2>
<!-- backwards compatibility -->
<a id="schemausernotificationmodel"></a>
<a id="schema_UserNotificationModel"></a>
<a id="tocSusernotificationmodel"></a>
<a id="tocsusernotificationmodel"></a>

```json
{
  "creator": {
    "id": "string",
    "owner": "string",
    "title": "string",
    "urlname": "string",
    "description": "string",
    "about": "string",
    "category": "string",
    "cover": {
      "width": 0,
      "height": 0,
      "path": "http://example.com",
      "childImages": [
        {
          "width": 0,
          "height": 0,
          "path": "http://example.com"
        }
      ]
    },
    "icon": {
      "width": 0,
      "height": 0,
      "path": "http://example.com",
      "childImages": [
        {
          "width": 0,
          "height": 0,
          "path": "http://example.com"
        }
      ]
    },
    "liveStream": {
      "id": "string",
      "title": "string",
      "description": "string",
      "thumbnail": {
        "width": 0,
        "height": 0,
        "path": "http://example.com",
        "childImages": [
          {
            "width": 0,
            "height": 0,
            "path": "http://example.com"
          }
        ]
      },
      "owner": "string",
      "channel": "string",
      "streamPath": "string",
      "offline": {
        "title": "string",
        "description": "string",
        "thumbnail": {
          "width": 0,
          "height": 0,
          "path": "http://example.com",
          "childImages": [
            {
              "width": 0,
              "height": 0,
              "path": "http://example.com"
            }
          ]
        }
      }
    },
    "subscriptionPlans": [
      {
        "id": "string",
        "title": "string",
        "description": "string",
        "price": "string",
        "priceYearly": "string",
        "currency": "string",
        "logo": "string",
        "interval": "string",
        "featured": true,
        "allowGrandfatheredAccess": true,
        "discordServers": [
          {
            "id": "string",
            "guildName": "string",
            "guildIcon": "string",
            "inviteLink": "http://example.com",
            "inviteMode": "string"
          }
        ],
        "discordRoles": [
          {
            "server": "string",
            "roleName": "string"
          }
        ]
      }
    ],
    "discoverable": true,
    "subscriberCountDisplay": "string",
    "incomeDisplay": true,
    "defaultChannel": "string"
  },
  "userNotificationSetting": {
    "createdAt": "2019-08-24T14:15:22Z",
    "updatedAt": "2019-08-24T14:15:22Z",
    "id": "string",
    "contentEmail": true,
    "contentFirebase": true,
    "creatorMessageEmail": true,
    "user": "string",
    "creator": "string"
  }
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|creator|[CreatorModelV2](#schemacreatormodelv2)|true|none|none|
|userNotificationSetting|object|true|none|none|
|» createdAt|string(date-time)|false|none|none|
|» updatedAt|string(date-time)¦null|false|none|none|
|» id|string|false|none|none|
|» contentEmail|boolean|true|none|none|
|» contentFirebase|boolean|true|none|none|
|» creatorMessageEmail|boolean|true|none|none|
|» user|string|true|none|none|
|» creator|string|true|none|none|

<h2 id="tocS_UserSubscriptionModel">UserSubscriptionModel</h2>
<!-- backwards compatibility -->
<a id="schemausersubscriptionmodel"></a>
<a id="schema_UserSubscriptionModel"></a>
<a id="tocSusersubscriptionmodel"></a>
<a id="tocsusersubscriptionmodel"></a>

```json
{
  "startDate": "2019-08-24T14:15:22Z",
  "endDate": "2019-08-24T14:15:22Z",
  "paymentID": 0,
  "interval": "string",
  "paymentCancelled": true,
  "plan": {
    "id": "string",
    "title": "string",
    "description": "string",
    "price": "string",
    "priceYearly": "string",
    "currency": "string",
    "logo": "string",
    "interval": "string",
    "featured": true,
    "allowGrandfatheredAccess": true,
    "discordServers": [
      {
        "id": "string",
        "guildName": "string",
        "guildIcon": "string",
        "inviteLink": "http://example.com",
        "inviteMode": "string"
      }
    ],
    "discordRoles": [
      {
        "server": "string",
        "roleName": "string"
      }
    ]
  },
  "creator": "string"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|startDate|string(date-time)¦null|true|none|none|
|endDate|string(date-time)¦null|true|none|none|
|paymentID|integer¦null|true|none|none|
|interval|string|true|none|none|
|paymentCancelled|boolean¦null|false|none|none|
|plan|[SubscriptionPlanModel](#schemasubscriptionplanmodel)|true|none|none|
|creator|string|true|none|none|

<h2 id="tocS_FaqSectionModel">FaqSectionModel</h2>
<!-- backwards compatibility -->
<a id="schemafaqsectionmodel"></a>
<a id="schema_FaqSectionModel"></a>
<a id="tocSfaqsectionmodel"></a>
<a id="tocsfaqsectionmodel"></a>

```json
{
  "faqs": [
    {
      "createdAt": "2019-08-24T14:15:22Z",
      "updatedAt": "2019-08-24T14:15:22Z",
      "id": "string",
      "question": "string",
      "answer": "string",
      "status": "public",
      "link": "string",
      "order": 0,
      "faqSection": "string"
    }
  ],
  "createdAt": "2019-08-24T14:15:22Z",
  "updatedAt": "2019-08-24T14:15:22Z",
  "id": "string",
  "name": "string",
  "description": "string",
  "status": "public",
  "order": 0
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|faqs|[object]|true|none|none|
|» createdAt|string(date-time)|true|none|none|
|» updatedAt|string(date-time)¦null|true|none|none|
|» id|string|true|none|none|
|» question|string|true|none|none|
|» answer|string|true|none|This field may contain HTML that should be rendered.|
|» status|string|true|none|none|
|» link|string|true|none|none|
|» order|number|true|none|none|
|» faqSection|string|true|none|none|
|createdAt|string(date-time)|true|none|none|
|updatedAt|string(date-time)¦null|true|none|none|
|id|string|true|none|none|
|name|string|true|none|none|
|description|string|true|none|none|
|status|string|true|none|none|
|order|number|true|none|none|

#### Enumerated Values

|Property|Value|
|---|---|
|status|public|
|status|public|

<h2 id="tocS_UserInteractionModel">UserInteractionModel</h2>
<!-- backwards compatibility -->
<a id="schemauserinteractionmodel"></a>
<a id="schema_UserInteractionModel"></a>
<a id="tocSuserinteractionmodel"></a>
<a id="tocsuserinteractionmodel"></a>

```json
[
  "like"
]

```

### Properties

*None*

<h2 id="tocS_EdgesModel">EdgesModel</h2>
<!-- backwards compatibility -->
<a id="schemaedgesmodel"></a>
<a id="schema_EdgesModel"></a>
<a id="tocSedgesmodel"></a>
<a id="tocsedgesmodel"></a>

```json
{
  "edges": [
    {
      "hostname": "string",
      "queryPort": 0,
      "bandwidth": 0,
      "allowDownload": true,
      "allowStreaming": true,
      "datacenter": {
        "countryCode": "string",
        "regionCode": "string",
        "latitude": 0,
        "longitude": 0
      }
    }
  ],
  "client": {}
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|edges|[[EdgeModel](#schemaedgemodel)]|true|none|none|
|client|object|true|none|none|

<h2 id="tocS_EdgeModel">EdgeModel</h2>
<!-- backwards compatibility -->
<a id="schemaedgemodel"></a>
<a id="schema_EdgeModel"></a>
<a id="tocSedgemodel"></a>
<a id="tocsedgemodel"></a>

```json
{
  "hostname": "string",
  "queryPort": 0,
  "bandwidth": 0,
  "allowDownload": true,
  "allowStreaming": true,
  "datacenter": {
    "countryCode": "string",
    "regionCode": "string",
    "latitude": 0,
    "longitude": 0
  }
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|hostname|string|true|none|none|
|queryPort|integer|true|none|none|
|bandwidth|integer(int64)|true|none|none|
|allowDownload|boolean|true|none|none|
|allowStreaming|boolean|true|none|none|
|datacenter|[EdgeDataCenter](#schemaedgedatacenter)|true|none|Location information for a datacenter. Not required.|

<h2 id="tocS_EdgeDataCenter">EdgeDataCenter</h2>
<!-- backwards compatibility -->
<a id="schemaedgedatacenter"></a>
<a id="schema_EdgeDataCenter"></a>
<a id="tocSedgedatacenter"></a>
<a id="tocsedgedatacenter"></a>

```json
{
  "countryCode": "string",
  "regionCode": "string",
  "latitude": 0,
  "longitude": 0
}

```

Location information for a datacenter. Not required.

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|countryCode|string|true|none|none|
|regionCode|string|true|none|none|
|latitude|number|true|none|none|
|longitude|number|true|none|none|

<h2 id="tocS_UpdateProgressRequest">UpdateProgressRequest</h2>
<!-- backwards compatibility -->
<a id="schemaupdateprogressrequest"></a>
<a id="schema_UpdateProgressRequest"></a>
<a id="tocSupdateprogressrequest"></a>
<a id="tocsupdateprogressrequest"></a>

```json
{
  "id": "string",
  "contentType": "video",
  "progress": 0
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|string|true|none|The video or audio attachment identifier for the piece of media that is being updated. Note: this is *not* the blogPost identifier.|
|contentType|string|true|none|Which type of media the corresponding identifier is.|
|progress|integer|true|none|The progress through the media that has been consumed by the user, in seconds.|

#### Enumerated Values

|Property|Value|
|---|---|
|contentType|video|
|contentType|audio|

<h2 id="tocS_GetProgressRequest">GetProgressRequest</h2>
<!-- backwards compatibility -->
<a id="schemagetprogressrequest"></a>
<a id="schema_GetProgressRequest"></a>
<a id="tocSgetprogressrequest"></a>
<a id="tocsgetprogressrequest"></a>

```json
{
  "ids": [
    "string"
  ],
  "contentType": "blogPost"
}

```

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|ids|[string]|true|none|The identifiers of the blog posts from which progress should be retrieved.|
|contentType|string|true|none|The type of the corresponding identifiers. The only value currently is `blogPost`.|

#### Enumerated Values

|Property|Value|
|---|---|
|contentType|blogPost|

<h2 id="tocS_GetProgressResponse">GetProgressResponse</h2>
<!-- backwards compatibility -->
<a id="schemagetprogressresponse"></a>
<a id="schema_GetProgressResponse"></a>
<a id="tocSgetprogressresponse"></a>
<a id="tocsgetprogressresponse"></a>

```json
[
  {
    "id": "string",
    "progress": 100
  }
]

```

A list of objects containing progress values for the requested identifiers. If no progress has been posted to an identifier, it may either not appear in the resulting list, or appear with a progress of `0`.

### Properties

|Name|Type|Required|Restrictions|Description|
|---|---|---|---|---|
|id|string|true|none|none|
|progress|integer|true|none|Percentage of the blog post's media that has been consumed so far. Ranges from 0 to 100.|

