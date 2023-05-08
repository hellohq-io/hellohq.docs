# Hi, nice to see you on hellohq!

This is the developer guide for the HQ API, which allows developers access to a
wide range of entities and business logic of the HQ.

To ask questions or report issues with our API, please head over to [Issues](https://github.com/hellohq-io/hellohq.docs/issues).

# Introduction

The HQ API allows access to your data. To do so, it follows the [OData 4.0](http://docs.oasis-open.org/odata/odata/v4.0/odata-v4.0-part1-protocol.html)
standard for retrieving, creating, updating and deleting entities through RESTful 
HTTP requests.

**Base URL:** `https://api.hqlabs.de`

**Before you get started, you need to register your client application in the HQ
admin panel.**

# Authentication

The HQ API uses [OAuth 2.0](https://oauth.net/2/) for authentication.

## Getting Started

OAuth 2.0 is an authentication framework used for secure user authentication.
With the typical OAuth 2.0 authentication flows, third-party applications do not
receive the user's password but only a token which is valid for a limited time.

To uniquely identify the client, each client application needs to be registered
first. The client receives an `App Id` and `App Secret` which they use to
authenticate against the API.

When a user wants to use the client application, the user should be redirected to our
login page to provide username and password. After a successful login, the client
application receives an `Access Token` and `Refresh Token`. These can  be used to
authenticate against the API. From then on, the user is no longer involved in the
authentication process.

### Client Application

A client application needs to be registered in your HQ. To do so, go to the
administration panel and add a new client application. You will be asked to
provide a unique name, and a display name for your client.

### App Id

The `App Id` used for OAuth 2.0 consists of your HQ customer id as well as the id of the
client application you created in the HQ: `{customer-id}-{client-id}`. For
example, `12345-clientapp`, where `12345` is your customer id and `clientapp` is the
id of the client. After you registered a client application in your HQ, you can
find the `App Id` in the list in the administration panel.

### App Secret

The `App Secret` is generated in the administration section of your HQ. It is located
next to the `App Id` in the administration panel list. The secret will be used to
authenticate your client application when requesting a token.

### Scopes

The OAuth 2.0 authentication flow uses scopes to define which rights are granted to the
application by the user. The HQ API currently supports the following scopes:

- `read_all`: read access to all resources
- `write_all`: write access to all resources

### Access Token

The `Access Token` is used to authenticate against the API resources. It needs to
be included in every request to the API. Each user must use their own unique `Access Token`,
since such tokens are only valid for the associated user. The toke is usually valid for a few
days only.

### Refresh Token

The `Refresh Token` is used to get a new `Access Token`, once it has expired. A
`Refresh Token` only expires when the user manually revokes access for the client
application.

### OAuth Endpoints

The OAuth endpoints are required to get an `Access Token` and exchange a `Refresh
Token` for a new `Access Token`:

- `/Account/Authorize` may be used to initially retrieve an authorization code
- `/Token` may be used to retrieve an authorization code for an `Access Token`
or to get a new `Access Token` in combination with the `Refresh Token`

### Authorization Flow

The OAuth 2.0 Authorization `Code Grant` is the default authorization flow and
mainly used by client applications. It is described here in detail:

#### Authorization Request

The client constructs the request URI by adding the following parameters to the
query string of the authorization endpoint URI using the
`application/x-www-form-urlencoded` format. Then, the client directs the user to
the constructed URI using a browser window. The user is asked to log in,
provides their username and password, and grants the requested permissions to your client
application.

**Parameter:**

- `ClientId`: the client id of the authentication client
- `State`: an arbitrary state string
- `RedirectUri`: the URI to redirect the browser to after the user granted the
access
- `Scope`: a space-separated list of API scopes

**Example:**
```
curl --request GET \
  --url 'https://api.hqlabs.de/Account/Authorize
  ?response_type=code
  &client_id=1234-testapp => 'From hellHQ UI'
  &state=xyz
  &redirect_uri=https%3A%2F%2Flocalhost%3A8090
  &scope=read_all%20write_all'
```

**Note:** the generated URL needs to be opened in a browser window so that the
user can log in and authorize the application.

All query parameters (especially the `RedirectUri`) should be properly URL-encoded.

If the user is already granted the same scopes to the client application, the
UI will not be rendered and the redirect with the authorization code happens
automatically.

#### User Authentication

The user logs in, and grants or revokes the access request.

#### Authorization Response

If the user grants the access request, the authorization server issues an
authorization code and delivers it to the client by adding the following
parameters to the query string of the redirection URI using the
`application/x-www-form-urlencoded` format.

**Parameter:**

- `RedirectUri`: the previously specified redirect URI
- `AuthenticationCode`: the authentication code that can be exchanged for a token
- `State`: an arbitrary state string

**Example:**

`302 Found`

```
https://client.example.com/cb?code=MWG5HTnFn9n8HJ&state=xyz
```

#### Access Token Request

The client makes a request to the token endpoint by sending the following
parameters in the request body using the `application/x-www-form-urlencoded` format.

**Parameter:**

- `AuthenticationCode`: the code that was sent in the previous response
- `RedirectUri`: the previously specified redirect URI
- `Authorization`: the [HTTP Authorization header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization)

**Note:** To construct a proper HTTP Authorization header for [Basic Access Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication), you need
to encode your `App Id` and `App Secret` using [Base64](https://en.wikipedia.org/wiki/Base64), and add it
to the `Authorization` header as follows: `Authorization: Basic
Base64({AppId}:{AppSecret})`

**Example:**

```
curl --request POST \
  --url https://api.hqlabs.de/Token \
  --header 'Content-Type: application/x-www-form-urlencoded' \
  --data client_id=1234-testapp =>'From hellHQ UI' \
  --data 'scope=read_all write_all' \
  --data redirect_uri=https://client.example.com/cb \
  --data grant_type=authorization_code \
  --data client_secret=fkgjqoer9tfiealdkcmakwhdf => 'From hellHQ UI' \
  --data code='Code from authorize call'
```

**Note:** All query parameters (especially the `RedirectUri`) should be properly URL-encoded.

#### Access Token Response

If the access token request is valid and authorized, the authorization server
issues an Access Token and Refresh Token. If the request failed or is invalid,
the authorization server returns an error response.

**Parameter:**

- `AuthenticationCode`: the code that was sent in the previous response
- `RedirectUri`: the previously specified redirect uri

**Example:**

```json
{
   "access_token": "FGDoJBgK96Z...",  // the Access Token for authorization
   "token_type": "bearer",            // the token type, usually bearer
   "expires_in": 1199,                // the expiration timespan, in seconds
   "refresh_token": "K8vma4VohMb...", // the Refresh Token to request a new Access Token
   "user_id": 12,                     // the Id of the user these tokens are valid for
   "user_name": "test.user"           // the username of the user these tokens are valid for
}
```

After receiving the Access Token, you can use it to request resources from the
API.

#### Resource Request

To retrieve resources from the API, add the Access Token to the Authorization
header in the following form: Bearer {AccessToken}

**Example:**

```
Authorization: Bearer FGDoJBgK96Z...
```

Access Tokens expire and need to be refreshed with the Refresh Token.

#### Refresh Token Request

Every Access Token expires after a time, usually after 30 days. You can use the
Refresh Token to retrieve a new Access Token. The Refresh Token only expires
when the user revokes the token.

**Parameter:**

- `GrantType`: the grant type needs to be set to refresh_token
- `RefreshToken`: the previously received Refresh Token
- `Authorization`: This request requires the HTTP Basic Authentication header.
You need to encode your {appId}:{appSecret} using the Base64 method and add it
to the Authorization header, like this: Authorization: Basic
Base64({AppId}:{AppSecret})

**Example:**

`POST https://api.hqlabs.de/Token`

```
grant_type=refresh_token&refresh_token=K8vma4VohMb...
```

**Example:**

```json
{
   "access_token": "ADFoJBgK96c...",  // the new Access Token for authorization
   "token_type": "bearer",            // the token type, usually bearer
   "expires_in": 1199,                // the expiration timespan, in seconds
   "refresh_token": "Gtv6a4VohaB...", // the new Refresh Token to request a new Access Token
   "user_id": 12,                     // the Id of the user these tokens are valid for
   "user_name": "test.user"           // the username of the user these tokens are valid for
}
```

For further questions, please contact us at
[support@helloHQ.io](support@helloHQ.io).

# Expand

Expands can be used to retrieve more data in one query. They allow to get
related entities and navigation properties from an entity.

In the URL, add the following to the `$expand=` parameter.

Here, some examples for using the `expand` option are shown.

## Expand one property

- On a company, expand the default address of the company: 

```
/v1/Companies?$expand=DefaultAddress
```

## Expand multiple properties

- On a company, expand the default address and the company types of the company:

```
/v1/Companies?$expand=DefaultAddress,CompanyTypes
```

## Expand multiple levels

- On an invoice, expand the positions and the custom fields of the positions: 

```
/v1/Invoices?$expand=Positions($expand=CustomFields)
```

- This will not work on paths with mixed cardinality, meaning that `Projects -> Positions -> CustomFields` works while `Projects -> Company -> CustomFields` will fail. In those cases, the best solution is to fetch projects as well as companies with their custom fields, and merge the results on the client.

- Be aware that multiple expanded levels can create long-running queries as well as large result sets.

# Filter

Filter are useful to construct queries against our API. This way, you don't need
to retrieve all data and filter on the client.

In the URL, add the following to the `$filter=` parameter.

If you want to filter on a sub-entity, make sure to expand the entity first.

Here, some examples for using the `filter` option are shown.

## Filter by date and time

- Get all entities updated since January 26th 2018: 

```
/v1/Companies?$filter=UpdatedOn ge 2018-01-26
```

- Get all entities updated since January 26th 2018, 6:15pm: 

```
/v1/Companies?$filter=UpdatedOn gt 2018-01-26T18:15:00+01:00
```

- Get all invoices within one month: 

```
/v1/Invoices?$filter=InvoiceDate ge 2018-03-01 and InvoiceDate lt 2018-04-01
```

## Filter by CompanyType

- Find all companies of a specific type, where 4002 is the Id of the CompanyType
'Customer': 

```
/v1/Companies?$expand=CompanyTypes&$filter=CompanyTypes/any(companyType: companyType/Id eq 4002)
```

- Find all companies of a specific type, where 'Lieferant' is the name of the
CompanyType: 

```
/v1/Companies?$expand=CompanyTypes&$filter=CompanyTypes/any(companyType: companyType/Name eq 'Lieferant')
```

# HTTP Conventions

## Response Types

Generally, all API responses, as well as the POST and PUT bodies, are in JSON
and follow the OData 4 definition.

The OData response wraps the actual data in the data field and adds additional
metadata.

However, there are some cases where the API provides access to binary data, like
PDFs and images.

Every document, for example invoices and quotations, can be retrieved as a
generated PDF file. The PDF is generated on the fly when the document is in a
draft state. After the document was sent, the PDF is archived and the archived
PDF is returned in the API. 

You can access the metadata of the document file like this: 
`/v1/Invoices(id)/DocumentFile`

You can access the binary stream (PDF) of the document file like this: 
`/v1/Invoices(id)/DocumentFile/$value`

For further questions, please contact us at support@helloHQ.io.

## HTTP Codes

Each response to an API request indicates the success with its status code.

### Success Status Codes

```

GET           200 (OK)	 
POST          201 (Created)	 
PUT           200 (OK)	 
DELETE        240 (No content)	 
do.Action()   200 (OK)            If base entity was changed
do.Action()   201 (Created)       If new entity was created

```

The body of the reponse with status code 200 and 201 contain the changed/created
object. Responses with code 204 always return an empty body.

### Failure Status Codes

```

GET / PUT / DELETE / do.Action()   404 (Not Found)     No object found / Invalid Id
POST / PUT / do.Action()           400 (Bad request)   Missing request body
POST / PUT / do.Action()           400 (Bad request)   Missing request argument
POST / PUT / do.Action()           400 (Bad request)   Validation failed

```

## Dates and Times

The date and time information is generally in UTC. The client is responsible for
converting the date and time information to user time. The user object contains
the selected timezone of the user. Date and time properties in UTC end on ...On,
for example StartOn or CreatedOn.

There are, however, a few exceptions, where only the date is relevant. In these
cases, the date information is represented without a timezone and should not be
converted. Date properties end on ...Date, for example StartDate or ShownDate.

## Actions

Actions are POST operations performed on specific API objects. Sometimes actions
need specific parameters for the performed operation but this is not necessary
every one of them. An action is mostly a more complex operations handled on the
server which usually perform different data manipulations and object creations.
Remember that an action should not be invoked too frequently on the same object
because of the data manipulation. One call is sufficient to get the certain
outcome.

When new objects are created during an action, in most cases the newly generated
action is beeing returned. The return value can be seen in the action
description iteself.

# Further References

Please refer to the following pages and guides to learn how to use our API.

- **OData**: [http://www.odata.org/](http://www.odata.org/)
- **OAuth 2.0**: [http://oauth.net/2/](http://oauth.net/2/)
- **HQ Help**: [http://help.hellohq.io](http://help.hellohq.io)
