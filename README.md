# NetSuite Connect

NetSuite Connect implements the following features to enable an external application to work with a NetSuite Account:

- Supports creating and sending signed requests from an external application to NetSuite using Token-Based Authentication.

- Provides an API for discovering, aggregating and accessing custom endpoints (implemented as RESTlets in NetSuite) within the NetSuite account.

## Background

I originally wrote a custom OAuth script based on code found in the [chanahc/phpoauth](https://github.com/ChanahC/phpoauth) GitHub repo. This script handled creating the necessary authorization headers based on pre-issued credentials to access a NetSuite account endpoint. This early version was difficult to maintain, and a glaring limitation was its use of the HMAC-SHA1 algorithm to sign requests as outlined in NetSuite's SuiteSupport documentation. However, using SHA1 for encryption has been a known security concern for a few years. Unsurprisingly, NetSuite announced they would deprecate support for the HMAC-SHA1 algorithm beginning with Release 2021.1 and drop support completely by Release 2021.2 in favor of using the HMAC-SHA256 algorithm. This announcement, along with the need to create a more maintainable authorization and connection management service prompted the creation of the NetSuite Connect package has been considered NetSuite's announcements NetSuite's Token-Based Authorization uses a variation of the OAuth 1.0 workflow to support authentication and authorization to its resources via "signed" requests. NetSuite's authentication strategy bears many similarities to the parameters and processes used to sign requests using AWS Signature Version 4. Although, I've been unsuccessful in leveraging the AWS SDK for this purpose, this package implements many of the same interfaces as AWS.

## Installation

```shell
$ composer require brazilliance/netsuite-connect
````

## NetSuite's OAuth 1.0 Implementation

NetSuite requires the creation of an Integration Record and User Generated Tokens in order to set up an authorized connection to a custom endpoint. The credentials include the following:

### Client Credentials

- Application ID – A value that uniquely identifies a single pathway through which an external application can connect with NetSuite. It's a good idea to create separate Integration Records for each service intended to connect with NetSuite as these are used in various logging facilities to identify events.
- Consumer Key – A randomly generated string that uniquely identifies the "integration" through which the external service can connect to NetSuite.
- Consumer Secret – A randomly generate string that is combined with a Token Secret to create a Signature used to sign a Request.

#### What is Signing?

"Signing" is a repeatable process that a server can use to ensure that an incoming request originated from an authorized User (or a Service operating on a User's behalf) is legitimate.

#### The 3-Legged Authorization Flow

In the standard, 3-Legged Authorization Flow, Client Credentials — provided to the user in advance or by way of a specifically designed endpoint — would be presented to NetSuite and, assuming they are valid (match an Integration Record that provisioned them) would be exchanged for a temporary payload known as a Request Token. These Temporary/Request Credentials would then be exchanged for the final Token Credentials once the User has approved that they want to give the requesting application access to NetSuite (typically done by signing in to NetSuite via a login screen that is presented on behalf of the external application). The external application would then be responsible for securely persisting the Token Credentials and including them with every future request to the Service. The Service, in turn, has stored on its system that requests received from a User/Client that includes these Token Credentials is authorized to access the information they are requesting.

This Request Token to Access Token exchange is the point in the traditional OAuth flow that NetSuite breaks from the specification (although it does support 3-Legged Auth). Instead, for server-to-server integrations, NetSuite expects a User to generate the Token Credentials that they would normally be given in the traditional flow. These Token Credentials are bound to an Integration Record.
  
### Token/Access Credentials

- Token/Access ID – Generated by a User in the NetSuite UI and bound to an existing Integration Record.
- Token/Access Secret – A randomly generated string that is combined with the Consumer Secret to create a Signature used to sign a Request.

## Usage

To use NetSuite Connect, you must create a new instance and pass to it an array of Account Details and Credentials like in the following:

```php
$nsConnect = new Service\NetSuite([
    'account.id' => 'netsuite-account-id',
    'client.key' => 'consumer-key',
    'client.secret' => 'consumer-secret',
    'access.key' => 'token-id',
    'access.secret' => 'token-secret'
]);
```

Once instantiated, the `$nsConnect` object can be used to access all the available endpoints defined in the NetSuite account.

You can see an index of all available endpoints by accessing the `getEndpoints` method:

```php
$nsConnect->getEndpoints();
```