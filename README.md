Unofficial documentation of the [Frappe](https://frappe.io) / [ERPNext](https://erpnext.org) API. Online Docs: [Swagger Hub](https://app.swaggerhub.com/apis-docs/alyf.de/Frappe/0.0.1)

---

Base URL: https://{YOUR ERPNEXT INSTANCE}/api

Example: https://demo.erpnext.com/api

# Authentication

## POST /method/login

Content-Type: application/x-www-form-urlencoded

Params (in body):

* usr (string)
	
	Username

* pwd (string)

	Password

Example:

```bash
curl -X POST https://demo.erpnext.com/api/method/login \
     -H 'Content-Type: application/json' \
     -H 'Accept: application/json' \
     -d '{"usr":"Administrator","pwd":"admin"}'
```

Returns:

* HTTP Code: 200
* application/json:

```json
	{
		"home_page": "/desk",
		"full_name": "Administrator",
		"message": "Logged in"
	}
```

* Cookie: `sid` (send this to authenticate future requests). [Expires in three days](https://github.com/frappe/frappe/blob/e551153ea0a5fb905f2d9508143a9d25ec74aa43/frappe/auth.py#L320).

```
	sid=05d8d46aaebff1c87a90f570a3ff1c0f570a3ff1c87a90f56bacd4; 
	path=/; 
	domain=.demo.erpnext.com; 
	Expires=Sat, 29 Sep 2018 00:59:54 GMT;
```

Error:

* HTTP Code: 401
* text/html: Wrong password or username.

## GET /method/logout

Example:

```bash
	curl -X GET https://demo.erpnext.com/api/method/logout
```

Returns:

* HTTP Code: 200
* application/json: `{}`

## GET /method/frappe.auth.get_logged_user

Get the ID of the currently authenticated user.

Example:

```bash
	curl -X GET https://demo.erpnext.com/api/method/frappe.auth.get_logged_user
```

Returns:

* HTTP Code: 200
* application/json:

```json
	{
	  "message": "demo@erpnext.com"
	}
```

# OAuth2

Use the header `Authorizaton: Bearer <access_token>` to perform authenticated requests. You can receive a [bearer token](https://tools.ietf.org/html/rfc6750) by combining the following two requests.

## POST /method/frappe.integrations.oauth2.authorize

Get an authorization code from the user to access ERPNext on his behalf. 

Params (in body):

* client_id (string)

	ID of your OAuth2 application

* state (string)

	Arbitrary value used by your client application to maintain state between the request and callback. The authorization server includes this value when redirecting the user-agent back to the client. The parameter should be used for preventing [cross-site request forgery](https://tools.ietf.org/html/rfc6749#section-10.12).

* response_type (string)

	"code"

* scope (string)
	
	The scope of access that should be granted to your application.

* redirect_uri (string)

	Callback URI that the user will be redirected to, after the application is authorized. The authorization code can then be extracted from the params.

Content-Type: application/x-www-form-urlencoded

Example:

```bash
curl -X POST https://demo.erpnext.com/api/method/frappe.integrations.oauth2.authorize \
     -H 'Content-Type: application/x-www-form-urlencoded' \
     -H 'Accept: application/json' \
     -d 'client_id=511cb2ac2d&state=444&response_type=code&scope=all
  	     &redirect_uri=https%3A%2F%2Fapp.getpostman.com%2Foauth2%2Fcallback'
```

For **testing purposes** you can also pass the parameters in the URL, like this (and open it in the browser):

`https://demo.erpnext.com/api/method/frappe.integrations.oauth2.authorize?client_id=511cb2ac2d&state=444&response_type=code&scope=all&redirect_uri=https%3A%2F%2Fapp.getpostman.com%2Foauth2%2Fcallback`


Returns:

* HTTP Code: 200
* text/html
	
	This will open the authorize page which then redirects you to the `redirect_uri`.

If the user clicks 'Allow', the redirect URI will be called with an authorization code in the query parameters:

`https://{REDIRECT URI}?code=plkj2mqDLwaLJAgDBAkyR1W8Co08Ud`

If user clicks 'Deny' you will receive an error:

`https://{REDIRECT URI}?error=access_denied`


## POST /method/frappe.integrations.oauth2.get_token

Trade the authorization code (obtained above) for an access token.

Params (in body):

* grant_type (string)

	"authorization_code"

* code (string)

	Authorization code received in redirect URI.

* client_id (string)

	ID of your OAuth2 application

* redirect_uri (string)

Content-Type: application/x-www-form-urlencoded

Example:

```bash
curl -X POST https://demo.erpnext.com/api/method/frappe.integrations.oauth2.get_token \
     -H 'Content-Type: application/x-www-form-urlencoded' \
     -H 'Accept: application/json' \
     -d 'grant_type=authorization_code&code=wa1YuQMff2ZXEAu2ZBHLpJRQXcGZdr
         &redirect_uri=https%3A%2F%2Fapp.getpostman.com%2Foauth2%2Fcallback&client_id=af615c2d3a'
```
For **testing purposes** you can also pass the parameters in the URL like this (and open it in the browser):

`https://demo.erpnext.com/api/method/frappe.integrations.oauth2.get_token?grant_type=authorization_code&code=A1KBRoYAN1uxrLAcdGLmvPKsRQLvzj&client_id=511cb2ac2d&redirect_uri=https%3A%2F%2Fapp.getpostman.com%2Foauth2%2Fcallback`

Returns:
	
```json
	{
	    "access_token": "pNO2DpTMHTcFHYUXwzs74k6idQBmnI",
	    "token_type": "Bearer",
	    "expires_in": 3600,
	    "refresh_token": "cp74cxbbDgaxFuUZ8Usc7egYlhKbH1",
	    "scope": "all"
	}
```


## POST /method/frappe.integrations.oauth2.revoke_token

Revoke token endpoint.

Params:

* token

	Access token to be revoked.

Returns:
	
```json
	{
		"message": "success"
	}
```

# Manipulating DocTypes

A DocTypes is a specific type of document, for example: Customer, Employee or Item.

A DocumentName is the unique ID of a Document, for example: CUST-00001, EMP-00001 or ITEM-00001.

Authentication is missing in the following examples. See [Basic Authentication] and [OAuth2] for more.

## GET /resource/{DocType}

Get a list of documents of this DocType.

Params (in path):

* DocType (string)

	The DocType you'd like to receive. For example, 'Customer'.

Params (in query):

* fields []

	By default, only the 'name' field will be returned. To add more fields, you can pass the *fields* parameter. For example, fields=["name","country"]

* filters [[(string)]]

	You can filter the listing using SQL conditions by passing them in the *filters* parameter. Each condition is an array of the format, [{doctype}, {field}, {operator}, {value}]. For example, filters=[["Customer", "country", "=", "India"]]

* limit_page_length (int)

	All listings will be paginated. With this parameter you can change the page size (how many items are teturned at once). Default: 20.

* limit_start (int)

	To request successive pages, pass a multiple of your limit_page_length as limit_start. For example, to request the second page, pass limit_start as 20.

Example:

*Get at most 20 Names (IDs) of all Customers whose phone number is 4915227058038.*

```bash
curl -X GET https://demo.erpnext.com/api/resource/Customer?fields=["name"]\
            &filters=[["Customer","phone","=","4915227058038"]]
```

Returns:

```json
	{
	  "data": [
	    {
	      "name": "CUST-00001"
	    },
	  ]
	}
```

## POST /resource/{DocType}

Create a new document of this DocType.

Params (in path):

* DocType (string)

	The DocType you'd like to create. For example, 'Customer'.

Content-Type: application/json

Request Body: `{"fieldname": value}`

Example:

*Create a new Lead named Mustermann.*

```bash
curl -X POST https://demo.erpnext.com/api/resource/Lead \
     -H 'Content-Type: application/json' \
     -H 'Accept: application/json' \
     -d '{"lead_name":"Mustermann"}'
```

## GET /resource/{DocType}/{DocumentName}

Retrieve a specific document by name (ID).

Params (in path):

* DocType (string)

	The type of the document you'd like to get. For example, 'Customer'.

* DocumentName (string)

	The name (ID) of the document you'd like to get. For example, 'EMP-00001'.

Example:

*Get the Customer with Name (ID) CUST-00001.* 

```bash
curl -X GET https://demo.erpnext.com/api/resource/Customer/CUST-00001
```

## PUT /resource/{DocType}/{DocumentName}

Update a specific document. This acts like a `PATCH` HTTP request in which you do not have to send the whole document but only the parts you want to change.

Params (in path):

* DocType (string)

	The type of the document you'd like to update. For example, 'Customer'.

* DocumentName (string)

	The name (ID) of the document you'd like to update. For example, 'EMP-00001'.

Content-Type: application/json

Request Body: `{"fieldname": value}`

Example:

*Update Next Contact Date.*

```bash
curl -X PUT https://demo.erpnext.com/api/resource/Lead/LEAD-00001 \
     -H 'Accept: application/json' \
     -H 'Content-Type: application/json' \
     -d '{"contact_date":"2018-10-08"}'
```

Returns:

```json
{
    "data": {
        "doctype": "Lead",
        "name": "LEAD-00001",
        "contact_date": "2018-10-08",
        "...": "..."
    }
}
```

## DELETE /resource/{DocType}/{DocumentName}

Params (in path):

* DocType (string)

	The type of the document you'd like to delete. For example, 'Customer'.

* DocumentName (string)

	The name (ID) of the document you'd like to delete. For example, 'EMP-00001'.

# Further Reading

HTTP Headers:

* [Content-Type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type)
* [Accept](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept)
* [Authorization](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization)

oAuth2:

* [Specification](https://tools.ietf.org/html/rfc6749)
* [Bearer token](https://tools.ietf.org/html/rfc6750)
