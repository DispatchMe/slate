---
title: Dispatch REST API v3

language_tabs:
  - json

toc_footers:
  - <a href="http://www.dispatch.me" target="_blank">Dispatch Website</a>

includes:
  - errors

search: true
---

# Getting Started

Welcome to the Dispatch REST API documentation. Our REST API allows you to easily interact with our core business entities to implement your workflow in our platform.

## Who should use this API?
* Job sources and lead generation platforms for third-party networks of service providers (e.g. warranty companies, equipment manufacturers)
* Service enterprises with multiple branches in a first-party network
* Individual service providers who receive work from these sources

Job sources, regardless of network type, must authenticate using [client credentials authentication](#client-credentials-auth), which will provide access to objects assigned to all service providers in the network. Service providers must authenticate using [user authentication](#user-auth).

## Environments
We have two environments for public access to the API.

Our **sandbox** environment is meant for you to test your integration before putting it into the wild. That API is located at https://api-sandbox.dispatch.me, and you can access our desktop application at https://work-sandbox.dispatch.me.

Our **production** environment is for real jobs and customers. That API is located at https://api.dispatch.me, and you can access our desktop application at https://work.dispatch.me.

Note that your account manager will give you a different set of credentials for each environment.

## Signing Up

If you're interested in using our API, please work with your account manager and we'll set you up with your credentials. Don't have an account manager? Provide your information here and we'll get back to you.

## Dispatch Business Objects

Here's a brief description of our business objects and their role in the Dispatch system.

Entity | Description
------ | -----------
[Job](#jobs) | A single body of work for a [customer](#customers), assigned to an [organization](#organization).
[Customer](#customers) | A job currently belongs to a single customer, representing the homeowner.
[Organization](#organizations) | Jobs are assigned to a single organization, who is responsible for doing the work. Depending on your business model, this could be a branch of your business, or a third-party service provider you have an agreement with.
[Appointment](#appointments) | Scheduled times at which an assigned [user](#users) (technician) will perform the work for the job
[Attachment](#attachments) | Attachments are notes or photos that you can add to jobs to provide the dispatchers and technicians with additional information. They can also add their own notes via our mobile or desktop application
[User](#users) | Users are the users of our application. Currently we have 2 types of users: the **dispatcher** and the **technician**. Users can also have both roles if it makes sense for your use case. We have separate mobile applications for each role, as well as a single desktop web application for dispatchers to manage their workforce from the office.
[Survey Response](#survey-responses) | Surveys are sent out to customers when an appointment or job is completed. 
[Brand](#brands) | You can assign jobs to your brands to control the branding (logo, copy, etc) of the application, customer portal, and notifications.

## <a name="external-ids"></a>External IDs

To make it easier for you to integrate your existing system with Dispatch, several of our business objects have an attribute called `external_ids`. This is an array of strings representing the ID or ID(s) of those objects in your own system. We use those for various purposes, such as:

* Identifying existing customers or organizations, instead of creating new ones, so you don't have to look them up before attempting to create
* Allowing you to look up entities using your ID instead of ours.

We do, however, recommend that you store Dispatch's business object ID for your objects somewhere in your system, since our external IDs support is limited.

<aside class="warning">The external IDs feature is currently only available to job sources.</aside>

## Filtering and Paginating
For `GET` requests, several endpoints allow you to filter the results based on certain record attributes. These are all provided as nested values on a `filter` object within the query string, like so: `?filter[job_id]=123&filter[status]=scheduled`

For all `GET` requests that return multiple records, you can paginate by providing `limit` and `offset` parameters, like so (the maximum value for `limit` is currently 100): `?filter[job_id]=123&filter[status]=scheduled&limit=100&offset=1000`

## Getting Data Back from Dispatch
You can obviously check up on your data by performing `GET` requests, but we also allow you to subscribe to changes on your objects via **webhooks**. This feature is not self-service, but if you get in touch with your account manager we can walk you through the details. We hope to create a complete self-service webhook system sometime in the next few quarters.

# Authentication

Our API is secured using simple OAuth2.0 bearer tokens. You can get a bearer token for your account by providing your **public** and **secret** keys that we will provide upon signing up for the platform.

**These tokens are temporary**, so you will need to be able to handle 401 status codes from our system to retrieve a new token in the event that it is no longer valid.

## <a name="client-credentials-auth"></a> Retrieving a Bearer Token - Client Credentials

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

Use this method to authenticate as a job source.

`POST /v3/oauth/token`

## <a name="user-auth"></a> Retrieving a Bearer Token - User Credentials

> Request:

```json
{
  "grant_type": "password",
  "username": "your-email",
  "password": "your-password",
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

Use this method to authenticate a single service provider. We recommend you create a separate user account with a `"dispatcher"` role for your integration to easily be able to track which actions were taken by the integration vs actual users.

`POST /v3/oauth/token`


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

As the source of the work, you have full access to the transactional data you send to Dispatch (meaning, the job and all child objects). However, depending on the relationship you have with the organizations, they may already be in our system and may already be receiving work from other sources. In that case, you will have limited access to the organizations, their customers, and their dispatchers and technicians. If you have a first-party network, you will have more access to those objects. We recommend you discuss this with your account manager so we can implement the best ACL for your use case.

If you are authenticated as a single service provider, you will have access to all objects within your organization.

# <a name="attachments"></a> Attachments

Attachments represent notes or photos belonging to [jobs](#jobs) in the Dispatch platform. These are useful for providing additional information to the service provider after they have already begun work.

## Attributes
Attribute | Type | Required | Updatable | Description
--------- | ---- | -------- | --------- | -----------
entity_type | string | Y | N | Type of parent entity. Currently only "Job" is supported.
entity_id | int | Y | N | ID of the parent job
description | string | N | Y | Content of the note
file_token | string | N | Y | File token (from [files API](#files-photos))

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
  "attachment": {
    "id": 1,
    "entity_type": "Job",
    "entity_id": 123,
    "description": "Here is a note"
  }
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

# <a name="appointments"></a> Appointments

Appointments are scheduled times at which a technician will do work related to a [job](#jobs). Jobs may have zero or more appointments (whether appointments are used depends upon your workflow and the engagement of your service providers on our platform).

Appointments are required to send customers notifications for "on my way" and "appointment scheduled".

## Attributes
Attribute | Type | Required | Updatable | Description
--------- | ---- | -------- | --------- | -----------
job_id | int | Y | N | ID of the parent job
time | string | N | Y | ISO8601 timestamp for the appointment's scheduled time
duration | int | N | Y | duration, in seconds, of the appointment. Defaults to `7200` (2 hours)
user_id | int | N | Y | ID of the assigned technician
status | string | Y | Y | Status of the appointment. See [Appointment Statuses](#appointment-statuses)

## <a name="appointment-statuses"></a>Appointment Statuses
Appointments can move freely between any of the supported statuses below. Via our mobile app, technicians can update the status of the appointment according to your workflow.

Status | Description
------ | -----------
draft | A tentative appointment. May be missing information
scheduled | The appointment is scheduled
enroute | The technician is on the way
in_progress | The technician has arrived and has started the work
complete | The appointment is finished
canceled | The appointment has been canceled

### Special Status Behavior
* If you *schedule* an appointment, its parent job's status will also change to "scheduled"
* If you *cancel* a job, the status of all child appointments will change to "canceled"

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
    {}
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

# <a name="brands"></a> Brands
<aside class="notice">Brands are accessible only to job sources</aside>

Brands represent different brands or divisions within your company, giving you full control over the copy and logo for the mobile and web applications, customer notifications, etc. Logos and job source details fall back to the account level if there is no `brand_id` property present on a job.

## Create a Brand

> Request

```json
{
  "name": "New Brand",
  "phone_number": "+15551234567",
  "logo_token": "9b553add-62cf-47c0-aec1-6705e395b4da",
  "email": "newbrand@newbrand.com"
}
```

> Response

```json
{
  "brand": {
    "id": 123,
    "name": "New Brand",
    "phone_number": "+15551234567",
    "logo_token": "9b553add-62cf-47c0-aec1-6705e395b4da",
    "email": "newbrand@newbrand.com"
  }
}
```

`POST /v3/brands`

For the `logo_token` attribute, use the `uid` attribute returned when you [upload your brand logo](#upload-photo).

## List Brands
> Response

```json
{
  "brands": [
    {},
    {}
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
<aside class="warning">
Careful! If you delete a brand then all jobs currently within that brand will use your default "account-level" branding.
</aside>

`DELETE /v3/brands/:id`

# <a name="customers"></a> Customers
Customers represent the homeowner, landlord or other entity the work is being done for. Customers in Dispatch belong to individual organizations, so if you have multiple organizations doing work for the same customer in your system, there will be multiple instances of that customer in Dispatch. We do this to allow organizations to keep their own notes and make their own updates to customer data without affecting other organizations.

## Attributes
Note that currently we support **multiple phone numbers** but only a **single email address** for customers. We have plans to improve this in the near future.

Attribute | Type | Required | Updatable | Description
--------- | ---- | -------- | --------- | -----------
organization_id | int | Y | N | ID of the parent organization
first_name | string | Y | Y |
last_name | string | N | Y |
company_name | string | N | Y |
notes | string | N | Y | Any additional notes about this customer
email | string | N | Y | Email address for the customer. This will be used to send email notifications if they are configured for your account.
phone_numbers | array&laquo;[PhoneNumber](#phone-number-object)&raquo; | N | Y | See below for phone number object attributes
home_address | [Location](#location-schema) | N | Y | This is used only when creating a new job for this customer in our application. For jobs that come in via our API, the `job.address` property is used
billing_address | [Location](#location-schema) | N | Y |
external_ids | array&laquo;string&raquo; | N | Y | See [external IDs](#external-ids)

### <a name="phone-number-object"></a>Phone Number Object
Attribute | Type | Description
--------- | ---- | -----------
number | string | Phone number in [RFC3966 format](https://www.ietf.org/rfc/rfc3966.txt)
primary | boolean | Set to `true` if you want this number to receive SMS notifications
type | string | E.g. "Mobile", "Work"

## Create a Customer
> Request

```json
{
  "organization_id": 10,
  "email": "customer@email.com",
  "phone_numbers": [
    {
      "number": "+15551234567",
      "primary": true,
      "type": "mobile"
    },
    {
      "number": "+15559876543",
      "primary": false,
      "type": "work"
    }
  ],
  "first_name": "Joe",
  "last_name": "Shmo",
  "external_ids": ["AAA123"]
}
```

> Response

```json
{
  "customer": {
    "id": 11,
    "organization_id": 10,
    "email": "customer@email.com",
    "phone_numbers": [
      {
        "number": "+15551234567",
        "primary": true,
        "type": "mobile"
      },
      {
        "number": "+15559876543",
        "primary": false,
        "type": "work"
      }
    ],
    "first_name": "Joe",
    "last_name": "Shmo",
    "external_ids": ["AAA123"]
  }
}
```

`POST /v3/customers`

Note that when you attempt to create a customer, we may find a duplicate of that customer already in our system. In that case, the data you provide will be merged onto the existing customer rather than be used to create a new one. You will receive the existing customer back in the response.

## Update a Customer

> Request

```json
{
  "first_name": "Changed"
}
```

> Response

```json
{
  "customer": {
    "id": 11,
    "organization_id": 10,
    "email": "customer@email.com",
    "phone_numbers": [
      {
        "number": "+15551234567",
        "primary": true,
        "type": "mobile"
      },
      {
        "number": "+15559876543",
        "primary": false,
        "type": "work"
      }
    ],
    "first_name": "Changed",
    "last_name": "Shmo",
    "external_ids": ["AAA123"]
  }
}
```

`PATCH /v3/customers/:id`

## List Customers

> Response

```json
{
  "customers": [
    {},
    {}
  ],
  "meta": {
    "total": 2,
    "limit": 200,
    "offset": 0
  }
}
```

### Query Parameters
Parameter | Description
--------- | -----------
email_eq | Find a customer by email
external_id_contains | Find a customer with a given external ID
organization_id_eq | Find a customer for a specific organization

## Delete a Customer
`DELETE /v3/customers/:id`

# <a name="files-photos"></a> Files and Photos
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

# <a name="jobs"></a> Jobs
Jobs are the core of the Dispatch experience. The job object includes the service location, customer information, description, and other details needed so the technician can get the work done. 

## Attributes

Attribute | Type | Required | Updatable | Description
--------- | ---- | -------- | --------- | -----------
title | string | Y | Y |
description | string | N | Y | Additional details for the job. Supports [markdown](https://daringfireball.net/projects/markdown/syntax)
service_type | string | N | Y | Type of service, e.g. "plumbing"
external_ids | array&laquo;string&raquo; | N | N | Your ID(s) for this job. See [external-ids](#external-ids)
address | [Location](#location-schema) | Y | Y | [Location](#location-schema) of the job.
brand_id | int | N | N | Optional ID for your [brand](#brands)
customer_id | int | Y | Y | ID of the customer object.
organization_id | int | Y | N | ID of the assigned organization
service_fee | float | N | Y | Fee the customer owes for service
status | string | Y | Y | Status of the job. See [job statuses](#job-statuses)
status_message | string| N | Y | Optional qualifier for the current status

## <a name="job-statuses"></a>Job Statuses
Jobs can move freely between any of the supported statuses below.

Status | Description
------ | -----------
unscheduled | The job belongs to this organization but has not yet been scheduled
scheduled | The job is currently scheduled. Note that Dispatch automatically sets the job's status to "scheduled" if an appointment is created
paused | The job has started, but had to be paused
complete | The job has finished
canceled | The job is canceled - work no longer needs to be done.

## Job Offers
If your workflow includes **offering** the job to an organization and requiring them to accept or reject it, you can create a job with status set to "offered", which is a special status that has slightly different behavior in our applications. 

### <a name="accept-reject-behalf"></a>Accepting/Rejecting on behalf of Organization

> Accept Job w/ Appointment

```json
{
  "appointment": {
    "time": "ISO8601 Time",
    "duration": 3600,
    "user_id": 123
  }
}
```

Dispatchers can accept and reject job offers via our application. However, if you have another mechanism for job offer interaction, and want to keep the Dispatch system up to date, you can do so by making a `POST` request to either `/v3/jobs/:id/accept` or `/v3/jobs/:id/reject`. 

For accepting, if you provide appointment information, the job will move into "scheduled" status. Otherwise, it will move into "unscheduled".

For rejecting, the job will move into "rejected" status and it will become read-only for the organization's users in our application.

<aside class="notice">If you'll notice, we do not have a status called "accepted". That's because when the job is accepted, it moves either into "unscheduled" or "scheduled" status, depending on if the dispatcher is also scheduling an appointment.</aside>

## Create a Job
> Request

```json
{
  "title": "Fix the Toilet",
  "description": "The toilet is **clogged**.",
  "service_type": "PLB",
  "address": {
    "street_1": "1234 Test Avenue",
    "city": "Boston",
    "state": "MA",
    "postal_code": "02115",
    "timezone": "America/New_York"
  },
  "status": "offered",
  "customer_id": 10,
  "organization_id": 254
}
```

> Response

```json
{
  "job": {
    "id": 1,
    "title": "Fix the Toilet",
    "description": "The toilet is **clogged**.",
    "service_type": "PLB",
    "address": {
      "street_1": "1234 Test Avenue",
      "city": "Boston",
      "state": "MA",
      "postal_code": "02115",
      "timezone": "America/New_York"
    },
    "organization_id": 254,
    "status": "offered",
    "customer_id": 10,
    "customer": {
      "id": 10,
      "first_name": "Joe",
      "last_name": "Shmo",
      "external_ids": ["AAA123"],
      "email": "joe.shmo@gmail.com",
      "phone_numbers": [
        {
          "number": "+15555555555",
          "type": "mobile",
          "primary": true
        }
      ]
    }
  }
}
```

`POST /v3/jobs`

## List Jobs

> Response

```json
{
  "jobs": [
    {},
    {}
  ],
  "meta": {
    "total": 2,
    "limit": 200,
    "offset": 0
  }
}
```

### Query Parameters
Parameter | Description
--------- | -----------
brand_id_eq | Show jobs in a certain brand
customer_id_eq | Show jobs for a certain customer
customer_id_in | Show jobs for a group of customers
external_ids_contains | Show jobs with a certain external ID
organization_id_eq | Show jobs assigned to a certain organization
organization_id_in | Show jobs assigned to a group of organizations
status_eq | Show jobs in a certain status
status_in | Show jobs in a group of statuses
status_not_eq | Show jobs that are not in a certain status

## View a Single Job

> Response

```json
{
  "job": {
    "id": 1,
    "title": "Fix the Toilet",
    "description": "The toilet is **clogged**.",
    "service_type": "PLB",
    "address": {
      "street_1": "1234 Test Avenue",
      "city": "Boston",
      "state": "MA",
      "postal_code": "02115",
      "timezone": "America/New_York"
    },
    "organization_id": 254,
    "status": "offered",
    "customer_id": 10,
    "customer": {
      "id": 10,
      "first_name": "Joe",
      "last_name": "Shmo",
      "external_ids": ["AAA123"],
      "email": "joe.shmo@gmail.com",
      "phone_numbers": [
        {
          "number": "+15555555555",
          "type": "mobile",
          "primary": true
        }
      ]
    }
  } 
}
```

`GET /v3/jobs/:id`

## Update Job

> Request

```json
{
  "status": "canceled"
}
```

> Response

```json
{
  "job": {
    "id": 1,
    "title": "Fix the Toilet",
    "description": "The toilet is **clogged**.",
    "service_type": "PLB",
    "address": {
      "street_1": "1234 Test Avenue",
      "city": "Boston",
      "state": "MA",
      "postal_code": "02115",
      "timezone": "America/New_York"
    },
    "organization_id": 254,
    "status": "canceled",
    "customer_id": 10,
    "customer": {
      "id": 10,
      "first_name": "Joe",
      "last_name": "Shmo",
      "external_ids": ["AAA123"],
      "email": "joe.shmo@gmail.com",
      "phone_numbers": [
        {
          "number": "+15555555555",
          "type": "mobile",
          "primary": true
        }
      ]
    }
  }
}
```

`PATCH /v3/jobs/:id`

<aside class="notice">Remember, to move a job from "offered" status, you need to <a href="#accept-reject-behalf">accept or reject it on behalf of the orgnanization</a> if they cannot do so in our application.</aside>

# Location Entity <a name="location-schema"></a>
Locations are not business objects in our system, but are attributes on several of our core business objects. 

Currently Dispatch only supports locations in the US and Canada.

attribute | type | notes
--------- | ---- | -----
street_1 | string | required
street_2 | string |
city | string | required
state | enum&laquo;string&raquo; | two-character abbreviation for the state. 
postal_code | string | 5-digit US or 6-character Canadian postal code
timezone | enum&laquo;string&raquo; | Timezone in [IANA](https://www.iana.org/time-zones) format. <br/>If not provided we will attempt to find the timezone from the provided postal code.

# <a name="organizations"></a> Organizations
Organizations are the service providers that perform the work.

## Attributes

Attribute | Type | Required | Updatable | Description
--------- | ---- | -------- | --------- | -----------
name | string | Y | Y |
external_ids | array&laquo;string&raquo; | N | Y | Your ID(s) for this organization. See [external ids](#external-ids)
address | [Location](#location-schema) | N | Y | Organization's physical address.
phone_number | string | N | Y | Phone number in [RFC3966 format](https://www.ietf.org/rfc/rfc3966.txt)
email | string | Y | Y |
logo_token | string | N | Y | Token for organization's logo in our [file system](#files-photos)

## Create an Organization

> Request

```json
{
  "name": "Joe's Plumbing",
  "address": {
    "street_1": "555 Test Avenue",
    "street_2": "Unit 5",
    "city": "Boston",
    "state": "MA",
    "postal_code": "02115",
    "timezone": "America/New_York"
  },
  "phone_number": "+15551234567",
  "email": "joe@joesplumbing.com",
  "external_ids": ["AAA123"]
}
```

> Response

```json
{
  "organization": {
    "id": 123,
    "name": "Joe's Plumbing",
    "address": {
      "street_1": "555 Test Avenue",
      "street_2": "Unit 5",
      "city": "Boston",
      "state": "MA",
      "postal_code": "02115",
      "timezone": "America/New_York"
    },
    "phone_number": "+15551234567",
    "email": "joe@joesplumbing.com",
    "external_ids": ["AAA123"]
  }
}
```

`POST /v3/organizations`

### Create an initial user for organization
When creating an organization via `POST /v3/organizations`, if you provide an additional `boolean` parameter on the organization payload called `create_user` and set it to `true`, a [user](#users) will be created with both the "dispatcher" and "technician" roles using the `email` attribute on the organization. While that user will not be able to log in directly (we won't make up a password), he or she can start receiving emails for jobs and start interacting them in the Dispatch mobile or desktop application. From there, the new user can set a mobile number and password to be able to log in normally.

## List Organizations

> Response

```json
{
  "organizations": [
    {},
    {}
  ],
  "meta": {
    "total": 2,
    "limit": 200,
    "offset": 0
  }
}
```

### Query Parameters
Parameter | Description
--------- | -----------
external_ids_contains | Show organizations with a certain external ID

## View a Single Organization

> Response

```json
{
  "organization": {
    "id": 123,
    "name": "Joe's Plumbing",
    "address": {
      "street_1": "555 Test Avenue",
      "street_2": "Unit 5",
      "city": "Boston",
      "state": "MA",
      "postal_code": "02115",
      "timezone": "America/New_York"
    },
    "phone_number": "+15551234567",
    "email": "joe@joesplumbing.com",
    "external_ids": ["AAA123"]
  }
}
```

`GET /v3/organizations/:id`

## Update an Organization

> Request

```json
{
  "phone_number": "+15557826394"
}
```

> Response

```json
{
  "organization": {
    "id": 123,
    "name": "Joe's Plumbing",
    "address": {
      "street_1": "555 Test Avenue",
      "street_2": "Unit 5",
      "city": "Boston",
      "state": "MA",
      "postal_code": "02115",
      "timezone": "America/New_York"
    },
    "phone_number": "+15557826394",
    "email": "joe@joesplumbing.com",
    "external_ids": ["AAA123"]
  }
}
```

`PATCH /v3/organizations/:id`

## Delete an Organization

`DELETE /v3/organizations/:id`

# <a name="survey-responses"></a> Survey Responses

Survey responses are submitted by customers via our customer portal. They're sent when an appointment moves into "complete" status, or, if there are no appointments, when the job itself moves to "complete". Surveys are **read only** from your perspective - we send them out automatically to the customer, and then you'll be able to view them by making a `GET` request to `/v3/survey_responses` with the appropriate filters.

<aside class="notice">Note that to avoid spam, we <b>do not</b> send a survey to a customer if they have received one for the same job within the past 24 hours.</aside>

## Attributes

Attribute | Type  | Description
--------- | ---- | -----------
job_id | int | 
appointment_id int | ID of the parent appointment. May be `null`, if there was no appointment.
rating | int | 0-5 rating from the customer
message | string | Additional feedback from the customer

## List Survey Responses

> Response

```json
{
  "survey_responses": [
    {
      "rating": 4,
      "job_id": 10,
      "appointment_id": 26,
      "message": "Technician was really nice"
    }
  ],
  "meta": {
    "total": 1,
    "offset": 0,
    "limit": 200
  }
}
```

`GET /v3/survey_responses`

### Query Parameters
Parameter | Description
--------- | -----------
job_id_eq | Show survey responses for a specific job

# <a name="users"></a> Users

Users are the dispatchers managing the workforce for an organization, or the technician performing the work.



## User Authentication

<aside class="notice">Note that users can log in either using their <b>phone number</b>, in which case we will send a code to their phone to verify their identity, or using a combination of their <b>email</b> and <b>password</b>. As such, if you provide a phone number, it must be a valid mobile number for the user to be able to log in.</aside>

## Access Control

* For organizations in a **third-party network**, you have **only read access** to their users.
* For organizations in a **first-party network**, you have **read and write access** to their users. You can create and update users, and move them within organizations in your first-party network.
* If you're authenticated as a user within an organization, you  have **read and write access** to that single organization.

## User Attributes

Attribute | Type | Required | Updatable | Description
--------- | ---- | -------- | --------- | -----------
organization_id | string | Y | Y | Organization this user is assigned to
first_name | string | Y | Y |
last_name | string | Y | Y |
address | [Location](#location-schema) | N | Y |
phone_number | string | N | Y | Phone number in [RFC3966 format](https://www.ietf.org/rfc/rfc3966.txt).
email | string | N | Y |
photo_token | string | N | Y | Token for user's photo in our [file system](#files-photos). This will show to the customer on the customer portal.
roles | array&laquo;string&raquo; | Y | Y | Valid roles are either "dispatcher" or "technician". Users can have both roles. Note that users with only the "technician" role cannot log in to the desktop application.
password | string | N | Y | User's password. Only used for creation and updating. Not returned in `GET` requests.

## Create a User

> Request

```json
{
  "organization_id": 10,
  "first_name": "Jim",
  "last_name": "the Technician",
  "roles": ["technician"],
  "phone_number": "+15551234567",
  "email": "jim@joesplumbing.com",
  "password": "supersecretpassword"
}
```

> Response

```json
{
  "user": {
    "id": 12834,
    "organization_id": 10,
    "first_name": "Jim",
    "last_name": "the Technician",
    "roles": ["technician"],
    "phone_number": "+15551234567",
    "email": "jim@joesplumbing.com"
  }
}
```

`POST /v3/users`

## List Users

> Response

```json
{
  "users": [
    {},
    {}
  ],
  "meta": {
    "total": 2,
    "limit": 200,
    "offset": 0
  }
}
```

`GET /v3/users`

### Query Parameters

Parameter | Description
--------- | -----------
by_user_roles | Search for users in the provided role(s).
email_eq | Search for users by email
organization_id_eq | Search for users in a specific organization


## View a Single User

> Response

```json
{
  "user": {
    "id": 12834,
    "organization_id": 10,
    "first_name": "Jim",
    "last_name": "the Technician",
    "roles": ["technician"],
    "phone_number": "+15551234567",
    "email": "jim@joesplumbing.com"
  }
}
```

`GET /v3/users/:id`

## Update a User

> Request

```json
{
  "password": "newpassword"
}
```

> Response

```json
{
  "user": {
    "id": 12834,
    "organization_id": 10,
    "first_name": "Jim",
    "last_name": "the Technician",
    "roles": ["technician"],
    "phone_number": "+15551234567",
    "email": "jim@joesplumbing.com"
  }
}
```

`PATCH /v3/users/:id`

## Delete/Deactivate a User
You can **soft-delete** a user to deactivate their account (they will no longer be able to log in) but still see their appointment history.

Common scenarios for using this feature are:
* If an employee is fired
* If you have a seasonal employee

<aside class="info">If a user is deactivated, you can re-use his or her phone number for a different user. This is useful if you have company-issued phones that you want to assign to a different employee, or if that employee moves between multiple organizations that you control.</aside>

`DELETE /v3/users/:id`

## Reactivate a User

`POST /v3/users/:id/restore`

