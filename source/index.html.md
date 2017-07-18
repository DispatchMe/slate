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

## View Attachments

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

## View Appointments

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
`

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

# Customers

# Jobs

# Service Providers

# Survey Responses

# Users


