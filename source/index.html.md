---
title: WinMAGI API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - vb

toc_footers:

  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the WinMAGI API! You can use our API to access your WinMAGI data from third-party applications. With this API, 
you can create, update, and delete customers and shipping addresses, create sales orders, and query all sorts 
of data in your WinMAGI database.

If there is data or a function you would like exposed in the API, contact us at <a href="mailto:support@magi-erp.com">support@magi-erp.com</a> 

# Pre-requisites

* MAGI License Server version 101 or later
* WinMAGI build V821332 or later
* Register the WinMAGI.exe

# Important information

* Each instance of WinMAGI.ApiBase consumes a limited user.
* Each instance of WinMAGI.ApiBase will be present in the MAGI License Server for management purposes.
* After installing a new WinMAGI build, it may be necessary to unregister and register the WinMAGI.exe again
* Supported fields and validations, in most cases, come from your WinMAGI System Dictionary. 
    * It would be wise to create a wrapper that limits what third-party applications can do with the API.


# Registering the library

> You must first register the WinMAGI API library on the machine(s) that will consume it.

Open an elevated command prompt, navigate to the directory containing your WinMAGI.exe, and type winmagi.exe /regserver

You may unregister in the same way using the /unregserver flag.

# Authentication

> To authorize, use this code:

```vb
' Initialize the object, connect with the license server, validate the license and available users
' objWinMAGI.InitOleObject("<path\to\winmagi>", "<DATA directory name>")

' InitOleObject takes a few seconds as it communicates with the MAGI License Server asynchronously.

Dim objWinMAGI = CreateObject("WinMAGI.ApiBase")
Dim lnResult As Int16 = objWinMAGI.InitOleObject("c:\winmagi", "DATA")  

If lnResult > 0 Then

    ' Wait for the property lLoginResponseReceived to indicate successful login to the MAGI License Server
    Do While objWinMAGI.lLoginResponseReceived = False
        System.Threading.Thread.CurrentThread.Sleep(1000)
    Loop
    
    Console.Write("Successful login!")
    
    ' Do stuff
Else
    ' Error with the login or authentication process
    Console.Write(objWinMAGI.LastErrorText)
End If
```

Authentication involves communication with your WinMAGI License Server to obtain license data including users 
and authorized modules.

Upon successful login to the MAGI License Server, your API user will be shown in the concurrent users list. 
 
Integration with the license server enables visibility into the processes that may be accessing the WinMAGI data. 
This is useful for having the ability to block API users from accessing the system WinMAGI maintenance and build upgrades.

Each instance of ApiBase consumes one concurrent Limited User rather than a full user seat.

One Limited User should be sufficient to interface with WinMAGI from your middleware. 

However, if you need multi-threading 
capabilities or would like to run instances from multiple machines, additional Limited Users may be purchased.

# Important API Details

* Data is queried by passing a set of fields in JSON format. 
    * Supported fields are any fields in your WinMAGI System Dictionary for the table you are querying.
    * Sales Orders have a limited set of supported fields.
  
* Wildcards (% and _) are supported.
  
* JSON fields MUST be all CAPS
  
* Multiple fields will be ANDed to find a result that matches all criteria provided

* Returns
    * Results found: a JSON array 
    * No results found: An empty JSON object
    * Error: An empty string. Query objCustomer.LastErrorText for details.
    
* Validations
    * Validation of values for an entity is controlled by the "When" and "Valid" methods in your WinMAGI System Dictionary for the table in question.
     
    * Any field in the dictionary may be supplied in your JSON-formatted object.
        * Only select fields are supported for Sales Orders.
     
    * If a supplied field's "When" method evaluates to false, no changes will be made to the object.
      
    * If the value you provide results in the field's "Valid" method evaluating to false, no changes will be made. 
    
    * Details of validation failure will be in the LastErrorText property.
 

# Factories

Each instance of WinMAGI.ApiBase can generate one instance of each entity type.
 
For example, you may call objWinMAGI.mthGetCustomerObject() and objWinMAGI.mthGetSalesOrderObject() which will 
 allow you to create a customer and an order with one authenticated session while consuming only a single limited user.
 
Our intention is that you leave objWinMAGI in memory and logged into the license server as long as possible.

# Customers

The customer object enables Customer and Customer Address actions that will run through the WinMAGI code for validation, 
consistency, and concurrency with WinMAGI users.

## Generate the Customer object

```vb
Dim objCustomer = objWinMAGI.mthGetCustomerObject()

' Errors will return null, and a message will be present in objWinMAGI.LastErrorText
```

Grab a customer object from the Customer Factory

## Get Customers

```vb
Dim jsonCustomers As String

' Get all customers
jsonCustomers = objCustomer.mthFindCustomer("{""CUSTID"":""%""}")   

' Get customer with company exactly matching Walmart (case-insensitive)
jsonCustomers = objCustomer.mthFindCustomer("{""COMPANY"":""Walmart""}")   

' Get customer with company exactly matching Walmart and city matching Bentonville
jsonCustomers = objCustomer.mthFindCustomer("{""COMPANY"":""Walmart"", 
""CITY"": ""Bentonville""}")    
```

> This method returns JSON structured like this:

```json
[
  {
    "CUSTID": "WALMART",
    "COMPANY": "Walmart",
    "CITY": "Bentonville",
    "STATE": "AR",
    "ZIP": "72712",
    "ADDRESS": "406 S Walton Blvd",
    "...All other fields in WinMAGI CUSTOMER table":""
  },
  {
    "CUSTID": "BESTBUY",
    "COMPANY": "Best Buy",
    "CITY": "Grand Rapids",
    "STATE": "MI",
    "ZIP": "49546",
    "ADDRESS": "2650 E Belt Line SE",
    "...All other fields in WinMAGI CUSTOMER table":""
  }
]
```

This endpoint retrieves all customers matching JSON-formatted criteria you provide.

Fields MUST be UPPER CASE

### JSON-formatted Parameters

Parameter | Required | Description
--------- | -------- | -----------
CUSTID | false | If supplied, return customers matching this CUSTID
COMPANY | false | If supplied, return customers matching this Company 
ANY_OTHER_FIELD | false | Any WinMAGI field in the CUSTOMER table
 
Multiple fields in the query will return results matching ALL supplied values. 

<aside class="success">
Wildcards are supported. % for multiple characters and _ for single character.
</aside>


## Create a Customer

```vb
Dim jsonCustomer As String

' Create a customer
jsonCustomer = objCustomer.mthCreateCustomer("{""CUSTID"":""MAGI"", 
""COMPANY"":""MAGI Software"",""CITY"":""Grand Rapids"", ""STATE"":""MI""}")   
```

> The above command returns the newly-created JSON-formatted customer object structured like this if successful:

```json
{
    "CUSTID": "MAGI",
    "COMPANY": "MAGI Software",
    "CITY": "Grand Rapids",
    "STATE": "MI",
    "ZIP": "",
    "ADDRESS": "",
    "...All other fields in WinMAGI CUSTOMER table":""
}
```

> Unsuccessful attempts return an empty string and assign details to objCustomer.LastErrorText

Create a WinMAGI customer.

* Only one customer can be created with each call to mthCreateCustomer

* Fields you may provide can be found in the WinMAGI System Dictionary for the CUSTOMER table on the Fields tab

* CUSTID must be unique and UPPER CASE

* Successful creation returns the complete customer object in JSON. Any other result indicates an error. Check LastErrorText.

* Fields MUST be UPPER CASE

* Validations from your WinMAGI System Dictionary will be evaluated

### JSON-formatted Parameters

Parameter | Required | Description
--------- | -------- | -----------
CUSTID | true | Unique, Upper-cased string customer ID
COMPANY | false | Company name
ADDRESS | false | Company address
CITY | false | Company city
STATE | false | Company state
ZIP | false | Company zip
ANY_OTHER_FIELD | false | Any WinMAGI field in the CUSTOMER table


## Update a Customer
```vb
Dim jsonCustomer As String

' Update an existing customer
' This code updates the CITY to Traverse City for customer MAGI
jsonCustomer = objCustomer.mthUpdateCustomer("{""CUSTID"":""MAGI"",""CITY"":""Traverse City""}")   
```

> The above command returns the JSON-formatted customer object structured like this if successful:

```json
{
    "CUSTID": "MAGI",
    "COMPANY": "MAGI Software",
    "CITY": "Traverse City",
    "STATE": "MI",
    "ZIP": "",
    "ADDRESS": "",
    "...All other fields in WinMAGI CUSTOMER table":""
}
```

> Unsuccessful attempts return an empty string and assign details to objCustomer.LastErrorText

Update a WinMAGI customer.

* CUSTID is REQUIRED and must be UPPER CASE

* Only one customer can be created with each call

* Fields you may provide can be found in the WinMAGI System Dictionary for the CUSTOMER table on the Fields tab

* Successful creation returns the complete customer object in JSON. Any other result indicates an error. Check LastErrorText.

* Fields MUST be UPPER CASE

* Validations from your WinMAGI System Dictionary will be evaluated

### JSON-formatted Parameters

Parameter | Required | Description
--------- | -------- | -----------
CUSTID | true | Existing, Upper-cased string WinMAGI customer ID
COMPANY | false | Company name
ADDRESS | false | Company address
CITY | false | Company city
STATE | false | Company state
ZIP | false | Company zip
ANY_OTHER_FIELD | false | Any WinMAGI field in the CUSTOMER table

## Delete a Customer

```vb
Dim jsonCustomer As String

' Attempt to delete a customer
jsonCustomer = objCustomer.mthDeleteCustomer("{""CUSTID"":""MAGI""}")   
```

> Successful calls return the following:

```json
{
    "CUSTID": "MAGI",
    "STATUS": "DELETED"
}
```

> Unsuccessful calls return an empty string and assign details to objCustomer.LastErrorText


Delete a WinMAGI customer.

* CUSTID is REQUIRED and must be UPPER CASE

* Deleting a customer is not permitted in most cases due to dependencies elsewhere in the system.

### JSON-formatted Parameters

Parameter | Required | Description
--------- | -------- | -----------
CUSTID | true | Existing, Upper-cased string WinMAGI customer ID


# Customer Addresses

The customer object enables Customer and Customer Address actions that will run through the WinMAGI code for validation, 
consistency, and concurrency with WinMAGI users.

## Generate the Customer object

```vb
Dim objCustomer = objWinMAGI.mthGetCustomerObject()

' Errors will return null, and a message will be present in objWinMAGI.LastErrorText
```

Grab a customer object from the Customer Factory

## Get Customer Addresses

```vb
Dim jsonAddresses As String

' Get all customer addresses for customer MAGI
jsonAddresses = objCustomer.mthFindAddress("{""CUSTID"":""MAGI""}")   
```

> The above command returns JSON structured like this:

```json
[
  {
    "CUSTID": "MAGI",
    "SHIPID": "1",
    "CITY": "Grand Rapids",
    "STATE": "MI",
    "ZIP": "49546",
    "ADDRESS": "2660 Horizon Dr SE",
    "...All other fields in WinMAGI CUSTADDR table":""
  },
  {
    "CUSTID": "MAGI",
    "SHIPID": "2",
    "CITY": "Traverse City",
    "STATE": "MI",
    "ZIP": "49686",
    "ADDRESS": "2211 North U.S. 31 North",
    "...All other fields in WinMAGI CUSTADDR table":""
  }
]
```

```
' Get specific customer address by City (case-insensitive)
jsonAddresses = objCustomer.mthFindAddress("{""CUSTID"":""MAGI"", ""CITY"":""Grand Rapids""}")   
```

> The above command returns JSON structured like this:

```json
[
  {
    "CUSTID": "MAGI",
    "SHIPID": "1",
    "CITY": "Grand Rapids",
    "STATE": "MI",
    "ZIP": "49546",
    "ADDRESS": "2660 Horizon Dr SE",
    "...All other fields in WinMAGI CUSTADDR table":""
  }
]
```

This endpoint retrieves all customer addresses matching JSON-formatted criteria you provide.

Fields MUST be UPPER CASE

### JSON-formatted Parameters

Parameter | Required | Description
--------- | -------- | -----------
CUSTID | false | If supplied, return customers matching this CUSTID
SHIPID| false | If supplied, return customers matching this SHIPID
ADDRESS | false | If supplied, return customers matching this Address
CITY | false | If supplied, return customers matching this City
ANY_OTHER_FIELD | false | Any WinMAGI field in the CUSTADDR table
 
Multiple fields in the query will return results matching ALL supplied values. 

<aside class="success">
Wildcards are supported. % for multiple characters and _ for single character.
</aside>


## Create a Customer Address

```vb
Dim jsonCustomer As String

' Create a customer address
jsonCustomer = objCustomer.mthAddAddress("{""CUSTID"":""MAGI"", ""SHIPID"":""1"", 
""COMPANY"":""MAGI Software"",""CITY"":""Grand Rapids"", ""STATE"":""MI""}")   
```

> The above command returns the newly-created JSON-formatted customer address object structured like this if successful:

```json
{
    "CUSTID": "MAGI",
    "SHIPID": "1",
    "COMPANY": "MAGI Software",
    "CITY": "Grand Rapids",
    "STATE": "MI",
    "ZIP": "",
    "ADDRESS": "",
    "...All other fields in WinMAGI CUSTOMER table":""
}
```

> Unsuccessful attempts return an empty string and assign details to objCustomer.LastErrorText

Create a WinMAGI customer address.

* Only one customer address can be created with each call to mthAddAddress

* Fields you may provide can be found in the WinMAGI System Dictionary for the CUSTADDR table on the Fields tab

* CUSTID must already exist and be UPPER CASE

* SHIPID must be unique and UPPER CASE

* Successful creation returns the new customer address object in JSON. Any other result indicates an error. Check LastErrorText.

* Fields MUST be UPPER CASE

* Validations from your WinMAGI System Dictionary will be evaluated

### JSON-formatted Parameters

Parameter | Required | Description
--------- | -------- | -----------
CUSTID | true | Existing, upper-cased string customer ID
SHIPID | true | Unique, upper-cased string customer address ID
COMPANY | false | Company name
ADDRESS | false | Company address
CITY | false | Company city
STATE | false | Company state
ZIP | false | Company zip
ANY_OTHER_FIELD | false | Any WinMAGI field in the CUSTADDR table


## Update a Customer Address
```vb
Dim jsonCustomer As String

' Update an existing customer
' This code updates the CITY to Traverse City for customer MAGI address record "1"
jsonCustomer = objCustomer.mthUpdateAddress("{""CUSTID"":""MAGI"", ""SHIPID"":""1"",""CITY"":""Traverse City""}")   
```

> The above command returns the JSON-formatted customer address object structured like this if successful:

```json
{
    "CUSTID": "MAGI",
    "SHIPID": "1",
    "COMPANY": "MAGI Software",
    "CITY": "Traverse City",
    "STATE": "MI",
    "ZIP": "",
    "ADDRESS": "",
    "...All other fields in WinMAGI CUSTADDR table":""
}
```

> Unsuccessful attempts return an empty string and assign details to objCustomer.LastErrorText

Update a WinMAGI customer.

* CUSTID is REQUIRED and must be UPPER CASE

* SHIPID is REQUIRED and must be UPPER CASE

* Only one customer address can be updated with each call

* Fields you may provide can be found in the WinMAGI System Dictionary for the CUSTADDR table on the Fields tab

* Successful update returns the complete customer address object in JSON format. Any other result indicates an error. Check LastErrorText.

* Fields MUST be UPPER CASE

* Validations from your WinMAGI System Dictionary will be evaluated



### JSON-formatted Parameters

Parameter | Required | Description
--------- | -------- | -----------
CUSTID | true | Existing, Upper-cased string WinMAGI customer ID
SHIPID | true | Existing, Upper-cased string WinMAGI customer address ID
COMPANY | false | Company name
ADDRESS | false | Company address
CITY | false | Company city
STATE | false | Company state
ZIP | false | Company zip
ANY_OTHER_FIELD | false | Any WinMAGI field in the CUSTADDR table


## Delete a Customer Address

```vb
Dim jsonCustomer As String

' Attempt to delete a customer address
jsonCustomer = objCustomer.mthDeleteAddress("{""CUSTID"":""MAGI"", ""SHIPID"":""1""}")   
```

> Successful calls return the following:

```json
{
    "CUSTID": "MAGI",
    "SHIPID": "1",
    "STATUS": "DELETED"
}
```

> Unsuccessful calls return an empty string and assign details to objCustomer.LastErrorText


Delete a WinMAGI customer address.

* CUSTID is REQUIRED and must be UPPER CASE

* SHIPID is REQUIRED and must be UPPER CASE

### JSON-formatted Parameters

Parameter | Required | Description
--------- | -------- | -----------
CUSTID | true | Existing, Upper-cased string WinMAGI customer ID
SHIPID | true | Existing, Upper-cased string WinMAGI customer address ID













# Kittens

## Get All Kittens

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get()
```

```shell
curl "http://example.com/api/kittens"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let kittens = api.kittens.get();
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

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.get(2);
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

## Delete a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.delete(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.delete(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -X DELETE
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.delete(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "deleted" : ":("
}
```

This endpoint deletes a specific kitten.

### HTTP Request

`DELETE http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to delete

