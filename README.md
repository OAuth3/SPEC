# SPEC

The OAuth3 Specification

Where possible, oauth3 should behave as a strict superset of OIDC (which we didn't know about when we started this). All variations should be noted, and fixed if possible.

* summary
* security
* [oauth3.json](#oauth3json)
* [oauth3.html](#oauth3html)
  * `response_type=directive`
* [reserved scope](#reserved-scope)

One of the goals of OAuth3 is to maintain backwards compatible with OAuth2 such that any OAuth2 provider can easily become an OAuth3 provider.

Another one of the goals is to standardize OAuth3 in such a way that it is useful for "the nexternet", the "peer web" - a federated, delegated internet that functions more like email - more like it would have if we had always-on internet instead of dial-up back in the days when email systems wanted you to download your data so they could delete it.

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

If the provider is not technically competent enough to provide an `oauth3.json`, `https://oauth3.org/providers/example.com/oauth3.json` may host such file on their behalf.

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

* `authn_scope` should be an empty string, but for systems like Google+'s OAuth2 implementation, it is the minimal scope required to allow the app to log the user in and get an `app_scoped_id`.

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

Ideal Flow
==========

Let's consider two parties: social.com and clickbait.com

### Authorization Code

1. The user visits ClickBait.com
2. The ClickBait.com application downloads to the user's browser
  * oauth3.html
  * oauth3.json
3. The user clicks on a *Connect with Social.com* button
4. A pop-up window opens
  * `https://{{consumer_uri}}/api/oauth3/authorization_redirect/{{provider_uri}}?state={{browser_state}}`
  * `https://clickbait.com/api/oauth3/authorization_redirect/social.com?state=abc123`
    * note that https is REQUIRED. No unencrypted connections
    * note this state is the *browser* state, generated by code from `oauth3.js`
    * note that social.com is *not arbitrary* and it should be URI Encoded
5. The clickbait.com **server** does some work that results in an redirect (browser side)
  * stores the browser state
  * creates a server state
  * redirects
    * `https://{{provider_uri}}/api/oauth3/authorization_dialog?state={{server_state}}&redirect_uri={{uri_encode(redirect_uri)}}`
    * `https://social.com/api/oauth3/authorization_dialog?grant_type=authorization_code&state=xyz789&redirect_uri=https%3A%2F%2Fclickbait.com%2Fapi%2Foauth3%2Fauthorization_code_callback`
      * *client_id* ?
      * redirect_uri is https://clickbait.com/api/oauth3/authorization_code_callback
  * It might work well to JWT encode the browser state `server_state=base64({ s: random, b: browser_state, p: provider_uri })`
6. The browser will now be showing social.com's login dialog

**TO BE CONTINUED**

social.com will redirect the browser to https://clickbait.com/api/oauth3/authorization_code_callback?state=xyz789&code=blahblahblah

api.clickbait.com will exchange the code for a token at https://api.social.com/api/oauth3/access_token

api.clickbait.com will then redirect the browser back to https://clickbait.com/oauth3.html#state=abc123&access_token=mno456&error=&error_message=&token=qrs000

both api.clickbait.com and www.clickbait.com can now use access_token to communicate with api.social.com

www.clickbait.com can use token to communicate with api.clickbait.com


### Implicit Grant


### Token Definitions

I've gotta check in with the OIDC (OpenID Connect) folks, but I think this is how OAuth3 should define the JWT keys.

References

  * [OIDC ID Token Spec](http://openid.net/specs/openid-connect-core-1_0.html#IDToken)
  * [ID Tokens Explained Simply](https://www.tbray.org/ongoing/When/201x/2013/04/04/ID-Tokens)
  * http://developer.okta.com/docs/api/resources/oidc

Draft Brief

There are generally 2-4 parties involved in issuing, using, and validating a token: the api that signs with the private key (the oauth3.org, auth0, stormpath, or self-hosted), the client app (static assets, mobile app), the resource manager (facebook), and the user (providing credentials and permissions to warrant issuence).

#### Parties

* `iss` - issuer (token signer), holder of private keys, signer of tokens (api or potentially device), location from which to fetch public keys for validation (this is often also the provider - facebook, the twitter, the api.daplie.com, but it could be the auth0, the stormpath, the oauth3.org, the self-hosted signer)
* `sub` - subject, (natural user, token authorizer), a Pairwise Pseudonymous Identifier (PPID) for user/device account(s)). OIDC seems to (mistakenly) assume that credentials are linked to exactly one account (unlike Google, Facebook, etc), so we have to figure out how to work around that constraint - maybe using comma-separated list, maybe restricting multiple accounts to internal implementation per-provider and standardizing only one-account per token
* `aud` - audience, (token receiver) fetcher of public keys, validater of tokens
* `azp` - authorized party, (token sender) supplier of credentials, local storer of tokens (i.e. client app (`origin`s, `referer`s), mobile app)

Again

* It's the `audience` if it needs to validate a token (typically to provide a resource or service)
* It's the `subject` if it is the owner (person or device), which grants permissions, identified by an non-changing account id (encrypted or not encrypted?).
* It's the `issuer` if it signs the token with the private key (or potentially if it delegates signing authority, but is still reachable for public key validation)
* It's the `authorized party` if it holds tokens on behalf of the user (or device) and sends them out to the `audience`s

Draft Examples

* `iss` - issuer, the cname referring to the origin of the private key (and often where public keys can be found to verify the signature)
  * generally `api.example.com` would be the issuer for the app hosted at `example.com`
  * if a 3rd party partner is allowed to use private key to sign on behalf of the 1st party, the 1st party is still the issuer (i.e. perhaps you use allow api.partner.com to sign with api.example.com keys). However, if the 1st party does not sign at all (it has no private key on premise and perhaps manages static resources only), then the 3rd party would be the issuer.

#### Metadata

* `auth_time` - almost always relevant for "natural users" (humans) - when the user last used their credentials
* `amr` - authentication modes - an array of stuff like passphrase, Authenticator, opt via email, otp via sms or phone, etc
