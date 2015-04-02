# SPEC
The OAuth3 Specification

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

But because we have facebook and others that adhere to no standard and aren't going to put any redirects on their servers in our behalf, the file might look more like this:

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
