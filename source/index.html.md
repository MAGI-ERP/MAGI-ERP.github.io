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

<aside class="danger">
This API requires knowledge of WinMAGI's System Dictionary. 
You will frequently need to access the WinMAGI System Dictionary while developing your intergration.
</aside>

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

### JSON-formatted Parameters

Parameter | Required | Description
--------- | -------- | -----------
CUSTID | false | If supplied, return customers matching this CUSTID
COMPANY | false | If supplied, return customers matching this Company 
ANY_OTHER_FIELD | false | Any WinMAGI field in the CUSTOMER table
 
<aside class="success">
Wildcards (% and _) are supported.
</aside>

<aside class="success">
Field values in lookups are not case sensitive
</aside>

<aside class="success">
Multiple fields can be provided. Records that match all values provided will be returned.
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

### JSON-formatted Parameters

Parameter | Required | Description
--------- | -------- | -----------
CUSTID | false | If supplied, return customers matching this CUSTID
SHIPID| false | If supplied, return customers matching this SHIPID
ADDRESS | false | If supplied, return customers matching this Address
CITY | false | If supplied, return customers matching this City
ANY_OTHER_FIELD | false | Any WinMAGI field in the CUSTADDR table

<aside class="success">
Wildcards (% and _) are supported.
</aside>

<aside class="success">
Field values in lookups are not case sensitive
</aside>

<aside class="success">
Multiple fields can be provided. Records that match all values provided will be returned.
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


# Sales Orders

The Sales Order object enables Sales Order actions that will run through the WinMAGI code for validation, 
consistency, and concurrency with WinMAGI users.

## Generate the Sales Order object

```vb
Dim objOrder = objWinMAGI.mthGetOrderObject()

' Errors will return null, and a message will be present in objWinMAGI.LastErrorText
```

Grab a Sales Order object from the Sales Order Factory

## Get Sales Orders

```vb
Dim jsonOrders As String

' Get all sales orders for customer MAGI
jsonOrders = objOrder.mthFindOrder("{""CUSTID"":""MAGI""}")   
```

> The above command returns JSON structured like this:

```json
[
  {
    "CUSTID": "MAGI",
    "OENO": "29387",
    "COMPANY": "MAGI Software",
    "ORDDATE": "2018-10-18",
    "STATUS": "3",
    "PONUM": "A11091",
    "CITY": "Grand Rapids",
    "STATE": "MI",
    "ZIP": "49546",
    "ADDRESS": "2660 Horizon Dr SE",
    "...All other fields in WinMAGI COMAST table":""
  },
  {
    "CUSTID": "MAGI",
    "OENO": "29323",
    "COMPANY": "MAGI Software",
    "ORDDATE": "2018-08-14",
    "STATUS": "4",
    "PONUM": "A54432",
    "CITY": "Grand Rapids",
    "STATE": "MI",
    "ZIP": "49546",
    "ADDRESS": "2660 Horizon Dr SE",
    "...All other fields in WinMAGI COMAST table":""
  }
]
```

```
' Find sales order by PO (case-insensitive)
jsonOrders = objCustomer.mthFindAddress("{""PONUM"":""A54432""}")   
```

> The above command returns JSON structured like this:

```json
[
  {
    "CUSTID": "MAGI",
    "OENO": "29323",
    "COMPANY": "MAGI Software",
    "ORDDATE": "2018-08-14",
    "STATUS": "4",
    "PONUM": "A54432",
    "CITY": "Grand Rapids",
    "STATE": "MI",
    "ZIP": "49546",
    "ADDRESS": "2660 Horizon Dr SE",
    "...All other fields in WinMAGI COMAST table":""
  }
]
```

This endpoint retrieves all sales orders matching JSON-formatted criteria you provide.

### JSON-formatted Parameters

Parameter | Required | Description
--------- | -------- | -----------
CUSTID | false | If supplied, return customers matching this CUSTID
SHIPID| false | If supplied, return customers matching this SHIPID
ADDRESS | false | If supplied, return customers matching this Address
CITY | false | If supplied, return customers matching this City
PONUM | false | If supplied, return customers matching this Po Number
ANY_OTHER_FIELD | false | Any WinMAGI field in the CUSTADDR table
 
Multiple fields in the query will return results matching ALL supplied values. 

<aside class="success">
Wildcards (% and _) are supported.
</aside>

<aside class="success">
Field values in lookups are not case sensitive
</aside>

<aside class="success">
Multiple fields can be provided. Records that match all values provided will be returned.
</aside>


## Create a Sales Order

```vb
Dim jsonOrder As String

' Create a sales order
jsonOrder = objOrder.mthCreateOrder("{""CUSTID"":""MAGI"", ""SHIPID"":""1"", ""PREPAYAMT"": 77.28
""ECORDERID"":""14423"", ""LINEITEMS"":[ {""PN"": ""A101"", ""QTYORD"": 4, ""CUSELL"": 19.32, ""TAXABLE"": false } ]}")   
```

> The above command returns the newly-created JSON-formatted sales order object structured like this if successful:

```json
{
    "OENO": "22545",
    "CUSTID": "MAGI",
    "SHIPID": "1",
    "ECORDERID": "14423",
    "PREPAYAMT": 77.28,
    "COMPANY": "MAGI Software",
    "CITY": "Grand Rapids",
    "STATE": "MI",
    "ZIP": "",
    "ADDRESS": "",
    "...All other fields in WinMAGI COMAST table":"",
    "LINEITEMS": [
      {
          "PN": "A101",
          "QTYORD": 4,
          "CUSELL": 19.32,
          "...All other fields in WinMAGI CODET table":""
      }
    ]
}
```

> Unsuccessful attempts return an empty string and assign details to objOrder.LastErrorText

Create a WinMAGI sales order.

* Only your custom "Z" fields and those listed below are supported.

* You may provide a WinMAGI TaxCode (see TaxCode lookup section), or you may provide leave TAXCODE blank to provide 
TAXES1-4 with TAXACCT1-4

* CUSTID must already exist and be UPPER CASE

* SHIPID must be unique and UPPER CASE if you choose to provide it.

* ECORDERID is required and must be unique. It is intended to be the link to the source system's order.

* Successful creation returns the new sales order object in JSON format. Any other result indicates an error. Check LastErrorText.

* Fields MUST be UPPER CASE

* Validations from your WinMAGI System Dictionary will be evaluated

<aside class="warning">
Only your custom "Z" fields and select WinMAGI fields are supported for sales orders! Request additional fields as needed.
</aside>

<aside class="success">
Status "2" (Approved) automatically becomes status "3" (Released) upon saving an order. Use Status "0" for quotes, 
"1" for planned orders, and "3" to release orders.
</aside>

### JSON-formatted Parameters

SALES ORDER:

Parameter | Required | Description
--------- | -------- | -----------
CUSTID | true | Existing, upper-cased string customer ID
SHIPID | false | Customer shipping address record ID (see Customer Address lookup section above)
ECORDERID | true | Unique order ID from external source
COMPANY  | false | Override company
ADDRESS1 | false | Override Billing Address 1
ADDRESS2 | false | Override Billing Address 2
CITY     | false | Override Billing City
STATE    | false | Override Billing State
ZIP      | false | Override Billing Zip
COUNTRY  | false | Override Billing Country
EMAIL    | false | Override Billing Email
PHONE    | false | Override Billing Phone
SCOMPANY | false | Shipping Company (Only if SHIPID is empty)
SCONTACT | false | Shipping Contact (Only if SHIPID is empty) 
SPHONE | false | Shipping Phone (Only if SHIPID is empty) 
SEMAIL | false | Shipping Email (Only if SHIPID is empty) 
SADDRESS1 | false | Shipping Address 1 (Only if SHIPID is empty) 
SADDRESS2 | false | Shipping Address 2 (Only if SHIPID is empty) 
SCITY | false | Shipping City (Only if SHIPID is empty) 
SSTATE | false | Shipping State (Only if SHIPID is empty) 
SZIP | false | Shipping Zip (Only if SHIPID is empty) 
SCOUNTRY | false | Shipping Country (Only if SHIPID is empty) 
LINEITEMS | true | a JSON array of lineitems (see below)
PONUM | depends | PO Number required depends on your WinMAGI settings
ENTRYDATE | false | Date of order. Defaults to current date. 
ORDDATE | false | Date of PO
REQDATE | false | Customer's requested ship date
SCHDDATE | false | Scheduled Ship Date
STATUS | false | Order status. Defaults to "1" (Planned)
TAXES | false | Tax total
TAXES1 | false | Tax amount for account 1
TAXES2 | false | Tax amount for account 2
TAXES3 | false | Tax amount for account 3
TAXES4 | false | Tax amount for account 4
TAXFRT | false | Freight taxable?
PREPAYMENT | false | Amount of prepayment
CURRENCY | false | Currency code
TOTAL | false | Order total. Used for automatic AR posting.
DATE_PAID | false | Date of payment. Used for automatic AR posting.
SHIPCODE | false | Shipping method from ShipVia table (see ShipVia lookup section below)
HOLDCODE | false | Order hold code
MDESC | false | Misc. charge description
MACCTNO | false | Misc. charge Account
MCHG | false | Misc. charge Amount
TAXMISC | false | Misc. charge taxable?
REMCUSTSVC | false | Customer service remarks
REMARKS | false | Order remarks
QOEREMARKS | false | Quote remarks
SALESMAN | false | Salesman ID (see Salesman lookup section below)
ZFIELDS | false | Any fields beginning with "Z" which is standard practice for custom fields
    

LINEITEMS:

Parameter | Required | Description
--------- | -------- | -----------
PN | true | Existing, upper-cased string WinMAGI Part Number
QTYORD | true | Quantity of item ordered
CUSELL | true | Selling price per item in customer currency
TAXABLE | false | Is item taxable?
REMARKS | false | Item details. Defaults to WinMAGI Item Master remarks.
ZFIELDS | false | Any fields beginning with "Z" which is standard practice for custom fields


## Update a Sales Order

<aside class="warning">
Updating existing sales orders is not yet supported.
</aside>


## Delete a Sales Order

<aside class="warning">
Deleting existing sales orders is not yet supported.
</aside>


## Get Salesman

```vb
Dim jsonResult As String

' Look up salesman ID
jsonResult = objOrder.mthFindSalesman("{""NAME"":""Jim R Johnson""}")   
```

> The above command returns the following JSON-formatted records:

```json
{
    "SALESMAN": "12",
    "CODE": "ALL",
    "NAME": "Jim R Johnson",
    "ADDRESS1": "12 8th Ave.",
    "PHONE": "616-555-4456",
    "COMPANY": "MAGI Software",
    "CITY": "Grand Rapids",
    "STATE": "MI",
    "ZIP": "",
    "...All other fields in WinMAGI SALESMAN table":""
}
```

Look up a salesman's ID for creating an order

Parameter | Required | Description
--------- | -------- | -----------
SALESMAN | false | Salesman Code
CODE| false | Salesman commission code
NAME | false | Salesman name
CITY | false | Salesman city
ZIP | false | Salesman ZIP
PHONE | false | Salesman phone number 
ANY_OTHER_FIELD | false | Any WinMAGI field in the SALESMAN table

<aside class="success">
Wildcards (% and _) are supported.
</aside>

<aside class="success">
Field values in lookups are not case sensitive
</aside>

<aside class="success">
Multiple fields can be provided. Records that match all values provided will be returned.
</aside>

## Get ShipVia

```vb
Dim jsonResult As String

' Look up ShipVia records
jsonResult = objOrder.mthFindShipVia("{""DESC"":""UPS Ground""}")   
```

> The above command returns the following JSON-formatted records:

```json
{
    "SHIPCODE": "01",
    "DESC": "UPS Ground",
    "CARCONTACT": "Jen Smith",
    "...All other fields in WinMAGI SHIPVIA table":""
}
```

Look up a ShipVia ID for creating an order

Parameter | Required | Description
--------- | -------- | -----------
SHIPCODE | false | ShipVia Code
DESC | false | Description
CARCONTACT | false | Carrier Contact
ANY_OTHER_FIELD | false | Any WinMAGI field in the SHIPVIA table

<aside class="success">
Wildcards (% and _) are supported.
</aside>

<aside class="success">
Field values in lookups are not case sensitive
</aside>

<aside class="success">
Multiple fields can be provided. Records that match all values provided will be returned.
</aside>


## Get TaxCode

```vb
Dim jsonResult As String

' Look up TaxRate records
jsonResult = objOrder.mthFindTaxRate("{""ZIP"":""49546""}")   
```

> The above command returns the following JSON-formatted records:

```json
{
    "TAXCODE": "01",
    "DESC": "Michigan - Grand Rapids",
    "ZIP": "49546",
    "RATE1": 6.000,
    "RATE2": 0.000,
    "RATE3": 0.000,
    "RATE4": 0.000,
    "DESC1": "",
    "DESC2": "",
    "DESC3": "",
    "DESC4": "",
    "ACCT1": "",
    "ACCT2": "",
    "ACCT3": "",
    "ACCT4": "",
    "...All other fields in WinMAGI TAXRATE table":""
}
```

Look up a TaxCode to automatically assign tax rates when creating an order

Parameter | Required | Description
--------- | -------- | -----------
TAXCODE | false | TaxRate record ID
DESC | false | Description
STATE | false | State
ZIP | false | Zip code
COUNTRY | false | Country
ANY_OTHER_FIELD | false | Any WinMAGI field in the TAXRATE table

<aside class="success">
Wildcards (% and _) are supported.
</aside>

<aside class="success">
Field values in lookups are not case sensitive
</aside>

<aside class="success">
Multiple fields can be provided. Records that match all values provided will be returned.
</aside>
