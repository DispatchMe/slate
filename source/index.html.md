---
title: Dispatch REST API v3

language_tabs:
  - json

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the Dispatch REST API documentation. Our REST API allows you to easily interact with our core business entities to implement your workflow in our platform.

# Authentication

Our API is secured using simple OAuth2.0 bearer tokens. You can get a bearer token for your account by providing your **public** and **secret** keys that we will provide upon signing up for the platform.

## Retrieving a Bearer Token

> Request:

```json
{
  "grant_type": "client_credentials",
  "client_id": "public_key",
  "client_secret": "secret_key"
}
```

> Response:

```json
{
  "access_token": "bearer token value",
  "token_type": "bearer",
  "created_at": 1499868367,
  "expires_in": 10800,
  "refresh_token": "refresh token value"
}
```

`POST /v3/oauth/token`

**These tokens are temporary**, so you will need to be able to handle 401 status codes from our system to retrieve a new token in the event that it is no longer valid.

## Refreshing your Token

> Request:

```json
{
  "grant_type": "refresh_token",
  "refresh_token": "refresh token you received with your original bearer token"
}
```

> Response:

```json
{
  "access_token": "bearer token value",
  "token_type": "bearer",
  "created_at": 1499868367,
  "expires_in": 10800,
  "refresh_token": "refresh token value"
}
```

`POST /v3/oauth/token`

## Access Control & Permissions

Describe third-party vs first-party, data ownership, etc.

# Filtering and Paginating
For `GET` requests, several endpoints allow you to filter the results based on certain record attributes. These are all provided as nested values on a `filter` object within the query string, like so: `?filter[job_id]=123&filter[status]=scheduled`

For all `GET` requests that return multiple records, you can paginate by providing `limit` and `offset` parameters, like so (the maximum value for `limit` is currently 100): `?filter[job_id]=123&filter[status]=scheduled&limit=100&offset=1000`

# Attachments

Attachments represent notes assigned to [jobs](#jobs) in the Dispatch platform. These are useful for providing additional information to the service provider after they have already begun work.

## Create an Attachment
> Request:

```json
{
  "entity_type": "Job",
  "entity_id": 123,
  "description": "Here is the note"
}
```

> Response:

```json
{
  "attachment": {
    "id": 1,
    "entity_type": "Job",
    "entity_id": 123,
    "description": "Here is the note",
    "created_at": "2017-01-01T00:00:00Z",
    "updated_at": "2017-01-01T00:00:00Z"
  }
}
```

`POST /v3/attachments`

## List Attachments

> Response:

```json
{
  "attachments": [
    {
      "id": 1,
      "entity_type": "Job",
      "entity_id": 123,
      "description": "Here is a note"
    }
  ],
  "meta": {
    "limit": 200,
    "offset": 0,
    "total": 1
  }
}
```

`GET /v3/attachments`

### Query Parameters
Parameter | Description
--------- | -----------
entity_type_eq | Type of parent object (currently only supported value is "Job")
entity_id_eq | ID of the parent object

### Example: view all attachments on job #123

`GET https://api.dispatch.me/v3/attachments?filter[entity_type_eq]=Job&filter[entity_id_eq]=123`

## View a Single Attachment
> Response:

```json
{
  "id": 1,
  "entity_type": "Job",
  "entity_id": 123,
  "description": "Here is a note"
}
```

`GET /v3/attachments/:id`

## Update Attachment
> Request:

```json
{
  "description": "New note here"
}
```

> Response:

```json
{
  "attachment": {
    "id": 1,
    "entity_type": "Job",
    "entity_id": 123,
    "description": "New note here",
    "created_at": "2017-01-01T00:00:00Z",
    "updated_at": "2017-01-01T00:00:00Z"
  }
}
```

`PATCH /v3/attachments/:id`

## Delete Attachment
`DELETE /v3/attachments/:id`

# Appointments

Appointments are scheduled times at which a technician will do work related to a [job](#jobs). Jobs may have zero or more appointments (whether appointments are used depends upon your workflow and the engagement of your service providers on our platform).

Appointments are required to send customers notifications for "on my way" and "appointment scheduled".

## Appointment Attributes
Attribute | Type | Description
--------- | ---- | -----------
job_id | int | ID of the parent job
time | string | ISO8601 timestamp for the appointment's scheduled time
duration | int | duration, in seconds, of the appointment. Defaults to `7200` (2 hours)
user_id | int | ID of the assigned technician
status | string | Status of the appointment. See [Appointment Statuses](#appointment-statuses)

## Create an Appointment

> Request

```json
{
  "job_id": 123,
  "user_id": 456,
  "status": "scheduled",
  "time": "2017-01-01T00:00:00Z",
  "duration": 3600
}
```

> Response

```json
{
  "appointment": {
    "id": 2,
    "job_id": 123,
    "organization_id": 3,
    "status": "scheduled",
    "time": "2017-01-01T00:00:00Z",
    "duration": 3600,
    "user_id": 456
  }
}
```

`POST /v3/appointments`

## List Appointments

> Response

```json
{
  "appointments": [
    {
      "id": 2,
      "job_id": 123,
      "organization_id": 3,
      "status": "scheduled",
      "time": "2017-01-01T00:00:00Z",
      "duration": 3600,
      "user_id": 456
    }
  ],
  "meta": {
    "total": 1,
    "limit": 200,
    "offset": 0
  }
}
```

`GET /v3/appointments`

### Query Parameters
Parameter | Description
--------- | -----------
job_id_eq | Parent job ID
job_id_in | Parent job IDs (separated by comma)
organization_id_eq | Parent organization Id
status_eq | Search for appointments in a specific status
status_in | Specific statuses (separated by comma)
time_gt | Show appointments scheduled after the given time (ISO8601 timestamp)
time_gteq | Show appointments scheduled on or after the given time (ISO8601 timestamp)
time_lt | Show appointments scheduled before the given time (ISO8601 timestamp)
time_lteq | Show appointments scheduled on or before the given time (ISO8601 timestamp)
user_id_eq | Show appointments assigned to a particular technician
user_id_in | Show appointments assigned to technicians (separated by comma)
user_id_null | Show appointments that are unassigned (value should be `true` here)

## View a Single Appointment
> Response

```json
{
  "id": 2,
  "job_id": 123,
  "organization_id": 3,
  "status": "scheduled",
  "time": "2017-01-01T00:00:00Z",
  "duration": 3600,
  "user_id": 456
}
```

`GET /v3/appointments/:id`

## Update Appointment

> Request

```json
{
  "time": "2017-01-02T00:00:00Z",
  "duration": 7200,
  "user_id": 789
}
```

> Response

```json
{
  "appointment": {
    "id": 2,
    "job_id": 123,
    "organization_id": 3,
    "status": "scheduled",
    "time": "2017-01-02T00:00:00Z",
    "duration": 7200,
    "user_id": 789
  }
}
```

`PATCH /v3/appointments/:id`

## Delete Appointment

`DELETE /v3/appointments/:id`

# Brands
Brands represent different brands or divisions within your company, giving you full control over the copy and logo for the mobile and web applications, customer notifications, etc. Logos and job source details fall back to the account level if there is no `brand_id` property present on a job.

## Create a Brand
> Request

```json
{
  "name": "New Brand",
  "phone_number": "+15551234567",
  "logo_token": "9b553add-62cf-47c0-aec1-6705e395b4da"
}
```

`POST /v3/brands`

For the `logo_token` attribute, use the `uid` attribute returned when you [upload your brand logo](#upload-photo).

## List Brands
> Response

```json
{
  "brands": [
    {
      "id": 1,
      "name": "Brand #1",
      "logo_token": "7961914c-7e8f-4012-a316-be00e7b1a40e",
      "phone_number": "+15559876543"
    },
    {
      "id": 2,
      "name": "Brand #2",
      "logo_token": "2d287898-936f-4820-964f-088f2c977336",
      "phone_number": "+15551111111"
    }
  ],
  "meta": {
    "limit": 200,
    "offset": 0,
    "total": 2
  }
}
```

`GET /v3/brands`

<aside class="notice">
There are no query parameters available for brands at the moment.
</aside>

## Update Brand
> Request

```json
{
  "name": "Changed Name"
}
```

> Response
```json
{
  "brand": {
    "id": 1,
    "name": "Changed Name",
    "logo_token": "7961914c-7e8f-4012-a316-be00e7b1a40e",
    "phone_number": "+15559876543"
  }
}
```

`PATCH /v3/brands/:id`

## Delete Brand
<aside class="notice">
Careful! If you delete a brand then all jobs currently within that brand will use your default "account-level" branding.
</aside>

`DELETE /v3/brands/:id`

# Customers
Customers represent the homeowner, landlord or other entity the work is being done for. Customers in Dispatch belong to individual organizations, so if you have multiple organizations doing work for the same customer in your system, there will be multiple instances of that customer in Dispatch. We do this to allow organizations to keep their own notes and make their own updates to customer data without affecting other organizations. 

# Files and Photos
You can upload photos and other files to share with service providers and customers using our `/v3/datafiles` endpoint.

## <a name="upload-photo"></a>Uploading a file
> Request

```curl
curl -H "Authorization: Bearer <token>" -F file=@test.png https://api.dispatch.me/v3/datafiles

```

> Response

```json
{
  "datafile": {
    "bucket_id": null,
    "created_at": "2017-07-19T01:42:48Z",
    "filename": "test.png",
    "mime_type": "image/png",
    "name": "",
    "uid": "fe9194b3-3cc2-4862-af52-d8b59f7dd062"
  }
}
```

`POST` using `Content-Type: multipart/form-data` to `/v3/datafiles`

## View a file by UID
> Response

```
HTTP/1.1 302 Found
Location: https://s3.amazonaws.com/dispatch_staging/datafiles/fe9194b3-3cc2-4862-af52-d8b59f7dd062/test.png
```

`GET /v3/datafiles/:uid`

# Jobs

# Service Providers

# Survey Responses

# Users


