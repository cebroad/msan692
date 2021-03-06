# REST APIs

[Huge source of public APIs](https://www.publicapis.com/)

[Open payments REST API sample query](http://openpayments.us/data?query=John+Chan)

## OAuth

When it comes to a data collection program (the *client*), most REST APIs will require the program to authenticate using OAuth.  A *resource owner*, such as a Facebook or twitter user, has data stored on a *resource server*. The idea is that the Facebook user should not have to give their login credentials to a client program so that the client can login. Instead, the client gets an access token from an *authorization server* (usually just another server at target company) with approval (e.g., logging in with a dialogue box) for the resource owner. Given the access token, the client can access resources from the resource server.

This sequence is a tricky thing to get right but the basic idea is as follows.

1. The client program developer goes to a target company's developer website, such as twitter, and asks for an application ID or some other ID that will allow them access to the API. You might also get a so-called secret key from the company.
1. The client program contacts a URL on the  authorization server at the target company to request access using the application ID and usually a secret key.
2. The  authorization server at that URL accepts the credentials and sends an access token back to the  client program via a URL typically specified in the application settings on the target company's developer dashboard or as a `redirect_uri` parameter on the authentication request. As you can see in my examples, authentication requires that your  client pretend to be a web server for one HTTP request, to collect to the access token generated by the target company server.
3. The client program can then make REST API calls using the access token, usually for a limited period of time.

 From the [OAuth 2 document](https://tools.ietf.org/html/rfc6749):
 
```
     +--------+                               +---------------+
     |        |--(A)- Authorization Request ->|   Resource    |
     |        |                               |     Owner     |
     |        |<-(B)-- Authorization Grant ---|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(C)-- Authorization Grant -->| Authorization |
     | Client |                               |     Server    |
     |        |<-(D)----- Access Token -------|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(E)----- Access Token ------>|    Resource   |
     |        |                               |     Server    |
     |        |<-(F)--- Protected Resource ---|               |
     +--------+                               +---------------+
```

Here is [Twitter's Application authentication diagram](https://dev.twitter.com/oauth/application-only)

<img src="https://g.twimg.com/dev/documentation/image/appauth_0.png" width=800>

## Facebook example

We'll use Facebook as an example authentication.

[Code to pull whitehouse feed](https://github.com/parrt/msan692/blob/master/notes/code/facebook/feed.py). Requires app ID and secret from your app on your dashboard.

1. [Register/configure an App overview](https://developers.facebook.com/docs/apps/register)
2. [Create developer account](https://developers.facebook.com/async/onboarding/dialog/)
3. [Create new Facebook App](https://developers.facebook.com/apps/async/create/platform-setup/dialog/)<br>
  <img src=figures/fb-add-app.png width=250><br>
  Click "Basic Setup"<br>
  <img src=figures/fb-create-id.png width=250><br>
  It will ask you to verify your account using phone or credit card. Note that I had to drop my mobile phone from the account and re-added in order for Facebook to allow me to create the application ID.
4. You need to make sure that you have specified where Facebook should go during OAuth:<br>
<img src=figures/fb-oauth-settings.png>

**Save the application ID you get, and in a safe place.** You will need it to communicate with Facebook's API.

Once you're set up, you should see a dashboard like this:
 
<img src=figures/fb-dashboard.png width=400>

Notice that under products it says "Facebook login" but you can add lots of other products, depending on what you'd like to do. In our case we simply want to try logging in as a particular person.

Make sure you enable the app:

<img src=figures/fb-enable-desktop-app.png width=600>

and set up security on same page:

<img src=figures/fb-security.png width=400>

[FaceBook Login Flow](https://developers.facebook.com/docs/facebook-login/manually-build-a-login-flow)

```
https://www.facebook.com/dialog/oauth?
  client_id={app-id}
  &redirect_uri={redirect-uri}
```

browser will attempt to log the user in and then FB response after login does a redirect back to our site:

```
http://localhost:8000/?code=XXX
```

with a code that you need to use for communication with the API.

NOW, you need to exchange that code for an access token using your app secret.

```
https://graph.facebook.com/v2.3/oauth/access_token?
    client_id={app-id}
   &redirect_uri={redirect-uri}
   &client_secret={app-secret}
   &code={code-parameter}
```

Note that the `app-id` and `app-secret` should be kept private. Do not embed them in any HTML, JavaScript, or anything that could become public (such as in a webpage you post). I specify them on the commandline when I start up my application so as not to embed them anywhere.

The `code-parameter` is what you got back from login above.

[securing your FB requests](https://developers.facebook.com/docs/graph-api/securing-requests/)

