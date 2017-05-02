---
title: Beeminder API Reference

language_tabs:
  - curl 
  - ruby

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---
# This doc is under development

You can find the [real, published Beeminder API Docs](https://www.beeminder.com/api) on our website.

## I repeat, the complete API docs are at [beeminder.com/api](https://www.beeminder.com/api)

# API Reference 
## Welcome 
In case you’re here to automate adding data to Beeminder, there’s a good chance we’ve got you covered with our
[Zapier integration](http://beeminder.com/zapier "Zapier is a service like IFTTT that connects hundreds of disparate webservices. In the case of Beeminder you can create triggers in other webservices that automatically cause data to be added to Beeminder graphs.")
or our
[IFTTT integration](http://ifthisMINDthat.com "IFTTT = If This Then That").
If you’re looking for ideas for things to do with the Beeminder API, we have a
[blog post with lots of examples](http://blog.beeminder.com/api This was the blog post that originally announced our API in 2012 but we've updated occasionally since then with links to things people are doing with it").
The
[tech category of our forum](http://forum.beeminder.com/c/tech "Yay Discourse.org!")
is a good place to ask questions and show off what you’re working on.


It’s really important to us that this API be easy for developers to use so please don’t be shy about asking us questions.
Whether you post in
[the forum](http://forum.beeminder.com "The above link is the Tech subset of the forum; this link is to the main page for the forum")
or email us at support@beeminder.com we’ve invariably found that questions people avoided asking for fear they were dumb turned out to point to things we needed to improve in the API or the documentation.
So lean on us heavily as you’re hacking away with our API — it helps us a lot when you do!


## API Endpoint
The base URL for all requests is
`https://www.beeminder.com/api/v1`

# Authentication

All API endpoints require authentication.
There are two ways to authenticate.


## Personal authentication token


> For example, if your username is `alice` and your token is `abc123` you can query information about your `weight` goal like so:

```curl
GET https://www.beeminder.com/api/v1/users/alice/goals/weight.json?auth_token=abc123
```

This authentication pattern is for making API calls just to your own Beeminder account.

After you
[log in to Beeminder](https://www.beeminder.com/users/sign_in),
visit
[https://www.beeminder.com/api/v1/auth_token.json](https://www.beeminder.com/api/v1/auth_token.json )
to get your personal auth token.

Append it to API requests you make as an additional GET or POST parameter `auth_token`.

<aside class="notice">
Note! a common bit of confusion is mixing up the parameter name. For your personal auth token the parameter name is <code>auth_token</code>. If you try to send a personal auth token as <code>access_token</code> you will get an error. 
</aside>



## Client Oauth

This authentication pattern is for clients (applications) accessing the Beeminder API on a user’s behalf.
Beeminder implements the
[OAuth 2](http://oauth.net/ )
provider protocol to allow access reasonably securely.
We aren't going to provide a full step-by step tutorial on how to implement an OAuth2 client in this doc.
If you're not familiar with the OAuth2 authentication flow, first go read 
[this](https://aaronparecki.com/oauth-2-simplified/ "Aaron literally wrote the book on OAuth2."), or 
[this](https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2 "Anyone notice that Digital Ocean has consistently great tutorials and docs for things?")
for a more in-depth background. 

Now, once we're all on the same page, here are the things you need to know to build an OAuth2 client with Beeminder.


### 1. Register your app

Register your app at
[beeminder.com/apps/new](https://www.beeminder.com/apps/new ).
Application name and redirect URL are required.
The redirect URL is where the user will be sent after authorizing your app.
You can change this later if you're not sure what it should be right now.

Make sure you copy down the client secret we show you!

### 2. Send your users to the Beeminder authorization URL

> The base URL for Beeminder authorization is

```curl
https://www.beeminder.com/apps/authorize
```

> Include the following parameters: 
```curl
* <code>client_id</code>: Your application’s client ID.
* <code>redirect_uri</code>: This is where Beeminder will send the user after they have authorized your app.
This <em>must match</em> the redirect URL you supplied when you registered your app above.
Make sure to
<a href="http://en.wikipedia.org/wiki/Percent-encoding" title="Where you replace characters that have special meaning in URLs with a percent sign and their ascii number (or plus signs for spaces)">url-encode</a>
this if it contains any special characters like question marks or ampersands.</li>
<code>response_type</code>: Currently this should just always be set to the value “<code>token</code>”.</li>

https://www.beeminder.com/apps/authorize?client_id=xyz456&redirect_uri=http://foo.com/auth_callback&response_type=token
```

You will need to redirect your user to your Beeminder authorization URL. The base URL is the same for all apps. You will need to add in your own `client_id`, `redirect_uri`, and `response_type` parameters.

You can see a list of your registered apps and retrieve their `client_id`'s at
[beeminder.com/apps](https://www.beeminder.com/apps "List of apps you've registered, not to be confused with the list of apps you've authorized to access your Beeminder account, or the list of services you've authorized Beeminder to access")


### 3. Receive and store user’s access token

> For example, we might see for user `alice`, the following string appended to the redirect URL: 

```curl
 ?access_token=abc123&username=alice
```

After the user authorizes your application they’ll be redirected to the `redirect_uri` that you specified, with two additional parameters,
`access_token` and
`username`, in the
[query string](http://en.wikipedia.org/wiki/Query_string "The query string is the parameters that come after the question mark in a URL").

You should set up your server to handle this GET request and have it remember each user’s access token.
The access token uniquely identifies the user’s authorization for your app.

The username is provided in this response for convenenience.


### 4. Include access token as a parameter

> You can literally use `me` in place of the username for any endpoint and it will be macro-expanded to the username of the authorized user.

```curl
  GET https://www.beeminder.com/api/v1/me.json?access_token=abc123
``` 

Append the access token as a parameter on any API requests you make on behalf of that user.
For example, your first request will probably be to get information about the
[User](#user)
who just authorized your app.


### 5. Optional: De-authorization callback


```curl
  POST https://www.yourapp.com/beeminder-deauth\ 
  -d 'access_token=abc123'
```

If you provide a Post De-Authorization Callback URL when you register your client, we will make a `POST` to your endpoint when a user removes your app. The `POST` will include the access_token removed in the body of the request.

### 6. Optional: Autofetch callback


```curl
  POST https://www.yourapp.com/beeminder-autofetch\
  -d 'username=alice'
  -d 'slug=weight'
```

The autofetch callback URL is also optional.
We will `POST` to this URL if provided, including the params `username`, and `slug` when the user wants new data from you.
E.g., when the user pushes the manual refresh button, or prior to sending alerts to the user, and before derailing the goal at the end of an eep day.

# Kittens

## Get All Kittens

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get
```

```shell
curl "http://example.com/api/kittens"
  -H "Authorization: meowmeowmeow"
```


> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Max",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

This endpoint retrieves all kittens.

### HTTP Request

`GET http://example.com/api/kittens`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -H "Authorization: meowmeowmeow"
```


> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve

