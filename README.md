# SPEC
The OAuth3 Specification

* summary
* security
* [oauth3.json](#oauth3json)
* [oauth3.html](#oauth3html)
  * `response_type=directive`
* [reserved scope](#reserved-scope)

One of the goals of OAuth3 is to maintain backwards compatible with OAuth2 such that any OAuth2 provider can easily become an OAuth3 provider.

Another one of the goals is to standardize OAuth3 in such a way that it is useful for "the nexternet", the "peer web" - a federated, delegated internet that functions more like email - more like it would have if we had always-on internet instead of dailup back in the days when email systems wanted you to download your data so they could delete it.

Since the goals are in stark contrast with one another, OAuth3 provides a discoverable, backwards compatible layer: oauth3.json.

Secure Connections with HTTPS
=============================

We live in the age of [Let's Encrypt](https://letsencrypt.org).
HTTPS Certificates are **free** and **automatic**. There is no excuse for http. Become competent.

oauth3.json
===========

Let's consider Facebook for a moment.

Users go to `www.facebook.com`. That's where the `oauth3.json` **must** live,
regardless of the fact that the API lives at `graph.facebook.com/api/v2`.

`https://facebook.com/oauth3.json`
  * **must** be accesible with or without the `www.`
  * **should** be a cacheable, static file
  * **should** be hosted on the bare domain, not redirected the `www.`
    * (but **may** be redirected if you're not technically competent enough to figure out how to make that work)

If the provider is not technically competent enough to provide an `oauth3.json`, `https://oauth3.org/providers/example.com/oauth3.json` may host such file on their behanlf.

**TODO** since this can be delegated and federated there will be a way to specify something like one of these:
  * `username@provider.com`
  * `username.provider.com`
  * `provider.com/username`

Why not `https://facebook.com/.well_known/oauth3.json`? Because the degree of competency required for knowing how to work with hidden dot files is higher than the level of competency that should be required to implement OAuth3 (assuming the libraries and documentation become available on this site) and that shouldn't be the limiting factor.

Ideal Contents
--------

Ideally, you wouldn't need an `oauth3.json` because you would be using the OAuth3-strict url structre, which looks like this.

```javascript
{ "authorization_dialog": {
    "method": "GET"
  , "url": "https://api.example.com/oauth3/authorization_dialog"
  }
, "access_token": {
    "method": "POST"
  , "url": "https://api.example.com/oauth3/access_token"
  }
, "inspect_token": {
    "method": "GET"
  , "url": "https://api.example.com/oauth3/inspect_token"
  }
, "authn_scope": ""
}
```

Backwards Compatibility
--------

But because there are many well-established providers out in the wild that have no standard to adhere to, as well as special circumstances dictating various locations of API, Session, and Asset serversl, the file might look more like this:

```javascript
{ "authorization_dialog": {
    "method": "GET"
  , "url": "https://www.facebook.com/dialog/oauth"
  }
, "access_token": {
    "method": "POST"
  , "url": "https://graph.facebook.com/v2.3/oauth/access_token"
  }
, "inspect_token": {
    "method": "GET"
  , "url": "https://graph.facebook.com/debug_token"
  }
, "authn_scope": "basic,login"
}
```

Explanation
-----------

* `oauthn_socpe` should be an empty string, but for systems like Google+'s OAuth2 implementation, it is the minimal scope required to allow the app to log the user in and get an `app_scoped_id`.

oauth3.html
===========

This file must live at the user endpoint, such as facebook.com/oauth3.html or johndoe.awesome.com/oauth3.html.

It facilitates communication between the provider and the consumer.

### Directive Autodiscovery Strategy

`GET https://example.com/oauth3.html?response_type=directive&state=1234&redirect_uri=<<REDIRECT_URI>>`
where `redirect_uri` may be `encodeURIComponent('https://myapp.com/oauth3.html?callback=onOauth3Directive')`

In the case that CORS is not available, this strategy allows using oauth3.html to retrieve oauth3.json and pass it back via url or post message.

The consumer's `oauth3.html` should be able to negotiate with the provider `oauth3.html` and simply callback into the main application without any concern on the developer's part.

Likewise, this could be a faster method of retrieving endpoints without CORS. I know that CORS is awesome, but for the purposes of widespread adoption, it may be best to leave all of the data retrieval magic up to window.postMessage since that doesn't require server configuration and it works all the way back to MSIE8.

Hmmm... the balance between doing things "the right way" and "the easy way"... I guess it will come down to what "Just Works"(TM) for the most people

Reserved Scope
==============

* `*` is reserved for *all permissions*. This is useful for:
  * the provider issues a token to its own frontend (i.e. JWT, other cookieless sessions)
  * one provider is requesting an account takeover from another provider (i.e. migrating from gmail to yahoo)
* `!` is reserved as a scope which may not be granted. This is useful for:
  * optional user switching
  * debugging your permissions dialog by causing it to popup every time
* `+` is reserved for *additive user switching*
  * user switching (add a login)
* `^` is reserved for *strict user switching*
  * user switching (replace current login)
* `oauth3_` anything prefixed with `oauth3_` is reserved for future use.

scope types
===========

Just in the thought stage here.

Standardized scopes might look something like this:

`oauth3_messages_rwx`

* read i.e. The app can read my messages
* write i.e. The app can write messages to me
* execute i.e. The app can write messages to others as me

`oauth3_friends_rwx`

* read i.e. The app can read friends list
* write i.e. The app can add me as a friend (or accept friends?)
* execute i.e. The app can add friend others as me
