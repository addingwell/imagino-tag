# Imagino Tag (server-side) By Addingwell

# Implementation
- Fill your Account key (API key provided by Imagino)
- Fill your Customer identifier (provided by Imagino)
- Add the authorized origin(s) (Optional — by default, all incoming origins are allowed)
- Fill the Website host (Optional — see below)
- Add Event type (For example, "product seen")
- Add Event value (For example, the SKU of the product)
- Fill your custom data (Optional)
- Fill the userId (Optional)

The Tag set automatically 2 cookies for each visitor : 
- `_imo_browser_id` set for 13 months
- `_imo_session_id` set for the session

## Website host

Imagino rejects the event when the `host` of the payload is not the hostname of the
calling website whitelisted with the API key (`NOK: site - Host is not whitelisted`).
In a first-party server-side setup, the `Host` header of the incoming request is the
hostname of the tagging server (for example `sgtm.mydomain.com`), not the one of the
website, so the tag resolves the `host` in this order:

1. the `Website host` field of the tag, if filled
2. `page_hostname` of the incoming event
3. the hostname of `page_location` of the incoming event
4. the `Host` header of the incoming request

## User id

The `userId` is required by Imagino to reconcile users. It is taken from the `User id`
field of the tag if filled, otherwise from the `user_id` of the incoming event (the key
sent by the GA4 client). It is never sent empty.

## Authorized origins

When at least one origin is configured, the `Origin` header of the incoming request must
match one of them, otherwise the event is not sent. Requests without an `Origin` header
(for example beacons) are always allowed.

# Imagino Tag - Server Side SDK description

The imagino Tag server side SDK consists of one API endpoint to be called.
POST https://#CUSTOMER-IDENTIFIER#.tag.imagino.com/#API_KEY#/events

The following payload must be respected:
```JSON
{
    "data": {
      "event": {
        "sku": "string"
      }, 
      "userOptions": {
        "utm_source": "string"
      }
    },
    "deviceId": "80faad33-3ce1-400c-88aa-939c2ad8058a",
    "sessionId": "a8c36ddf-88ee-4e3c-8f0e-e516f80b551d",
    "userId": "12345",
    "eventType": "string",
    "eventValue": "12345",
    "host": "www.imagino.com"
}
```

Below is a description of the different payload variables.

| Key              | Type           | Required | Description                                                                                               |
|------------------|----------------|----------|-----------------------------------------------------------------------------------------------------------|
| host             | String         | Required | URL of the calling website. Needs to be whitelisted and associated with the tag API key.                  |
| sessionId        | String         | Required | Id of the session. Needs to be generated server side.                                                     |
| deviceId         | String         | Required | Id associated with the device or browser.                                                                 |
| eventType        | String         | Required | Name of the event. For exemple, product seen.                                                                                       |
| eventValue       | String, Number | Required | Value associated with the event. For exemple, the SKU of the product.                                                                         |
| userId           | String, Number | Optional | Id associated with the user. Required for Imagino to reconcile users.                                     |
| data.event       | Object         | Optional | Json object contaning the value associated with the event. For exemple, product price, categorie or name. |
| data.userOptions | Object         | Optional | Json object contaning the value associated with the user. For exemple, name or UTM values.                |

The payload is sent as `application/json`.

The Imagino Tag API always responds with a code 200. 
- On success, API responds with `OK`.
- On error, API responds with `NOK` and the error description.
    - `NOK: no referrer - Host is empty`
    - `NOK: site - Host is not whitelisted`
    - `NOK: <json cannot be unserilzed> - Payload does not respect the format NOK: unauthorized - Wrong API key`
    - `NOK: event - Error while processing the event`
