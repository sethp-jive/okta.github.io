---
layout: docs_page
title: Okta Expression Language
redirect_from: "/docs/getting_started/okta_expression_lang.html"
---

## Overview

Expressions allow you to reference, transform and combine attributes before you store them on a user profile or before passing them to an application for authentication or provisioning.  For example, you might use a custom expression to create a username by stripping @company.com from an email address.  Or you might combine `firstName` and `lastName` attributes into a single `displayName` attribute.

This document details the features and syntax of Okta's Expression Language which can be used throughout the Okta Admin Console and API. This document will be updated over time as new capabilities are added to the language.  Okta's expression language is based on [SpEL](http://docs.spring.io/spring/docs/3.0.x/reference/expressions.html) and uses a subset of functionalities offered by SpEL.

## Referencing User Attributes
When you create an Okta expression, you can reference any attribute that lives on an Okta user profile or App user profile.

###Okta user profile
Every user has an Okta user profile.  The Okta user profile is the central source of truth for a user's core attributes.  To reference an Okta user profile attribute, just reference `user` and specify the attribute variable name.  


Syntax  | Definitions | Examples
-------- | ---------- | ------------
`user.$attribute` | `user` reference to the Okta user<br>`$attribute` the attribute variable name | user.firstName<br>user.lastName<br>user.username<br>user.email

###Application user profile
In addition to an Okta user profile, all users have separate Application user profiles for each of their applications.  The Application user profiles are used to store application specific information about a user, such as application username or user role.  To reference an App user profile attribute, just specify the application variable and the attribute variable in that application's App user profile. In specifying the application you can either name the specific application you're referencing or use an implicit reference to an in-context application.

Syntax  | Definitions | Examples
-------- | ---------- | ------------
`$appuser.$attribute` | `$appuser` explicit reference to specific app<br>`$attribute` the attribute variable name | zendesk.firstName<br>active_directory.managerUpn<br>google_apps.email
`appuser.$attribute` | `appuser` implicit reference to in-context app<br>`$attribute` the attribute variable name | appuser.firstName

###IdP user profile
In addition to an Okta user profile, some users have separate IdP user profiles for their external Identity Provider. These IdP user profiles are used to store identity provider specific information about a user, such as how to transform an external user's username into the equivalent Okta username. To reference an IdP user profile attribute, specify the identity provider variable and the corresponding attribute variable for that identity providers IdP user profile. This profile is only available when specifying the username transform used to generate an Okta username for the IdP user.

Syntax                | Definitions                                                                                | Examples
----------------------|--------------------------------------------------------------------------------------------|------------
`idpuser.$attribute`  | `idpuser` implicit reference to in-context IdP<br>`$attribute` the attribute variable name | idpuser.firstName


> With Universal Directory, there are about 30 attributes in the base Okta profile and any number of custom attributes can be added.  All App user profiles have a username attribute and possibly others depending on the application.   To find a full list of Okta user and App user attributes and their variable names, go to People > Profile Editor.  If you're not yet using Universal Directory, contact your Support or Professional Services team.

## Referencing Application and Organization Properties
In addition to referencing user attributes, you can also reference App properties, and the properties of your Organization.  To reference a particular attribute, just specify the appropriate binding and the attribute variable name.  Here are some examples:

###Application properties

Syntax | Definitions | Examples
------ | ---------- | ------------
`$app.$attribute` | `$app` explicit reference to specific app instance<br>`$attribute` the attribute variable name | google_apps_app.domain<br>zendesk_app.companySubDomain
`app.$attribute` | `app` implicit reference to in-context app instance<br>`$attribute` the attribute variable name | app.domain<br>app.companySubDomain

###Organization properties

Syntax  | Definitions | Examples
-------- | ---------- | ------------
`org.$attribute` | `org` reference to Okta org<br>`$attribute` the attribute variable name | org.domain


> For a full list of App and Org attributes, contact your Support or Professional Services team.


## Functions

Okta offers a variety of functions to manipulate attributes or properties to generate a desired output.  Functions can be combined and nested inside a single expression.

### String Functions

Function | Input Parameter Signature | Return Type | Example | Output
-------- | ------------------------- | ----------- | ------- | ------
`String.stringSwitch` | (String input, String defaultString, String... keyValuePairs) | String | `String.stringSwitch("This is a test", "default", "key1", "value1")`| default
 | | | `String.stringSwitch("This is a test", "default", "is", "value1")`| value1
 | | | `String.stringSwitch("This is a test", "default", "key1", "value1", "test", "value2")` | value2
`String.stringContains` | (String input, String searchString) | Boolean |`String.stringContains("This is a test", "test")`  | true
 | | | `String.stringContains("This is a test", "doesn'tExist")` | false
`String.removeSpaces` | (String input) | String | `String.removeSpaces("This is a test")` | Thisisatest
`String.join` | (String separator, String... strings) | String | `String.join(",", "This", "is", "a", "test")` | This,is,a,test
 | | | `String.join("", "This", "is", "a", "test")` | Thisisatest
`String.len` | (String input) | Integer | `String.len("This")` | 4
`String.append` | (String str, String suffix) | String | `String.append("This is", " a test")` | This is a test
`String.toUpperCase` | (String input) | String | `String.toUpperCase("This")` | THIS
`String.toLowerCase` | (String input) | String | `String.toLowerCase("ThiS")` | this
`String.substringBefore` | (String input, String searchString) | String | `String.substringBefore("abc@okta.com", "@")` | abc
`String.substringAfter` | (String input, String searchString) | String | `String.substringAfter("abc@okta.com", "@")` | okta.com
`String.substring `| (String input, int startIndex, int endIndex) | String | `String.substring("This is a test", 2, 9)` | is is a

The following deprecated functions perform some of the same tasks as the ones in the above table.

Function  | Example | Input | Output
-------- | --------- | -------| --------
`toUppercase(string)` | `toUppercase(source.firstName)` | Alexander | ALEXANDER
`toLowercase(string)` | `toLowercase(source.firstName)` | AlexANDER | alexander
`substringBefore(string, string)` | `substringBefore(user.email, '@')` | alex@okta.com | alex
`substringAfter(string, string)` | `substringAfter(user.email, '@')` | alex@okta.com | okta.com
`substring(string, int, int)` | `substring(source.firstName, 1, 4)` | Alexander | lex

### Array Functions

Function  | Return Type | Example | Output
-------- | ---------| --------- | --------
`Arrays.add(array, value)` | Array | `Arrays.add({10, 20, 30}, 40)` | {10, 20, 30, 40}
`Arrays.remove(array, value)` | Array | `Arrays.remove({10, 20, 30}, 20)` | {10, 30}
`Arrays.clear(array)` | Array | `Arrays.clear({10, 20, 30})` | { }
`Arrays.get(array, position)` | - | `Arrays.get({0, 1, 2}, 0)` | 0
`Arrays.flatten(list of values)` | Array | `Arrays.flatten(10, {20, 30}, 40)` | {10, 20, 30, 40}
`Arrays.contains(array, value)` | Boolean | `Arrays.contains({10, 20, 30}, 10)` | true
 |  | `Arrays.contains({10, 20, 30}, 50)` | false
`Arrays.size(array)` | Integer | `Arrays.size({10, 20, 30})` | 3
 |  | `Arrays.size(NULL)` | 0
`Arrays.isEmpty(array)` | Boolean | `Arrays.isEmpty({10, 20})` | false
 |  | `Arrays.isEmpty(NULL)` | true


### Conversion Functions

##### Data Conversion Functions

Function  | Return Type | Example | Input | Output
-------- | ---------| --------- | -------| --------
`Convert.toInt(string)` | Integer | `Convert.toInt(val)` | String val = '1234' | 1234
`Convert.toInt(double)` | Integer | `Convert.toInt(val)` | Double val = 123.4 | 123
 | | | Double val = 123.6 | 124
`Convert.toNum(string)` | Double | `Convert.toNum(val)` | String val = '3.141' | 3.141

**Note:**  Convert.toInt(double) rounds the passed numeric value either up or down to the nearest integer. Be sure to consider
integer type range limitations when converting from a number to an integer with this function.

##### Country Code Conversion Functions

These functions convert between ISO 3166-1 2-character country codes (Alpha 2), 3-character country codes (Alpha 3), numeric country codes, and full ISO country names.

Function  | Return Type | Example  | Output
-------- | ---------| --------- |  | --------
`Iso3166Convert.toAlpha2(string)` | String | `Iso3166Convert.toAlpha2("IND")`  | "IN"
`Iso3166Convert.toAlpha3(string)` | String | `Iso3166Convert.toAlpha3("840")` | "USA"
`Iso3166Convert.toNumeric(string)` | String | `Iso3166Convert.toNumeric("USA")` | "840"
`Iso3166Convert.toName(string)` | String | `Iso3166Convert.toString("IN")` | "INDIA"

**Note:**  All these functions take ISO 3166-1 2-character country codes (Alpha 2), 3-character country codes (Alpha 3), and numeric country codes as input. The function determines the input type and returns the output is in the format specified by the function name.

For more information on these codes, see the [ISO 3166-1 online lookup tool](https://www.iso.org/obp/ui/#search/code/).


### Group Functions

Function  | Return Type | Example | Output
--------- | ----------- | ------- | -------
`isMemberOfGroupName` | Boolean | `isMemberOfGroupName("group1")` | **True**, if the user under consideration is a member of *group1'; otherwise, **False**.
`isMemberOfGroup` | Boolean | `isMemberOfGroup("groupId")` | **True**, if the user under consideration is a member of group with id *groupId*; otherwise,  **False**.
`isMemberOfAnyGroup` | Boolean | `isMemberOfAnyGroup("groupId1", "groupId2", "groupId3")` | **True**, if the user under consideration is a member of any groups with ids *groupId1*, *groupId2* or *groupId3*; otherwise **False**.
`isMemberOfGroupNameStartsWith` | Boolean | `isMemberOfGroupNameStartsWith("San Fr")` | **True**, if the user under consideration is a member of any groups with names that starts with *San Fr*; otherwise,  **False**.
`isMemberOfGroupNameContains` | Boolean | `isMemberOfGroupNameContains("admin")` | **True**, if the user under consideration is a member of any groups with names that contains *admin*; otherwise,  **False**.
`isMemberOfGroupNameRegex` | Boolean | `isMemberOfGroupNameRegex("/.*admin.*")` | **True**, if the user under consideration is a member of any groups with names that contain *admin*; otherwise,  **False**.

### Time Functions

Function  | Input Parameter Signature | Return Type | Example | Output
--------- | ------------------------- | ----------- | ------- | -------
`Time.now`  | (String timeZoneId, String format) | String      | `Time.now()` | 2015-07-31T17:18:37.979Z (Current time, UTC format)
| | | `Time.now("EST")` | 2015-07-31T13:30:49.964-04:00 (Specified time zone)
| | | `Time.now("EST", "YYYY-MM-dd HH:mm:ss")` | 2015-07-31 13:36:48 (Specified time zone and format, military time)
`Time.fromWindowsToIso8601`|(String time)|String|Windows timestamp time as a string (Windows/LDAP timestamp doc)|The passed-in time expressed in ISO 8601 format (specifically the RFC 3339 subset of the ISO standard).
`Time.fromUnixToIso8601`|(String time)|String|Unix timestamp time as a string (Unix timestamp reference)|The passed-in time expressed in ISO 8601 format (specifically the RFC 3339 subset of the ISO standard).
`Time.fromStringToIso8601`|(String time, String format)|String|Timestamp time in a human-readable yet machine-parseable arbitrary format format (as defined by Joda time pattern)|The passed-in time expressed in ISO 8601 format (specifically the RFC 3339 subset of the ISO standard).
`Time.fromIso8601ToWindows`|(String time)|String|ISO 8601 timestamp time as a string|The passed-in time expressed in Windows timestamp format.
`Time.fromIso8601ToUnix`|(String time)|String|ISO 8601 timestamp time as a string|The passed-in time expressed in Unix timestamp format.
`Time.fromIso8601ToString`|(String time, String format)|String|ISO 8601 timestamp time, to convert to format using the same Joda time format semantics as fromStringToIso8601|The passed-in time expressed informat format.

>Both input parameters are optional for the Time.now function. The time zone ID supports both new and old style formats, listed below. The third example for 
the Time.now function shows how to specify the military time format.

##### Time Zone IDs

The following old style IDs are supported: GMT, WET, CET, MET, ECT, EET, MIT, HST, AST, PST, MST, PNT, CST, EST, IET, PRT, CNT, AGT, BET, ART, CAT, EAT, NET, PLT, IST, BST, VST, CTT, JST, ACT, AET, SST, NST.

The following new style IDs are supported: UTC, WET, CET, CET, CET, EET, Pacific/Apia, Pacific/Honolulu, America/Anchorage, America/Los_Angeles, America/Denver, America/Phoenix, America/Chicago,
America/New_York, America/Indiana/Indianapolis, America/Puerto_Rico, America/St_Johns, America/Argentina/Buenos_Aires, America/Sao_Paulo, Africa/Cairo,
Africa/Harare, Africa/Addis_Ababa, Asia/Yerevan, Asia/Karachi, Asia/Kolkata, Asia/Dhaka, Asia/Ho_Chi_Minh, Asia/Shanghai, Asia/Tokyo, Australia/Darwin,
Australia/Sydney, Pacific/Guadalcanal, Pacific/Auckland.

### Manager/Assistant Functions

Function  | Description | Example
--------- | ----------- | -------
`getManagerUser(managerSource).$attribute` | Gets the manager's Okta user attribute values | `getManagerUser("active_directory").firstName`
`getManagerAppUser(managerSource, attributeSource).$attribute` | Gets the manager's app user attribute values for the app user of any appinstance. | `getManagerAppUser("active_directory", "google").firstName`
`getAssistantUser(assistantSource).$attribute` | Gets the assistant's Okta user attribute values. | `getAssistantUser("active_directory").firstName`
`getAssistantAppUser(assistantSource, attributeSource).$attribute` | Gets the assistant's app user attribute values for the app user of any appinstance. | `getAssistantAppUser("active_directory", "google").firstName`

> Pass the correct **app name** for the *managerSource*, *assistantSource*, and *attributeSource* parameters.<br />
> Note: At this time, only **active_directory** is supported for *managerSource* and *assistantSource*.

### Directory and Workday Functions

Function  | Description
-------- | ---------
`hasDirectoryUser()` | Checks whether the user has an Active Directory assignment and returns a boolean
`hasWorkdayUser()` | Checks whether the user has a Workday assignment and returns a boolean
`findDirectoryUser()` | Finds the Active Directory App user object and returns that object, or null if the user has more than one or no Active Directory assignments
`findWorkdayUser()` | Finds the Workday App user object and returns that object, or null if the user has more than one or no Active Directory assignments

The functions above are often used in tandem to check whether a user has an AD or Workday assignment, and if so return an AD or Workday attribute.  See the 'Popular Expressions' table below for some examples.

## Constants and Operators

Common Actions  | Example
----------------| --------
Refer to a `String` constant | `'Hello world'`
Refer to a `Integer` constant | `1234`
Refer to a `Number` constant | `3.141`
Refer to a `Boolean` constant | `true`
Concatenate two strings | `user.firstName + user.lastName`
Concatenate two strings with space | `user.firstName + " " + user.lastName`
Ternary operator example:<br>If group code is 123, assign value of Sales, else assign Other | `user.groupCode == 123 ? 'Sales' : 'Other'`

## Popular Expressions

Sample user data:

* Firstname = Winston
* Lastname = Churchill
* Email = winston.churchill@gmail.com
* Login = winston.churchill@gmail.com

Value to Obtain | Expression  | Example Output | Explanation
---------- | ---- | ----- | ---------------
Firstname | `user.firstName` | Winston | Obtain the value of users' firstname attribute.
Firstname + Lastname | `user.firstName + user.lastName` | WinstonChurchill | Obtain Firstname and Lastname values and append each together.
Firstname + Lastname with Separator | `user.firstName + "." + user.lastName` | Winston.Churchill | Obtain Firstname value, append a "." character. Obtain and append the Lastname value.
First Initial + Lastname | `substring(user.firstName, 0, 1) + user.lastName` | WChurchill | Obtain Firstname value. From result, retrieve characters greater than position 0 thru position 1, including position 1. Obtain and append the Lastname value.
First Initial + Lastname with Limit | `substring(user.firstName, 0, 1) + substring(user.lastName, 0, 6)` | WChurch | Obtain Firstname value. From result, retrieve 1 character starting at the beginning of the string. Obtain Lastname value. From result, retrieve characters greater than position 0 thru position 6, including position 6.
Lower Case First Initial + Lower Case Lastname with Separator | `toLowerCase(substring( user.firstName, 0, 1)) + "." + toLowerCase(user.lastName)` | w.churchhill | Obtain Firstname value. From result, retrieve characters greater than position 0 thru position 1, including position 1. Convert result to lowercase. Append a "." character. Obtain the Lastname value. Convert to lowercase and append.
Email Domain + Email Prefix with Separator | `toUpperCase(substringBefore( substringAfter(user.email, "@"), ".")) + "\" + substringBefore( user.email, "@")` | GMAIL\winston.churchill | Obtain Email value. From result, parse everything after the "@ character". From result, parse everything before the "." character. Convert to uppercase. Append a backslash "\" character. Obtain the email value again. From result, parse for everything before the "@" character.
Email Domain + Lowercase First Initial and Lastname with Separator | `toUpperCase(substringBefore( substringAfter(user.email, "@"), ".")) + "\" + toLowerCase(substring( user.firstName, 0, 1)) + toLowerCase(user.lastName)` | GMAIL\wchurchill | Obtain Email value. From result, parse everything after the "@ character". From result, parse everything before the "." character. Convert to uppercase. Append a backslash "\" character. Obtain the Firstname value. From result, retrieve characters greater than position 0 thru position 1, including position 1. Convert it to lowercase. Obtain the Lastname value and convert it to lowercase.
Static Domain + Email Prefix with Separator | `"XDOMAIN\" + toLowerCase(substring( user.firstName, 0, 1)) + toLowerCase(user.lastName)` | XDOMAIN\wchurchill | Add "XDOMAIN" string. Append a backslash "\" character. Obtain the Firstname value. From result, retrieve characters greater than position 0 thru position 1, including position 1. Convert it to lowercase. Obtain the Lastname value. Convert it to lowercase.
Workday ID | `hasWorkdayUser() ? findWorkdayUser().employeeID : null` | 123456 | Check if user has a Workday assignment, and if so, return their Workday employee ID.
Active Directory UPN | `hasDirectoryUser() ? findDirectoryUser().managerUPN : null` | bob@okta.com | Check if user has an Active Directory assignment, and if so, return their Active Directory manager UPN.
