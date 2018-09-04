# Vello's OpenAPI Specification

This repository contains [OpenAPI specifications][openapi] for Vello's API.

Files can be found in the `openapi/` directory:

* `spec3.{json,yaml}:` OpenAPI 3.0 spec.

[openapi]: https://www.openapis.org/


## Examples for API v1

Vello identifies entities by using uuids.

### Authentication using OAuth

To authenticate communication between the servers we are using the Client Credentials Grant flow of OAuth.

```
POST /v1/oauth/token
Headers
  Content-Type: application/json
Body
{
  "grant_type": "client_credentials",
  "client_id": "CLIENT_ID",
  "client_secret": "LIENT_SECRET",
  "audience": "https://api.vello.fi/v1/"
}

```

The result will be an Access Token that can be used to make requests to the Vello API.

```
HTTP/1.1 200 OK
Content-Type: application/json
{
  "access_token": "eyJz93a...k4laUWw",
  "token_type":"Bearer",
  "expires_in":86400
}

```

### Retrieving list of offices near user


After authentication you can retrieve list of offices with resource & service information. Provide additional filters to limit the search result.

```
GET /v1/offices?country=FI&zipcode=90130&industry[]=barber
Headers:
  Content-Type: application/json
  Authorization: Bearer <token>

```

Returns list of matching offices for filters.

```
HTTP/1.1 200 OK
Content-Type: application/json
{
  "object": "list",
  "url": "/v1/offices",
  "next_batch": "...",
  "has_more": true,
  "data": [
    {
      "id": "off_12345",
      "object": "office",
      "title": "Kampaamo Salon Maija",
      "company": {
        "id": "com_12345",
        "object": "company",
        "name": "Kampaamo Salon Maija",
        "industry": "barber",
        "timezone": "Europe/Helsinki",
        "language": "fi"
      },
      "services": {
        "object": "list",
        "data": [
          {
            "id": "ser_12345",
            "object": "service",
            "title": "Hiustenleikkuu, naiset",
            "duration": 2700,
            "office": "off_12345",
            "company": "com_12345",
            "payment_price": 4500,
            "payment_currency": "EUR",
            "payment_required": false
          },
          {
            "id": "ser_23456",
            "object": "service",
            "title": "Hiustenleikkuu, miehet",
            "duration": 1800,
            "office": "off_12345",
            "company": "com_12345",
            "payment_price": 3500,
            "payment_currency": "EUR",
            "payment_required": false
          },
          {
            "id": "ser_34567",
            "object": "service",
            "title": "Värjäys",
            "duration": 3600,
            "office": "off_12345",
            "company": "com_12345",
            "payment_price": 8500,
            "payment_currency": "EUR",
            "payment_required": false
          }
        ],
        "has_more": false,
        "url": "/v1/office/off_12345/services"
      },
      "resources": {
        "object": "list",
        "data": [
          {
            "id": "...",
            "object": "resource",
            "title": "Maija Kampaaja",
            "office": "off_12345",
            "company": "com_12345"
          }
        ],
        "has_more": false,
        "url": "/v1/office/off_12345/resources"
      }
    }
  ]
}

```

### Resource availability

When service is selected, we can provide suggestions for next available times for resources.

```
GET /v1/resources_availability?service=...&resource[]=...&from=123&to=123&limit=10
Headers:
  Content-Type: application/json
  Authorization: Bearer <token>

```

The result is list of suggested times

```
HTTP/1.1 200 OK
Content-Type: application/json
{
  "object": "list",
  "url": "/v1/office/off_12345/resources_availability",
  "data": [
    {
      "id": "res_12345",
      "object": "resource",
      "available_times": [
        {
          "id": "all_12345",
          "service": "ser_12345",
          "start": 123450000,
          "duration": 2700,
          "type": "single"
        },
        {
          "id": "all_12345",
          "service": "ser_12345",
          "start": 123460000,
          "duration": 2700,
          "type": "single"
        },
      ],
      "company": "com_12345",
      "office": "off_12345",
    }
  ],
  "next_batch": "...",
  "has_more": true
}

```

### Booking a time for customer

To book selected time you need to write record with required information:

```
POST /v1/records
Headers
  Content-Type: application/json
  Authorization: Bearer <token>
Body
{
  "company": "com_12345",
  "office": "off_12345",
  "resource": "res_12345",
  "service": "ser_12345",
  "allocation": "all_12345",
  "time": 123450000,
  "duration": 2700,
  "type": "single",
  "details": {
    "name": "Matti Meikälainen"
    "email": "matti.meikalainen@vello.fi",
    "phone": "+35845123456"
  },
  "tos": 123456789
}

```

If everything went well, we will return a newly created record object for you:


```
HTTP/1.1 200 OK
Content-Type: application/json
{
  "id": "rec_12345",
  "object": "record",
  "company": "com_12345",
  "office": "off_12345",
  "resource": "res_12345",
  "service": "ser_12345",
  "allocation": "all_12345",
  "customer": "cus_123456",
  "user": null,
  "time": 123450000,
  "duration": 2700,
  "type": "single",
  "details": {
    "name": "Matti Meikälainen"
    "email": "matti.meikalainen@vello.fi",
    "phone": "+35845123456"
  },
  "group": null,
  "record_number": "rec_12345",
  "payment_paid": false,
  "language": "fi",
  "allocated_time_start": 123445000,
  "allocated_time_end": 123452300,
  "confirmation_status": "confirmed",
  "created_by": null,
  "tos": 123456789,
  "created": 123400000,
  "changed": 123400000,
  "version": 1
}

```

To update or cancel existing record as anonymous user, you need to know `record id` and `record number` from original record. Note that you can patch only fields that are not protected, eg. language and details.

```
PUT /v1/record/rec_12345?record_number=rec_12345
Headers
  Content-Type: application/json
  Authorization: Bearer <token>
Body
{
  "language": "se"
}

```

Result will be updated record. Protected fields like `changed` and `version'` are managed and updated by the system automatically.

```
HTTP/1.1 200 OK
Content-Type: application/json
{
  "id": "rec_12345",
  "object": "record",
  "company": "com_12345",
  "office": "off_12345",
  "resource": "res_12345",
  "service": "ser_12345",
  "allocation": "all_12345",
  "customer": "cus_123456",
  "user": null,
  "time": 123450000,
  "duration": 2700,
  "type": "single",
  "details": {
    "name": "Matti Meikälainen"
    "email": "matti.meikalainen@vello.fi",
    "phone": "+35845123456"
  },
  "group": null,
  "record_number": "rec_12345",
  "payment_paid": false,
  "language": "se",
  "allocated_time_start": 123445000,
  "allocated_time_end": 123452300,
  "confirmation_status": "confirmed",
  "created_by": null,
  "tos": 123456789,
  "created": 123400000,
  "changed": 12345678,
  "version": 2
}

```

### Record payment

Record payment status is one of the protected fields and can be updated only programmatically. If customer is able to pay booking forehand within your application, you need to provide payment information after record is created. This feature requires that your OAuth Customer credentials are permitted to access records payment data.

Following example shows how to provide payment data for your record. Payment value is incremential. In the future we plan to support also partial payments. This feature is required if user has vouchers or uses multiple payment methods to complete the full payment. Record `payment_paid` field is updated to `true` after a full payment.

```
POST /v1/record/rec_12345/payment?record_number=rec_12345
Headers
  Content-Type: application/json
  Authorization: Bearer <token>
Body
{
  "method": "[custom_payment_method]",
  "payment_value": 4500,
  "payment_currency": EUR
}

```

Result is:

```
HTTP/1.1 200 OK
Content-Type: application/json
{
  "status": true,
  "current_value": {
    "payment_paid": true,
    "payment_value": 4500,
    "payment_currency": "EUR"
  },
  "previous_value": {
    "payment_paid": false,
    "payment_value": 0,
    "payment_currency": "EUR"
  }
}

```
