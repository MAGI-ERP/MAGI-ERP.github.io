# Errors

<aside class="notice">
There are numerous errors you may encounter at every point in the API process. Many of them depend on your unique 
WinMAGI System Dictionary. Ask MAGI for clarification of error messages.
</aside>

Here are some of the most common error messages you may receive in the LastErrorText property. 
This list is not all-inclusive. 

## InitOleObject - MAGI License Server errors

Error Message | Meaning
---------- | -------
License Server date and time do not match this client | Server and client date/time out of sync
WinMAGI license is NOT valid for use | License is invalid
WinMAGI license has expired | License is expired
You are trying to run a WinMAGI version that is newer than your maintenance expiration | Not authorized to run this build version
Maximum concurrent users exceeded | No available user seats
The WinMAGI License Server is set to deny access for all users | WinMAGI access is shut down for maintenance
Company not found | Invalid parameter passed to InitOleObject
Cannot connect to the WinMAGI License Server at | Communication with License Server was unsuccessful
Lost connection to the WinMAGI License Server | Connection to the License Server dropped


## General errors

Error Message | Meaning
---------- | -------
Missing data in JSON format | No data passed in or not properly formatted 
No search fields provided | No fields provided in the JSON-formatted parameter
Module not authorized for use | Trying to use an API module that is not authorized for use 


## Create / Update - Field validation errors

Error Message | Meaning
---------- | -------
<field> validation failed: SySeek(... | Uniqueness constraint violated for field <field>


## Delete - Relationship constraints

Error Message | Meaning
---------- | -------
Unable to delete due to dependencies | Dependencies exist in other tables that prevent deletion
Unable to lock record for delete | Someone is using the record
Invalid CustId | CUSTID provided does not exist
Missing CustId | Required CUSTID not provided

