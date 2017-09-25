--- 
layout: post
title: "Azure AD and Office 365 OAuth Integration via Postman"
author: "Author"
comments: true
tags : [Azure, OAUth, Azure Active Directory, Office 365, Outlook, Postman]
---

I spent last week answering a question. Is there a way to find available meeting times on a given user's office 365 calendar next week? The short answer is **yes**. 

## Longer answer.
At work we have MSDN Azure subscriptions. These are linked to our organization's Azure Active Directory (AAD) and we can sign on to Azure with our windows credentials. We also office 365 accounts linked to our exchange AD and Azure AD. I can seamlessly navigate to office 365 on the browser after logging into Azure portal through my windows account (OAuth magics). If this works for you too, then your Single Sign On (SSO) setup is also done. 

The goal here is to understand how Azure OAuth authentication works when calling the outlook APIs through Azure AD. Preferably, I want to achieve this through raw HTTP calls, using browser windows or Postman REST client, so that HTTP based protocols and interactions are clear and open to visual inspection. In a nutshell, the steps are as follows.

-   Create an Azure AD application / Service principal.
-   Redirect user to exchange Active Directory for Authentication.
-   Returns Auth Code after Azure, through OAUTH, authenticates against organization AD. 
-   Convert OAuth Code into Azure bearer token. 
-   Call Microsoft office APIs in SSO mode using token received above to retrieve the available meeting times for a employee given a specified time window. 

Now, many online resources already exist on this, yet getting all the pieces playing together was rough. So, we'll walk the route I did and hope that the eco-system clears itself up. 

In the process, I will briefly touch on OAuth in Azure, Azure AD, Scopes and Resources in MS Online Api, Azure Service Principals aka App registrations, App permissions aka OAuth on-behalf-of consent flow, Azure bearer tokens in Postman, JWT.io and the Microsoft Graph explorer. Oh! and the Graph and Oulook sandboxes. 

*Note: In Azure, things change. The information here is of 24th Sept 2017.*

*For the rest you need an Azure subscription, an office 365 account, and an Azure AD membership. You can have non work accounts for all three, but its a lot easier if your company admin has done the configuring for you. Blogs are the way to go here if you want to setup trial accounts for each.*

## First, the APIs.

Do such APIs exist? Well, yes. 

Problem is, more than __*one*__ exist.

### Outlook/Office 365 APIs
Documentation link here - [https://msdn.microsoft.com/en-us/office/office365/api/calendar-rest-operations#find-meeting-times](https://msdn.microsoft.com/en-us/office/office365/api/calendar-rest-operations#find-meeting-times).

API looks like this.

    POST https://outlook.office.com/api/{version}/me/findmeetingtimes

And there are *three* *version*s of it

    -   v1.0
    -   v2.0 
    -   beta 

Also note - The first three lines in the relevant section of the link mention that we need at least one of the following scopes. Note these scopes, these are important. We'll see more of them.

    -   https://outlook.office.com/calendars.read.shared
    -   wl.calendars 
    -   wl.contacts_calendars

And the sandbox to play-around with this is here [https://oauthplay.azurewebsites.net/](https://oauthplay.azurewebsites.net/).

### Microsoft Graph APIs
Again, documentation is here - [https://developer.microsoft.com/en-us/graph/docs/concepts/findmeetingtimes_example](https://developer.microsoft.com/en-us/graph/docs/concepts/findmeetingtimes_example).

API looks like this,

    POST https://graph.microsoft.com/v1.0/me/findMeetingTimes

And the sandbox (called MS Graph Explorer), is here - [https://developer.microsoft.com/en-us/graph/graph-explorer](https://developer.microsoft.com/en-us/graph/graph-explorer).

*Note: I wonder if the book 'Don't make me think.' is popular in MS?*

### Outlook vs Graph.

So why two? When to use what? 

Well previously MS had multiple APIs available - Office, Azure AD, etc. MS Graph is intended to unify all of these endpoints into a single consolidated REST gateway for all of Microsoft's underlying platform APIs viz. Azure AD, Excel, Outlook, OneDrive, OneNote, SharePoint etc. It is not fully at par with existing APIs, but MS recommendation is to prefer MS Graph unless you need a feature which it does not have.

For current and future work, prefer MS Graph over other means.

### The Request

In either case the POST body is the same, so that's a little better.

```json
{ 
  "attendees": [ 
    { 
      "type": "required",  
      "emailAddress": { 
        "name": "Samantha Booth",
        "address": "samanthab@contoso.onmicrosoft.com" 
      } 
    }
  ],  
  "locationConstraint": { 
    "isRequired": "false",  
    "suggestLocation": "false",  
    "locations": [ 
      { 
        "resolveAvailability": "false",
        "displayName": "Conf room Hood" 
      } 
    ] 
  },  
  "timeConstraint": {
    "activityDomain":"work", 
    "timeslots": [ 
      { 
        "start": { 
          "dateTime": "2017-04-18T09:00:00",  
          "timeZone": "Pacific Standard Time" 
        },  
        "end": { 
          "dateTime": "2017-04-20T17:00:00",  
          "timeZone": "Pacific Standard Time" 
        } 
      } 
    ] 
  },  
  "meetingDuration": "PT2H",
  "returnSuggestionReasons": "true",
  "minimumAttendeePercentage": "100"
}

```

Endpoints - Check. 
Request Body and Response - Check.
Enter Azure AD and OAuth SSO.

## Azure Active Directory (AAD) and OAuth.

Azure OAuth based authentication is a big and complex topic and cannot be covered adequately in a single blog post. Conceptually, the Azure OAuth flow is like ![this]({{ site.pageUrl }}/img/posts/azureoauth.png). 

*More documentation can be found at [https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-protocols-oauth-code#main](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-protocols-oauth-code#main).*

In our case, for the authentication, we need to do the following -

-   Register an app with Azure AD / Generate Service Principal.
-   Authorize and retrieve authentication code.
-   Convert authentication code into Azure authorization token for use by Outlook calendar APIs.

### App Registration / Generate Service Principal
First, someone needs to get authenticated. This, in Azure terms, is a Service Principal. This is achieved by registering an application with Azure AD, which gives us three keys -*tenantId, clientId, clientSecret*- at the end. 

-   Added a new application called 'Postman' in the linked Azure Active Directory. This is our work AD.
-   Specified 'Read User and Shared Calendars' in the permissions panel, and explicitly granted those permissions. 
-   Added two response_uri, one for Postman REST client to use the response auth code, and one to see the raw http auth request and response in a browser window myself. In app registration, I specified these two callback urls as 
    
    -   https://www.getpostman.com/oauth2/callback 
        -   *(As per Postman REST client requirements)*
    -   http://localhost/myapp/
        -   *(For my in-browser call-backs)*

*For details about how to register an app, see  [https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-integrating-applications](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-integrating-applications).*

After registration, we save the following keys for subsequent use.
-   *tenantId*: guid looking code of the AD instance.
-   *clientId*: guid looking code of the application being registered.
-   *clientSecret*: guid looking key corresponding to a code we create in the application properties on Azure. 

*Note : The clientSecret will be shown only once when creating a app property. If you don't note it down, you need to delete and recreate a key. It canot be recovered.*

### OAUTH Authorize via Azure AD
The OAuth dance is a two-step process here. Get a authentication code from the underlying authentication provider (OpenId, Active Directory), and then convert that code into a JSON Web Token (see specs at [JWT](https://jwt.io)). For subsequent calls, use this token as the Authorization header.

#### Get Authentication Code / Authorize API
The following are the current (as-of-date) logon authorization endpoints.

*(Note: Previous versions had the domain login.windows.net. Also note we have two versions of the current endpoint as well - v1.0 and v2.0)*

    v1.0 - https://login.microsoftonline.com/{tenantId}/oauth2/authorize 
    v2.0 - https://login.microsoftonline.com/{tenantId/oauth2/v2.0/authorize

For an example, a complete logon request with query parameters is as follows - 

    GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=CLIENT_ID&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F&response_type=code&scope=openid+https://office.outlook.com/Calendars.Read.Shared

The following are the Query Params - 

    -   client_id: the clientId generated before during registering the app. This lets Azure know which app/service principal is requesting the logon.
    -   redirect_uri: the location that Azure will redirect to once the user has granted consent to the app. This value must correspond to the value of Redirect URI used when *registering the app*.
    -   response_type: the type of response the app is expecting. For the Authorization Grant Flow, this should always be the string *code*.
    -   scope: a space-delimited list of access scopes that the app requires. For access to all outlook shared readable calendars in our Active Directory We have specified the following 
        -   scope=openid+https://office.outlook.com/Calendars.Read.Shared

Key things to remember - 

-   For multi-tenant apps, replace *tenantId* with '*common*'.
-   The v2.0 endpoint does not understand the query parameter resource and throws an error, the first versions does not. *(Note - not sure about this, need to check)*.
-   The keys given to us from Azure app registration are __*clientId, tenantId, clientSecret*__. However, in the authorize call, the API expects the query parameters as *underscore_separated*, i.e., as __*client_id, tenant_id, and client_secret*__. I got stuck here for a while too.

The authorization takes the required query string parameters, in our case, returns an authentication code. This is one of the possible variations on the request. 

Using OAuth, authentication is redirected to your organizations windows active directory SSO page, so that you can log in with your windows credentials.  

It then returns the code to the *response_uri* specified. The *code* looks like this 

```batch
QABAAIAAAABlDrqfEFlSaui6xnRjX5Ef1Ul7-oG7KBrwigC8DAjBC0nxZdQznVfSBuLT7kuLY0D2CdcRC3J4Mz7neJRhmGBNSE6NQuGEzC2CNeEcr8TBO7szAZW9_gU8yUc9mGwupdc-pq_WRNDBVQVjnElgWlWszPAjIsBCWphvMdGGsAXu_QeJNtcDA1pi9eaDdgElri94ZBj2emnE9eCOOZIXkKqgiL4xCSNwYUPiZFRSuZSXUS-X5ZB6_RGSbv8s0Avh09frjYHqmIuM_4cb5tDHGinl4wwlxqb_rHoHh1PqJYPCdTss8-fnT60JwvpnM0JeywsqEZAl0Og6cg0yEhuKZ0fNt-NCMHmEAC8DoKzGrd28HbXtz9WJS1yo_KbtPBosyvFixyuqTQJD_UhM2xtaDP1V7BO2SVv_vosBAEsOoEIMbgiYs9SjVT9Ywe_Q9OWnNhthY6l44ttk2Z8ozlClzMDRcfJuOdxDBMLTvbqDpNcGnx_NdJ8hOWGe1pyJOu-6PthlOLdpPHUYFkvzz7_riHp_3Kf_o7Fa-YIr1uMrYj9C-0pvYicDZL7CIxaTJkQ8vaJH0xEUHqkm3aV1ZoTTZRMlSoziVyXHadlcFo51afFW5OhzgCcxR_Y-q3eSaZq7u_Bc4SzHHDRlbk3liUassQOSXEQcdFf2RLPAXbz30RgbyAA
```
*(Token is not accurate and wont work in JWT.io. Have removed 1 character)*.

#### A note on Scopes.
Scopes correspond to permissions which the account making the Rest API call will need to successfully interact with Azure resources. In our case, it the application corresponding to the *clientId* which needs the permissions, as we are logging on to our tenant as this client, i.e. we will use this *clientId* during the logon/authorize call.

Roughly speaking, we can think of an Azure Resource equals an domain viz. office.outlook.com, graph.microsoft.com, etc. Note that Azure resources can be a lot more granular though. 

The application we registered can request these set of permissions after the logon page is past, 
OR we can pre-allow these permissions to the application while registering. This option is the *OAuth On-Behalf-Of Consent Flow*.

A fuller discussion on scopes can be found at [https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-scopes](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-scopes).

A key thing to remember is the difference between how scopes are stated for Outlook REST APIs (outlook.office.com) vs. MS Graph (graph.microsoft.com). For Outlook - resource level permissions (permissions for a prticular URL) need to be specified in the query parameter in the format {url/permission} - 

    openid+offline access+https://outlook.office.com/Calendars.Read.Shared+https://outlook.office.com/Contacts.Read

For Graph API, the URL can be omitted, so the query param for MS Graph will be

    openid+offline access+Calendars.Read.Shared+Contacts.Read

Here even though we may have asked for permissions to the Graph API at https://graph.microsoft.com, we dont need to add it in the URL.

### Convert Auth Code To Bearer Token

The second part of authentication is in converting the received token into a JWT specs based azure access token. 

The base tokenization URL is:

    POST https://login.windows.net/common/oauth2/token

The body is form encoded:

    grant_type=authorization_code&code={code from the authorize request}&redirect_uri={reply url for your application}&client_id={your application's client id in AAD}&client_secret={your application's client secret}

The result is a JSON document of form - 

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

A access token is as per the JWT specification. Encoded, it looks like this - 

    eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IkhIQnlLVS0wRHFBcU1aaDZaRlBkMlZXYU90ZyIsImtpZCI6IkhIQnlLVS0wRHFBcU1aaDZaRlBkMlZXYU90ZyJ9.eyJhdWQiOiJodHRwczovL291dGxvb2sub2ZmaWNlLmNvbS8iLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC9mZWE4NThmMC01MTJkLTQ2NDktODIyOC1kNzhmZDllZjNjN2UvIiwiaWF0IjoxNTA2MDcwODQ5LCJuYmYiOjE1MDYwNzA4NDksImV4cCI6MTUwNjA3NDc0OSwiYWNyIjoiMSIsImFpbyI6IlkyVmdZSGp1SmoxUFNKdjFxTGI4aDlZOXMzd3ZSYTVOU2VnTEVGTnFzSHF4OWQ2TEhnY0EiLCJhbXIiOlsicHdkIl0sImFwcGlkIjoiNzk0YjUzYTYtZTE3Ni00MTg1LTkyZmUtNjE3ZGQ4NTEyZGEwIiwiYXBwaWRhY3IiOiIxIiwiZV9leHAiOjI2MjgwMCwiZW5mcG9saWRzIjpbXSwiZmFtaWx5X25hbWUiOiJTZW5ndXB0YSIsImdpdmVuX25hbWUiOiJJbmRyYW5pbCIsImlwYWRkciI6IjIxOS45MS4xNjAuOTgiLCJuYW1lIjoiU2VuZ3VwdGEsIEluZHJhbmlsIiwib2lkIjoiNTFhYmU5MWUtMzZlZS00ZGUyLTg0OTUtYWRlMjlkYjAwYjQyIiwib25wcmVtX3NpZCI6IlMtMS01LTIxLTE3MDg1Mzc3NjgtNzg5MzM2MDU4LTcyNTM0NTU0My0xOTc0MjA4IiwicHVpZCI6IjEwMDNCRkZEODcwRjRCQjQiLCJzY3AiOiJDYWxlbmRhcnMuUmVhZCBDYWxlbmRhcnMuUmVhZC5BbGwgQ2FsZW5kYXJzLlJlYWQuU2hhcmVkIENvbnRhY3RzLlJlYWQgQ29udGFjdHMuUmVhZC5TaGFyZWQgb2ZmbGluZV9hY2Nlc3MiLCJzdWIiOiI1TTRfWWhyUEV0ZDU1OFRDTElXVkhHeGhnS3l4cXNjdEt2dGpZcG5FeDFvIiwidGlkIjoiZmVhODU4ZjAtNTEyZC00NjQ5LTgyMjgtZDc4ZmQ5ZWYzYzdlIiwidW5pcXVlX25hbWUiOiJJU2VuZ3VwdGFAc3BpZGVybG9naWMuY29tIiwidXBuIjoiSVNlbmd1cHRhQHNwaWRlcmxvZ2ljLmNvbSIsInZlciI6IjEuMCJ9.FBbuguQOmDFp5Jm5BLE-HgRrAs640eEBJltm_uesld6J950MBlol-IVCAYejuQaYIRj96EICbb2GMkD6O2cEcz-ypFCu_E0yn-VnixM2VsXs5vy45biowPO85xdWQXxqyYm7A94z7nc-jnMvy7LFa1e8I_C2AonmhYXD7ckMiw9OMhFYs9r2steeWuIihCgOUaLGjOTHU-2-uTn5Ce4xcuMtPrx8pynreWoBK2Ry0XZhGKMKMFnYKwoN3KA9285eCquOxctf3f6w67LvO_XJXz-y-vlA16ifAL1mlE6FgnG8jpAtaoR-HdXI-eWFCTN2M3GXpo53GaXko0FgwPFjPA

However, this contains a lot of information. When decoded (via [JWT.io](http://jwt.io)), we can see the information it contains - 
``` JSON
{
  "aud": "https://outlook.office.com/",
  "iss": "https://sts.windows.net/fea858f0-512d-4649-8228-d78fd9ef3c7e/",
  "iat": 1506070849,
  "nbf": 1506070849,
  "exp": 1506074749,
  "acr": "1",
  "aio": "Y2VgYHjuJj1PSJv1qLb8h9Y9s3wvRa5NSegLEFNqsHqx9d6LHgcA",
  "amr": [
    "pwd"
  ],
  "appid": "794b53a6-e176-4185-92fe-617dd8512da0",
  "appidacr": "1",
  "e_exp": 262800,
  "enfpolids": [],
  "family_name": "my_family_name",
  "given_name": "my_first_name",
  "ipaddr": "219.91.160.98",
  "name": "my_full_name",
  "oid": "51abe91e-36ee-4de2-8495-ade29db00b42",
  "onprem_sid": "S-1-5-21-1708537768-789336058-725345543-1974208",
  "puid": "1003BFFD870F4BB4",
  "scp": "Calendars.Read Calendars.Read.All Calendars.Read.Shared Contacts.Read Contacts.Read.Shared offline_access",
  "sub": "5M4_YhrPEtd558TCLIWVHGxhgKyxqsctKvtjYpnEx1o",
  "tid": "fea858f0-512d-4649-8228-d78fd9ef3c7e",
  "unique_name": "my logon mail id here",
  "upn": "my logon mail id here",
  "ver": "1.0"
}
```

### MS Office API, REST call.
Most of the nitty-gritties should have been worked out of the way now. We just need to call the Outlook REST API as follows -

    POST https://outlook.office.com/api/{version}/me/findmeetingtimes

Add the following HTTP headers to the request

-   Content-Type : application/json
-   Authorization : bearer *{token_returned_by_token_endpoint}*

**NOTE** : Note the string *bearer*, followed by a space, in front of the token. I struggled for a long time at this step before figuring this out.

And voila. everything works. or does it? Since practise beats theory everytime, let's get down to browsers and POSTMAN.

## Putting it all together.

Okay, lets get down to it.

####   Add Application into AD. Save the *tenantId, clientId and clientSecret*.

My Application is named 'Postman'. My key-values are as follows - 

```JSON
tenantId : fea858f0-512d-4649-8228-d78fd9ef3c7f
clientId : 794b53a6-e176-4185-92fe-617dd8512db5
clientSecret: MyQt+7mKw/Vz7p6XjHRfBOe68ffWvjmjVhJP69K1dec!=

```

#### Invoke Azure Authorize API.

The full GET request for my use case is as follows. Paste this into a browser window which is not already signed into azure - 

    https://login.microsoftonline.com/common/oauth2/authorize?client_id=794b53a6-e176-4185-92fe-617dd8512db5&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F&response_type=code&scope=https://outlook.office.com/calendars.read.shared

Important points to remember
-   Note I have used the endpoint 'common'.
-   Note that I have used v1.0 endpoint. v2.0 did not work for me. The Authorize call succeeds, but the token call fails with a version mismatch error.
-   Note the callback used here 'http://localhost/myapp/'. This was one of the endpoints registered with AD as the call back URI for the 'Postman' application/service principal.
-   Ensure that all URLs mentioned are escaped properly. in the above the *request_uri* is escaped right, but the scope one isn't. It still works, but try not to do this.
-   Ensure that all URLs end with a trailing slash. This one causes a nasty error with an incomprehensible error message.
-   Add the scopes correctly and ecnsure that the corresponding permissions are granted in Azure portal to the Application.

When the call returns, it shows me a browser page with HTTP 404. That is expected, as the actual URL does not exist. The URL that can be seen in the browser address bar, is like this - 

    http://localhost/myapp/?code=QABAAIAAAABlDrqfEFlSaui6xnRjX5Ef1Ul7-oG7KBrwigC8DAjBC0nxZdQznVfSBuLT7kuLY0D2CdcRC3J4Mz7neJRhmGBNSE6NQuGEzC2CNeEcr8TBO7szAZW9_gU8yUc9mGwupdc-pq_WRNDBVQVjnElgWlWszPAjIsBCWphvMdGGsAXu_QeJNtcDA1pi9eaDdgElri94ZBj2emnE9eCOOZIXkKqgiL4xCSNwYUPiZFRSuZSXUS-X5ZB6_RGSbv8s0Avh09frjYHqmIuM_4cb5tDHGinl4wwlxqb_rHoHh1PqJYPCdTss8-fnT60JwvpnM0JeywsqEZAl0Og6cg0yEhuKZ0fNt-NCMHmEAC8DoKzGrd28HbXtz9WJS1yo_KbtPBosyvFixyuqTQJD_UhM2xtaDP1V7BO2SVv_vosBAEsOoEIMbgiYs9SjVT9Ywe_Q9OWnNhthY6l44ttk2Z8ozlClzMDRcfJuOdxDBMLTvbqDpNcGnx_NdJ8hOWGe1pyJOu-6PthlOLdpPHUYFkvzz7_riHp_3Kf_o7Fa-YIr1uMrYj9C-0pvYicDZL7CIxaTJkQ8vaJH0xEUHqkm3aV1ZoTTZRMlSoziVyXHadlcFo51afFW5OhzgCcxR_Y-q3eSaZq7u_Bc4SzHHDRlbk3liUassQOSXEQcdFf2RLPAXbz30RgbyAA&session_state=f4db04de-60f7-426b-a599-48be3e45d8d4

Note that this is the callback URL specified during App Registration with the auth code appended in the querystring. We will need this code subsequently.

#### Invoke Azure token API

In POSTMAN, Create a new POST request to the following endpoint 

    https://login.microsoftonline.com/common/oauth2/token

Create a request body as type 'x-www-form-urlencoded'. In the POSTMAN bulk-edit mode, add the following JSON parameters to it.

    client_id: 794b53a6-e176-4185-92fe-617dd8512db5
    scope: https://outlook.office.com/calendars.read.shared/
    resource: https://outlook.office.com/
    code: QABAAIAAAABlDrqfEFlSaui6xnRjX5Ef1Ul7-oG7KBrwigC8DAjBC0nxZdQznVfSBuLT7kuLY0D2CdcRC3J4Mz7neJRhmGBNSE6NQuGEzC2CNeEcr8TBO7szAZW9_gU8yUc9mGwupdc-pq_WRNDBVQVjnElgWlWszPAjIsBCWphvMdGGsAXu_QeJNtcDA1pi9eaDdgElri94ZBj2emnE9eCOOZIXkKqgiL4xCSNwYUPiZFRSuZSXUS-X5ZB6_RGSbv8s0Avh09frjYHqmIuM_4cb5tDHGinl4wwlxqb_rHoHh1PqJYPCdTss8-fnT60JwvpnM0JeywsqEZAl0Og6cg0yEhuKZ0fNt-NCMHmEAC8DoKzGrd28HbXtz9WJS1yo_KbtPBosyvFixyuqTQJD_UhM2xtaDP1V7BO2SVv_vosBAEsOoEIMbgiYs9SjVT9Ywe_Q9OWnNhthY6l44ttk2Z8ozlClzMDRcfJuOdxDBMLTvbqDpNcGnx_NdJ8hOWGe1pyJOu-6PthlOLdpPHUYFkvzz7_riHp_3Kf_o7Fa-YIr1uMrYj9C-0pvYicDZL7CIxaTJkQ8vaJH0xEUHqkm3aV1ZoTTZRMlSoziVyXHadlcFo51afFW5OhzgCcxR_Y-q3eSaZq7u_Bc4SzHHDRlbk3liUassQOSXEQcdFf2RLPAXbz30RgbyAA
    redirect_uri: http://localhost/myapp/
    grant_type: authorization_code
    client_secret: MyQt+7mKw/Vz7p6XjHRfBOe68ffWvjmjVhJP69K1dec!=

Fire the request. We should get the following response (*tokens are edited. Not valid*) -

    {
        "token_type": "Bearer",
        "scope": "Calendars.Read Calendars.Read.All Calendars.Read.Shared Contacts.Read Contacts.Read.Shared offline_access",
        "expires_in": "3599",
        "ext_expires_in": "262800",
        "expires_on": "1506082448",
        "not_before": "1506078548",
        "resource": "https://outlook.office.com/",
        
        "access_token": "myJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IkhIQnlLVS0wRHFBcU1aaDZaRlBkMlZXYU90ZyIsImtpZCI6IkhIQnlLVS0wRHFBcU1aaDZaRlBkMlZXYU90ZyJ9.myJhdWQiOiJodHRwczovL291dGxvb2sub2ZmaWNlLmNvbS8iLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC9mZWE4NThmMC01MTJkLTQ2NDktODIyOC1kNzhmZDllZjNjN2UvIiwiaWF0IjoxNTA2MDc4NTQ4LCJuYmYiOjE1MDYwNzg1NDgsImV4cCI6MTUwNjA4MjQ0OCwiYWNyIjoiMSIsImFpbyI6IlkyVmdZSkMxZHpxOGhmM1NJcjRvbGVqSFU2NHFYWTM1dDdtMFlQVzVxSGhXYzNrSjNxOEEiLCJhbXIiOlsicHdkIl0sImFwcGlkIjoiNzk0YjUzYTYtZTE3Ni00MTg1LTkyZmUtNjE3ZGQ4NTEyZGEwIiwiYXBwaWRhY3IiOiIxIiwiZV9leHAiOjI2MjgwMCwiZW5mcG9saWRzIjpbXSwiZmFtaWx5X25hbWUiOiJTZW5ndXB0YSIsImdpdmVuX25hbWUiOiJJbmRyYW5pbCIsImlwYWRkciI6IjIxOS45MS4xNjAuOTgiLCJuYW1lIjoiU2VuZ3VwdGEsIEluZHJhbmlsIiwib2lkIjoiNTFhYmU5MWUtMzZlZS00ZGUyLTg0OTUtYWRlMjlkYjAwYjQyIiwib25wcmVtX3NpZCI6IlMtMS01LTIxLTE3MDg1Mzc3NjgtNzg5MzM2MDU4LTcyNTM0NTU0My0xOTc0MjA4IiwicHVpZCI6IjEwMDNCRkZEODcwRjRCQjQiLCJzY3AiOiJDYWxlbmRhcnMuUmVhZCBDYWxlbmRhcnMuUmVhZC5BbGwgQ2FsZW5kYXJzLlJlYWQuU2hhcmVkIENvbnRhY3RzLlJlYWQgQ29udGFjdHMuUmVhZC5TaGFyZWQgb2ZmbGluZV9hY2Nlc3MiLCJzdWIiOiI1TTRfWWhyUEV0ZDU1OFRDTElXVkhHeGhnS3l4cXNjdEt2dGpZcG5FeDFvIiwidGlkIjoiZmVhODU4ZjAtNTEyZC00NjQ5LTgyMjgtZDc4ZmQ5ZWYzYzdlIiwidW5pcXVlX25hbWUiOiJJU2VuZ3VwdGFAc3BpZGVybG9naWMuY29tIiwidXBuIjoiSVNlbmd1cHRhQHNwaWRlcmxvZ2ljLmNvbSIsInZlciI6IjEuMCJ9.lnN62jNPXBGodcrRYSWKyPkFxG4dXu200S2WAD4ud_zb_fQJS-duItEFx_CbTLaSnUbh2zb5bQfYwJJ2Ur-iWH7XPrmraf0UqugpTsT6pwBxhz1DXwiucasT6VK-xifEFBa95LZbW3gURuOuJ1ViQXPszuIaQN9OAwSSjV09FQfSOdTF9JoPQfv-t7_DaTgNCTREWIDWgT0VMFalsuz-rrid845RioXCa-QwTzw2uFo9Gv3mcEcuHAU_b3-04VtcXIaXi6p3zaZJdToVKQ433eo55cWX-8xUB1XWmttAG0Rr8M97rcsMfqII701ZBGnBLE_oX1PNukX3nBlRfzh6Sg",
        
        "refresh_token": "QABAAAAAAABlDrqfEFlSaui6xnRjX5EEyM3jOuyerkOIS68ew5lWXOgIkUIbI0-kEfeP1G9tNWsuGmvyZmNHaTSWUsg315AdmsPGBDhS9FtRVTuUSY1lHSVTzWQWaUF-XlEWAcCgKC7XQlKtbmnjuwzZtKMj-Nc5s4qo5S4waeu7IOUZzlvD_R-E2YaC2aA5yEf_ule2mgcvfOiUOZVujNRxrxUzoP497W4H4-VpVfxlA7o5athBpdlqQYCvq0K8E_GhoKo1UYSDAdBiLUrRZ6qTV2SqOpdxQTY6WEiSbKv_Q3E1a8C0eLQlndV54K0Epuk2PhGJKEXeNRmoLsyHSVGqOvKG9jdHBx3Aco1iKBOPbPzpaP9i1xE6v7xgOrUtAOdi98q22RzaT7Zxl4rtVMDhqn_bhtNSZpThTJ1oKCJ6WCKq3zQ9l2iBsN8ld9kTVcVq02wLSubVYi3ubxiPVDyZ5C524jodTkkrAVdhdAPJ6YNN-0Y5WQk-FfY5I-iBy_v-M6YAYj86ZuQs1OB2nTNVlo7snlUTWWFtXVJafW6eTFhV_Q8rFUz3NkkpVYa92Vyw-bsMjIB_GaVXunbYNWT0Ks72JJNelNwokVqtfb926Cj0JhP2TJHQ2s7ziP4TLI2LoJoAC2PTHGVSQ6QVFicbR1DDPpiBnp-0JwmGcWX1dYFHNLmceG2KroXprLLjx7DZGNCjPYD12mgHE4QROMt1RDpiP4haGf86_f_r6rfQvnUGvD0gpL2nKpn5znjAFhMZQ6JXi0BKI4zeGvmmURD-8uJXFgTS3UdK4OBLicg1Rv1sAA-kMlRu1FoGzajO9Od6mdk8_NUQDMgJ3oq9KdtmBwGokeqOGHicL2tzuoBFATkE6ZJHyAA",
        
        "id_token": "yJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiI3OTRiNTNhNi1lMTc2LTQxODUtOTJmZS02MTdkZDg1MTJkYTAiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC9mZWE4NThmMC01MTJkLTQ2NDktODIyOC1kNzhmZDllZjNjN2UvIiwiaWF0IjoxNTA2MDc4NTQ4LCJuYmYiOjE1MDYwNzg1NDgsImV4cCI6MTUwNjA4MjQ0OCwiYW1yIjpbInB3ZCJdLCJmYW1pbHlfbmFtZSI6IlNlbmd1cHRhIiwiZ2l2ZW5fbmFtZSI6IkluZHJhbmlsIiwiaXBhZGRyIjoiMjE5LjkxLjE2MC45OCIsIm5hbWUiOiJTZW5ndXB0YSwgSW5kcmFuaWwiLCJvaWQiOiI1MWFiZTkxZS0zNmVlLTRkZTItODQ5NS1hZGUyOWRiMDBiNDIiLCJvbnByZW1fc2lkIjoiUy0xLTUtMjEtMTcwODUzNzc2OC03ODkzMzYwNTgtNzI1MzQ1NTQzLTE5NzQyMDgiLCJzdWIiOiJ6eW5Ob084bkxfMmY3bWhLaWo4Y3R4RlBXczlTMW0yZ3d0VDZhYjJuX1BzIiwidGlkIjoiZmVhODU4ZjAtNTEyZC00NjQ5LTgyMjgtZDc4ZmQ5ZWYzYzdlIiwidW5pcXVlX25hbWUiOiJJU2VuZ3VwdGFAc3BpZGVybG9naWMuY29tIiwidXBuIjoiSVNlbmd1cHRhQHNwaWRlcmxvZ2ljLmNvbSIsInZlciI6IjEuMCJ9."
    }

#### Invoke Outlook REST API.

Again in POSTMAN, craft another HTTP POST request for the following URI.

    https://outlook.office.com/api/v2.0/me/findmeetingtimes

Add the HTTP headers 

    Content-Type: application/json,
    Authorization: bearer myJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IkhIQnlLVS0wRHFBcU1aaDZaRlBkMlZXYU90ZyIsImtpZCI6IkhIQnlLVS0wRHFBcU1aaDZaRlBkMlZXYU90ZyJ9.myJhdWQiOiJodHRwczovL291dGxvb2sub2ZmaWNlLmNvbS8iLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC9mZWE4NThmMC01MTJkLTQ2NDktODIyOC1kNzhmZDllZjNjN2UvIiwiaWF0IjoxNTA2MDc4NTQ4LCJuYmYiOjE1MDYwNzg1NDgsImV4cCI6MTUwNjA4MjQ0OCwiYWNyIjoiMSIsImFpbyI6IlkyVmdZSkMxZHpxOGhmM1NJcjRvbGVqSFU2NHFYWTM1dDdtMFlQVzVxSGhXYzNrSjNxOEEiLCJhbXIiOlsicHdkIl0sImFwcGlkIjoiNzk0YjUzYTYtZTE3Ni00MTg1LTkyZmUtNjE3ZGQ4NTEyZGEwIiwiYXBwaWRhY3IiOiIxIiwiZV9leHAiOjI2MjgwMCwiZW5mcG9saWRzIjpbXSwiZmFtaWx5X25hbWUiOiJTZW5ndXB0YSIsImdpdmVuX25hbWUiOiJJbmRyYW5pbCIsImlwYWRkciI6IjIxOS45MS4xNjAuOTgiLCJuYW1lIjoiU2VuZ3VwdGEsIEluZHJhbmlsIiwib2lkIjoiNTFhYmU5MWUtMzZlZS00ZGUyLTg0OTUtYWRlMjlkYjAwYjQyIiwib25wcmVtX3NpZCI6IlMtMS01LTIxLTE3MDg1Mzc3NjgtNzg5MzM2MDU4LTcyNTM0NTU0My0xOTc0MjA4IiwicHVpZCI6IjEwMDNCRkZEODcwRjRCQjQiLCJzY3AiOiJDYWxlbmRhcnMuUmVhZCBDYWxlbmRhcnMuUmVhZC5BbGwgQ2FsZW5kYXJzLlJlYWQuU2hhcmVkIENvbnRhY3RzLlJlYWQgQ29udGFjdHMuUmVhZC5TaGFyZWQgb2ZmbGluZV9hY2Nlc3MiLCJzdWIiOiI1TTRfWWhyUEV0ZDU1OFRDTElXVkhHeGhnS3l4cXNjdEt2dGpZcG5FeDFvIiwidGlkIjoiZmVhODU4ZjAtNTEyZC00NjQ5LTgyMjgtZDc4ZmQ5ZWYzYzdlIiwidW5pcXVlX25hbWUiOiJJU2VuZ3VwdGFAc3BpZGVybG9naWMuY29tIiwidXBuIjoiSVNlbmd1cHRhQHNwaWRlcmxvZ2ljLmNvbSIsInZlciI6IjEuMCJ9.lnN62jNPXBGodcrRYSWKyPkFxG4dXu200S2WAD4ud_zb_fQJS-duItEFx_CbTLaSnUbh2zb5bQfYwJJ2Ur-iWH7XPrmraf0UqugpTsT6pwBxhz1DXwiucasT6VK-xifEFBa95LZbW3gURuOuJ1ViQXPszuIaQN9OAwSSjV09FQfSOdTF9JoPQfv-t7_DaTgNCTREWIDWgT0VMFalsuz-rrid845RioXCa-QwTzw2uFo9Gv3mcEcuHAU_b3-04VtcXIaXi6p3zaZJdToVKQ433eo55cWX-8xUB1XWmttAG0Rr8M97rcsMfqII701ZBGnBLE_oX1PNukX3nBlRfzh6Sg

Note the space between bearer and the rest of the *access_token* returned by the previous token call.

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

Fire the request, we should see outlook API return a valid response against your outlook API. Worked for me. Hope it works for you too. 

Well, that's the whole of it. Hope this helps, and all the best.

-------------------------------------------------------------------------------------------------------------------


