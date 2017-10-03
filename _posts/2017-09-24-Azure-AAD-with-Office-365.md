--- 
layout: post
title: "Azure AD and Office 365 OAuth integration through browsers and Postman."
author: "Author"
comments: true
tags : [Azure, OAuth, Azure Active Directory, Office 365, Outlook, Postman]
---

I spent last week answering a question. Is there a way to find available meeting times on a given user's Office 365 calendar next week? The short answer is **yes**. 

## Longer answer.
At work we have MSDN Azure subscriptions. These are linked to our organization's Azure Active Directory (AAD) and we can sign on to Azure with our Windows credentials. We also Office 365 accounts linked to our Exchange AD and Azure AD. I can seamlessly navigate to Office 365 on the browser after logging into Azure portal through my Windows account (OAuth magics). If this works for you too, then your Single Sign On (SSO) setup is also done. 

The goal here is to understand how Azure OAuth authentication works when calling the outlook APIs through Azure AD. Preferably, I want to achieve this through raw HTTP calls, using just the browsers or Postman REST client, so that the HTTP based protocols and interactions are clear and open to visual inspection. In a nutshell, the steps are as follows.

-   Create an Azure AD application / Service principal.
-   Redirect user to Exchange Active Directory for Authentication. 
-   Return Auth Code to user after Azure AD, through OAuth, has authenticated against organization's Exchange AD. 
-   Convert OAuth Code into Azure bearer token. 
-   Call Microsoft Office APIs in SSO mode using token received above to retrieve the available meeting times for a employee given a specified time window. 

Now, many online resources already exist on this, yet getting all the pieces playing together was rough. So, we'll walk the route I did and hope that the eco-system clears itself up. 

In the process, I will briefly touch on OAuth in Azure, Azure AD, Scopes and Resources in MS Online API, Azure Service Principals aka App registrations, App permissions aka OAuth *on-behalf-of consent* flow, Azure bearer tokens in Postman, JSON Web Tokens (JWT) and the Microsoft Graph explorer. Oh! and the Graph and Outlook sandboxes. 

*Note: In Azure, things change. The information here is of 24th Sept 2017.*

*For the rest you need an Azure subscription, an Office 365 account, and an Azure AD membership. You can have non work accounts for all three, but its a lot easier if your company admin has done the configuring for you. Blogs are the way to go here if you want to setup trial accounts for each.*

## findMeetingTimes API

Do such APIs exist? Well, yes, they do. Problem is, more than __*one*__ exist.

#### Outlook/Office 365 APIs
Documentation link here - [https://msdn.microsoft.com/en-us/office/office365/api/calendar-rest-operations#find-meeting-times](https://msdn.microsoft.com/en-us/office/office365/api/calendar-rest-operations#find-meeting-times),
The API is shown below - 
    
    POST https://outlook.office.com/api/{version}/me/findmeetingtimes
and there are *three* *version*s of it - *v1.0, v2.0, beta*.

Moreover, to use the API, the docs mention that we also need at least one of the following scopes -  `https://outlook.office.com/calendars.read.shared, wl.calendars, wl.contacts_calendars`. Scopes are an important concept in Azure auth governing permissions, and we'll see more of them in later sections.

The sandbox to play-around with all of this is here [https://oauthplay.azurewebsites.net/](https://oauthplay.azurewebsites.net/).

#### Microsoft Graph APIs
This is the second, latest and greatest option from Microsoft. Again, documentation is here - [https://developer.microsoft.com/en-us/graph/docs/concepts/findmeetingtimes_example](https://developer.microsoft.com/en-us/graph/docs/concepts/findmeetingtimes_example), the API looks like this 

    POST https://graph.microsoft.com/v1.0/me/findMeetingTimes

and the sandbox (called MS Graph Explorer), is here - [https://developer.microsoft.com/en-us/graph/graph-explorer](https://developer.microsoft.com/en-us/graph/graph-explorer).
#### Outlook vs Graph.

So why two? When to use what? 

Well previously MS had multiple APIs available - Office, Azure AD, etc. MS Graph is intended to unify all of these endpoints into a single REST-ful gateway for all of Microsoft's underlying platform APIs viz. Azure AD, Excel, Outlook, OneDrive, OneNote, SharePoint etc. It is not fully at par with existing APIs, but Microsoft's recommendation is to prefer MS Graph unless you need a feature which it does not have.

For current and future work, prefer MS Graph over other means.

#### API Request body.

In either case the POST body is the same, so that's a relief. A detailed request is given towards the end of this post. 

So endpoints - Check. 

Request Body and Response - Check. 

Well, on to Azure AD and OAuth SSO, then.
## Azure Active Directory (AAD) and OAuth.
Azure OAuth based authentication is a big and complex topic and cannot be covered adequately in a single blog post. Conceptually, the Azure OAuth flow is like ![this](/blog/img/posts/azureoauth.png). 

*The diagram source and more documentation can be found at [https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-protocols-oauth-code#main](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-protocols-oauth-code#main).*

In our case, as mentioned before, the full authentication process needs the following - 
-   Create an Azure AD application / Service principal.
-   Redirect user to Exchange Active Directory for Authentication. 
-   Return Auth Code to user after Azure AD, through OAuth, has authenticated against organization's Exchange AD. 
-   Convert OAuth Code into Azure bearer token. 
-   Call Microsoft Office APIs in SSO mode using token received above to retrieve the available meeting times for a employee given a specified time window. 

But lets look at the details, where the devils lurk. 

#### Theory - App Registration / Generate Service Principal
First, someone needs to get authenticated. This, in Azure terms, is a Service Principal. This is achieved by registering an application with Azure AD, which gives us three important keys, the *tenantId, clientId* and the *clientSecret* after registration.
*For details about how to register an app, see  [https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-integrating-applications](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-integrating-applications).*

After registration, we save the following key-value pairs for subsequent use.
-   *tenantId*: guid looking code of the AD instance in your Azure subscription.
-   *clientId*: guid looking code of the application being registered.
-   *clientSecret*: guid looking key corresponding to a code we create in the application properties on Azure. 

*Note : The clientSecret will be shown only once when creating a app property. If you don't note it down, you need to delete and recreate a key. It canot be recovered.*
#### Theory - Azure AD and OAuth.
The OAuth dance is a two-step process here.
-   Get a authentication code from the underlying authentication provider (OpenId, Active Directory).
-   Convert that code into a [JSON Web Token](https://jwt.io). For subsequent calls, this token needs to be used as the Authorization header.

__Theory - Authorize API__

The following are the current logon/authorization endpoints for both v1.0 and v2.0.

    https://login.microsoftonline.com/{tenantId}/oauth2/authorize
    https://login.microsoftonline.com/{tenantId/oauth2/v2.0/authorize

and a sample logon request with query parameters is as follows - 

    GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=CLIENT_ID&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F&response_type=code&scope=openid+https://office.outlook.com/Calendars.Read.Shared

The query parameter definitions are given below - 

    -   client_id: the clientId generated before during registering the app. This lets Azure know which app/service principal is requesting the logon.
    -   redirect_uri: the location that Azure will redirect to once the user has granted consent to the app. This value must correspond to the value of Redirect URI used when *registering the app*.
    -   response_type: the type of response the app is expecting. For the Authorization Grant Flow, this should always be the string `code`.
    -   scope: a space-delimited list of access scopes that the app requires. For access to all outlook shared readable calendars in our Active Directory We have specified the following 
        - scope=openid+https://office.outlook.com/Calendars.Read.Shared

__Theory - Key things to remember__
-   Previous versions had the domain `login.windows.net`. Some blogs use this. This is now not exposed, even though, based on inspection of HTTP headers, can be seen to be active in the background.
-   Also note we have two versions of the current endpoint as well - v1.0 and v2.0, and v2.0 has some differences from v1.0, viz., *v2.0 endpoint does not understand the query parameter 'resource' and throws an error*.
-   For multi-tenant apps, replace *tenantId* with *'common'*.
-   The keys given to us from Azure app registration are __*clientId, tenantId, clientSecret*__. However, in the authorize call, the API expects the query parameters as *underscore_separated*, i.e., as __*client_id, tenant_id, and client_secret*__. I got stuck here for a while too.

__Theory - OAuth response__

The authorization takes the required query string parameters, in our case, returns an authentication code. This is one of the possible variations on the request. 

Using OAuth, authentication is redirected to your organizations Windows Active Directory SSO page, so that you can log in with your Windows credentials.  

It then returns the code to the *response_uri* specified. The *code* looks like this 

    QABAAIAAAABlDrqfEFlSaui6xnRjX5Ef_{removed_lots_of_crazy_characters_here}_OSXEQcdFf2RLPAXbz30RgbyAA

#### Theory - Scopes in brief.

Scopes correspond to permissions which the account making the Rest API call will need to successfully interact with Azure resources. In our case, it the application corresponding to the *clientId* which needs the permissions, as we are logging on to our tenant as this client, i.e. we will use this *clientId* during the logon/authorize call.

Roughly speaking, we can think of an Azure Resource equals an domain viz. `office.outlook.com, graph.microsoft.com,` etc. Note that Azure resources can be a lot more granular though. 

The application we registered can request these set of permissions after the logon page is past, 
OR we can pre-allow these permissions to the application while registering. This option is the *OAuth On-Behalf-Of Consent Flow*.

A fuller discussion on scopes can be found at [https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-scopes](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-scopes).

A key thing to remember is the difference between how scopes are provided for Outlook REST APIs at `outlook.office.com` vs. MS Graph at `graph.microsoft.com`. For Outlook - *resource* permissions (permissions for a particular URL) need to be specified in the query parameter in the format `https://url/permissions`. See example below. 

    openid+offline access+https://outlook.office.com/Calendars.Read.Shared+https://outlook.office.com/Contacts.Read

For Graph API, the URL can be omitted, so the query param for MS Graph will be

    openid+offline access+Calendars.Read.Shared+Contacts.Read

We do not need `https://graph.microsoft.com` in the URL's *scope* parameter.

### Theory - Convert Auth Code To Bearer Token

The second part of authentication is in converting the auth code into a JSON Web Token (see `https://jwt.io`). The base tokenization URL is 

    POST https://login.windows.net/common/oauth2/token

The body is of type www-form-encoded and has the following key-value pairs.

    grant_type=authorization_code&code={code from the authorize request}&redirect_uri={reply url for your application}&client_id={your application's client id in AAD}&client_secret={your application's client secret}

The HTTP response is a JSON document of form - 
``` JSON
{
    "token_type": "Bearer",
    "scope": "Calendars.Read Calendars.Read.All Calendars.Read.Shared offline_access"
    "expires_in": "3599",
    "ext_expires_in": "262800",
    "expires_on": "1506082448",
    "not_before": "1506078548",
    "resource": "https://outlook.office.com/",
    "access_token": "{removed-use your own tokens}",
    "refresh_token": "{removed-use your own tokens}",
    "id_token": "{removed-use your own tokens}"
}
```
A access token is as per the JWT specification. Encoded, it looks like this - `eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1-{removed_lots_of_chars_here}-CTN2M3GXpo53GaXko0FgwPFjPA`

However, this contains a lot of information. When decoded (via [JWT.io](http://jwt.io)), we can see the information it contains - 
``` JSON
{
  "aud": "https://outlook.office.com/",
  "iss": "https://sts.windows.net/{tenantId}/",
  "iat": 1506070849,
  "nbf": 1506070849,
  "exp": 1506074749,
  "acr": "1",
  "aio": "Y2VgYHjuJj1PSJv1qLb8h9Y9s3wvRa5NSegLEFNqsHqx9d6LHgcA",
  "amr": [
    "pwd"
  ],
  "appid": "registered_app_id",
  "appidacr": "1",
  "e_exp": 262800,
  "enfpolids": [],
  "family_name": "my_family_name",
  "given_name": "my_first_name",
  "ipaddr": "219.91.160.98",
  "name": "my_full_name",
  "oid": "oid",
  "onprem_sid": "S-1-5-21-1708537768-789336058-725345543-1974208",
  "puid": "1003BFFD870F4BB4",
  "scp": "Calendars.Read Calendars.Read.All Calendars.Read.Shared Contacts.Read Contacts.Read.Shared offline_access",
  "sub": "5M4_YhrPEtd558TCLIWVHGxhgKyxqsctKvtjYpnEx1o",
  "tid": "tenantId",
  "unique_name": "my logon mail id here",
  "upn": "my logon mail id here",
  "ver": "1.0"
}
```

### Theory - MS Office API, REST call.
Most of the nitty-gritties should have been worked out of the way now. We just need to call the Outlook REST API as follows -

    POST https://outlook.office.com/api/{version}/me/findmeetingtimes

Add the following HTTP headers to the request

-   Content-Type : application/json
-   Authorization : bearer *{token_returned_by_token_endpoint}*

**NOTE** : Note the string *bearer*, followed by a space, in front of the token. I struggled for a long time at this step before figuring this out.

And voila. everything works. or does it? Since practise beats theory everytime, let's get down to browsers and POSTMAN.

## Practise - Putting it all together.

Okay, brass-tacking time now.

####   Practise - Add Application into AD.  Save the *tenantId, clientId and clientSecret*.
-   Added a new application called 'Postman' in the linked Azure Active Directory through Azure portal. 
-   Specify 'Read User and Shared Calendars' in the permissions panel, and explicitly granted those permissions. Through this, we are implementing *OAuth On-Behalf-Of-Consent flow*. Otherwise, we would have an intermediate screen after entering our Windows credentials where we would have to accept the use of these permissions. 
-   Add two response_uri, one for Postman REST client to use the response auth code, and one to see the raw http auth request and response in a browser window myself. For Postman I added `https://www.getpostman.com/oauth2/callback`, as per Postman REST client requirements, and a second one, `http://localhost/myapp/` for my browser based calls.

Once done, we keep the following key-values (*not exact, of course*) handy.
```JSON
tenantId : fea858f0-512d-4649-8228-d78fd9ef3c7f
clientId : 794b53a6-e176-4185-92fe-617dd8512db5
clientSecret: MyQt+7mKw/Vz7p6XjHRfBOe68ffWvjmjVhJP69K1dec!=
```
#### Practise - Invoke Azure Authorize API.
The full `GET` request for my use case is as follows. Paste this into a browser window which is not already signed into azure.

    https://login.microsoftonline.com/common/oauth2/authorize?client_id=794b53a6-e176-4185-92fe-617dd8512db5&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F&response_type=code&scope=https://outlook.office.com/calendars.read.shared

Important points to remember
-   Note I have used the endpoint 'common'.
-   Note that I have used v1.0 endpoint. v2.0 did not work for me. The Authorize call succeeds, but the token call fails with a version mismatch error.
-   Note the callback used here 'http://localhost/myapp/'. This was one of the endpoints registered with AD as the call back URI for the 'Postman' application/service principal.
-   Ensure that all URLs mentioned are escaped properly. in the above the *request_uri* is escaped right, but the scope one isn't. It still works, but try not to do this.
-   Ensure that all URLs end with a trailing slash. This one causes a nasty error with an incomprehensible error message.
-   Add the scopes correctly and ensure that the corresponding permissions are granted in Azure portal to the Application.

When the call returns, it shows me a browser page with HTTP 404. That is expected, as the actual URL does not exist. The URL that can be seen in the browser address bar, is like this - 

    http://localhost/myapp/?code=QABAAIAAAABlDrqfEFlSaui6-{removed_lots_of_chars_here}-e3e45d8d4

Note that this is the callback URL specified during application registration. The auth code is appended to the querystring. We will need this code subsequently.

#### Practise - Invoke Azure token API

In POSTMAN, Create a new `POST` request to the following endpoint 

    https://login.microsoftonline.com/common/oauth2/token

Create a request body as type `x-www-form-urlencoded`. In the POSTMAN bulk-edit mode, add the following JSON parameters to it.

    client_id: 794b53a6-e176-4185-92fe-617dd8512db5
    scope: https://outlook.office.com/calendars.read.shared/
    resource: https://outlook.office.com/
    code: QABAAIAAAABlDrq-{removed_lots_of_chars_here}-UassQOSXEQcdFf2RLPAXbz30RgbyAA
    redirect_uri: http://localhost/myapp/
    grant_type: authorization_code
    client_secret: MyQt+7mKw/Vz7p6XjHRfBOe68ffWvjmjVhJP69K1dec!=

Fire the request. We should get the following response.

    {
        "token_type": "Bearer",
        "scope": "Calendars.Read Calendars.Read.All Calendars.Read.Shared Contacts.Read Contacts.Read.Shared offline_access",
        "expires_in": "3599",
        "ext_expires_in": "262800",
        "expires_on": "1506082448",
        "not_before": "1506078548",
        "resource": "https://outlook.office.com/",        
        "access_token": "myJ0eXAiOiJKV1Qi-{removed_lots_of_chars_here}-ukX3nBlRfzh6Sg",        
        "refresh_token": "QABAAAAAAABlDrq-{removed_lots_of_chars_here}-uoBFATkE6ZJHyAA",        
        "id_token": "yJ0eXAiOiJKV1QiL-{removed_lots_of_chars_here}-SIsInZlciI6IjEuMCJ9."
    }

#### Practise - Invoke Outlook REST API. 
Again in POSTMAN, create another `POST` request for the following URI.

    https://outlook.office.com/api/v2.0/me/findmeetingtimes

Add the HTTP headers 

    Content-Type: application/json,
    Authorization: bearer myJ0eXAiOiJKV1-{removed_lots_of_chars_here}-PNukX3nBlRfzh6Sg

__Note__ the space between bearer and the rest of the *access_token* returned by the previous token call.

Add the following as raw request JSON - 
```json
{ 
"Attendees": [ 
    { 
    "Type": "Required",  
    "EmailAddress": { 
        "Name": "my_name",
        "Address": "my_mail_id" 
    } 
    },
{ 
    "Type": "Optional",  
    "EmailAddress": { 
        "Name": "other_name",
        "Address": "other_mail_id" 
    } 
    }  
],  
"TimeConstraint": { 
    "ActivityDomain":"Work",
    "Timeslots": [ 
    { 
        "Start": { 
        "DateTime": "2017-09-22T07:00:00",  
        "TimeZone": "Pacific Standard Time" 
        },  
        "End": { 
        "DateTime": "2017-09-23T17:00:00",  
        "TimeZone": "Pacific Standard Time" 
        } 
    } 
    ] 
},  
"MeetingDuration": "PT1H" 
} 
```

Fire the request.  The outlook API should now return a valid response from Office 365. 

If you want to try out the equivalent graph API, you need to do a fresh authentication. This time use the scopes as required for MS Graph *(no url, just the scope value)*. Then use this Auth Code to get a fresh bearer token. Using this token for a call to the MS Graph API should work. It did work for me without any hassles.

And that's all of it. We can go home now. 

-------------------------------------------------------------------------------------------------------------------


