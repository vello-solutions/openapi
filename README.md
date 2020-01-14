# Vello Events and Webhooks

Events makes it possible to let you know when something happens in your account.
When an event occurs, we create an `Event` object. This happens for example
when someone makes reservation eg. creates a `Record` object.

Vello can notify your application service when something happens using webhooks
by sending `Event` objects to an endpoint on your server. This way webhooks can
be used to create things like external analytics and automation for business
tasks.

Note: To enable this feature, you need to sign in to our Beta program.

Webhooks are POST-requests to your endpoint.
Following example shows `record.created` webhook data:

```
POST to https://app.yourservice.com/webhooks/vello

Headers
  Content-Type: application/json
  Vello-Signature: <signature>
Body
{
  "id": "[UUID]",
  "type": "record.created",
  "data": {
    "object": {
      "id": "[RECORD UUID]",
      ...
    }
  }
}

Response

HTTP/1.1 200 OK
```

You should always return a `200 OK` response to webhook callbacks. If request fails, Vello will
try to resend the request for a while.

## Webhook signatures

All webhook events that are sent yo your endpoints are signed by Vello using
your endpoint"s secret token. Signature is included in each event"s
Vello-Signature header.

You can verify the request body using signature and by checking SHA-512 hash.

Following example is for NodeJS. Result should be compared to Vello-Signature header.

```
const input = webhookToken + endpointUrl + body
const hash = crypto.createHash("sha512").update(input).digest("hex");
```

## List of interesting `Event` object types:

We can send inform you about all kind of changes in event system, but these are
the most used ones:

`connection.test` - Connection test event.

`record.created` - Record is just created.

`record.updated` - Record is updated.

`record.cancelled` - Record is cancelled.

`record.deleted` - Record is deleted.

`record.update_metadata` - Dynamic webhook to provide metadata. See example below.

## Implementing and testing webhook connection

To implement webhooks you should sign in to our Beta program. After this you can
configure your application services webhook endpoint to your Vello account.

We will also provide you the secure token keys to interact with Vello.

To test webhook connection you can request Vello to send you a `connection.test`
webhook event from UI. We will send you following event:

```
POST to https://app.yourservice.com/webhooks/vello

Headers
  Content-Type: application/json
  Vello-Signature: <signature>

Body
{
  "id": "[UUID]",
  "type": "connection.test",
  "data": {}
}

Response

HTTP/1.1 200 OK
```

## Dynamic webhook events

Some webhook events are used to provide additional data for Vello. For example,
if you want to update `Record` object on the fly to assign a Video conference link
to your meeting, implement `record.update_metadata` webhook:

In this case you should always return `200 OK` and `metadata` object to be assigned
to `Record` object"s `metadata` field.

```
POST to https://app.yourservice.com/webhooks/vello

Headers
  Content-Type: application/json
  Vello-Signature: <signature>
Body
{
  "id": "[UUID]",
  "type": "record.update_metadata",
  "data": {
    "id": "639e9909-db32-4b5b-8159-42e1cabd7f24",
    "object": "record",
    "company": "90d491a1-50f9-42da-8c3a-9b0683f4a10d",
    "office": "1aeef479-7c49-40e2-9ab6-09fd454fa226",
    "resource": "f58c2d1e-b77f-439f-b7e5-d43c3265c65a",
    "service": "74df9d2b-4ba3-45ef-a2be-30f87cf52e81",
    "allocation": "685f7d57-cb01-4ca7-8f4e-882bf2d7ad84",
    "user": "fbb70166-608d-49b9-b207-ab5c4197df4b",
    "customer": "686ff2eb-36dd-450d-a03b-de859af80240",
    "record_number": "b264c6ff-a8fc-4",
    "quantity": 1,
    "group": null,
    "properties": null,
    "language": "fi",
    "time": 1578952635,
    "duration": 1800,
    "allocated_time_start": 1578952635,
    "allocated_time_end": 1578954435,
    "payment_paid": false,
    "confirmation_status": "confirmed",
    "details": {},
    "metadata": {},
    "ip": "",
    "created_by": null,
    "tos": 1578952635,
    "canceled": false,
    "archived": false,
    "created": 1578952635,
    "changed": 1578952635,
    "version": 1
  }
}
```

Your response to this requst should contain the actual metadata:

```
Response

HTTP/1.1 200 OK
Content-Type: application/json
{
  "metadata": {
    "video_link": "https://app.yourservice.com/meeting/12345"
  }
}
```

At the moment you can provide following metadata keys: `video_link` and `tag`.
