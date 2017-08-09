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

## Signing Up

If you're interested in using our API, please work with your account manager and we'll set you up with your credentials. Don't have an account manager? Provide your information here and we'll get back to you.

## Dispatch Business Objects

Here's a brief description of our business objects and their role in the Dispatch system.

Entity | Description
------ | -----------
[Work Order](#work-orders) | An object containing all of the information to create jobs, customers, organizations, and appointments. We recommend using this unless you have a use case that requires individual access to the underlying objects.
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

## Filtering and Paginating
For `GET` requests, several endpoints allow you to filter the results based on certain record attributes. These are all provided as nested values on a `filter` object within the query string, like so: `?filter[job_id]=123&filter[status]=scheduled`

For all `GET` requests that return multiple records, you can paginate by providing `limit` and `offset` parameters, like so (the maximum value for `limit` is currently 100): `?filter[job_id]=123&filter[status]=scheduled&limit=100&offset=1000`

# Authentication

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

As the source of the work, you have full access to the transactional data you send to Dispatch (meaning, the work order and all child objects). However, depending on the relationship you have with the organizations, they may already be in our system and may already be receiving work from other sources. In that case, you will have limited access to the organizations, their customers, and their dispatchers and technicians. If you have a first-party network, you will have more access to those objects. We recommend you discuss this with your account manager so we can implement the best ACL for your use case.

# <a name="attachments"></a> Attachments

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

# <a name="appointments"></a> Appointments

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

### Residual Status Behavior
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

# <a name="brands"></a> Brands
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

}

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
<aside class="notice">
Careful! If you delete a brand then all jobs currently within that brand will use your default "account-level" branding.
</aside>

`DELETE /v3/brands/:id`

# <a name="customers"></a> Customers
Customers represent the homeowner, landlord or other entity the work is being done for. Customers in Dispatch belong to individual organizations, so if you have multiple organizations doing work for the same customer in your system, there will be multiple instances of that customer in Dispatch. We do this to allow organizations to keep their own notes and make their own updates to customer data without affecting other organizations.

<aside class="notice">
Note: customers will be automatically created if you use the <a href="#job-factory">job factory endpoint</a>, so you shouldn't have to interact directly with the Customer models unless you are maintaining them separately from jobs.
</aside>

## Customer Attributes
Note that currently we support **multiple phone numbers** but only a **single email address** for customers. We have plans to improve this in the near future.

Attribute | Type | Description
--------- | ---- | -----------
organization_id | `int` | ID of the parent organization
first_name | `string` |
last_name | `string` |
company_name | `string` |
notes | `string` | Any additional notes about this customer
email | `string` | Email address for the customer. This will be used to send email notifications if they are configured for your account.
phone_numbers | `array<PhoneNumber>` | See below for phone number object attributes
home_address | `Location` | This is used only when creating a new job for this customer in our application. For jobs that come in via our API, the `job.address` property is used. See [Location entity schema](#location-schema)
billing_address | `Location` | See [Location entity schema](#location-schema)
external_ids | `array<string>` | See [external IDs](#external-ids)

### Phone Number Object
Attribute | Type | Description
--------- | ---- | -----------
number | `string` | Phone number in [RFC3966 format](https://www.ietf.org/rfc/rfc3966.txt)
primary | `boolean` | Set to `true` if you want this number to receive SMS notifications
type | `string` | E.g. "Mobile", "Work"

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

## Job Attributes
Note that the `organization` and `customer` nested objects will be ignored if `organization_id` or `customer_id` are provided, respectively.

Attribute | Type | Description
--------- | ---- | -----------
title | `string` |
description | `string` | Additional details for the job. Supports [markdown](https://daringfireball.net/projects/markdown/syntax)
service_type | `string` | Type of service, e.g. "plumbing"
external_ids | `array<string>` | Your ID(s) for this job. See [external-ids](#external-ids)
address | `Location` | [Location](#location-schema) of the job.
brand_id | `int` | Optional ID for your [brand](#brands)
customer_id | `int` | ID of the customer object.
organization_id | `int` | ID of the assigned organization
service_fee | `float` | Fee the customer owes for service
status | `string` | Status of the job. See [job statuses](#job-statuses)
status_message | `string`| Optional qualifier for the current status

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

<aside class="notice">If you'll notice, we do not have a status called "accepted". That's because when the job is accepted, it moves either into "unschedled" or "scheduled" status, depending on if the dispatcher is also scheduling an appointment.</aside>

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

# <a name="organizations"></a> Organizations
Organizations are the service providers that perform the work.

## Organization Attributes

Attribute | Type | Description
--------- | ---- | -----------
name | `string` |
external_ids | `array<string>` | Your ID(s) for this job. See [external ids](#external-ids)
address | `Location` | Organization's physical address. See [Location entity schema](#location-schema)
phone_number | `string` | Phone number in [RFC3966 format](https://www.ietf.org/rfc/rfc3966.txt)
email | `string` |
logo_token | `string` | Token for organization's logo in our [file system](#files-photos)

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

Survey responses are submitted by customers via our customer portal. They're sent when an appointment moves into "complete" status, or, if there are no appointments, when the job itself moves to "complete".

<aside class="notice">Note that to avoid spam, we **do not** send a survey to a customer if they have received one for the same job within the past 24 hours.</aside>

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

<aside class="notice">Note that users can log in either using their **phone number**, in which case we will send a code to their phone to verify their identity, or using a combination of their **email** and **password**. As such, if you provide a phone number, it must be a valid mobile number for the user to be able to log in.</aside>

## User Attributes

Attribute | Type | Description
--------- | ---- | -----------
organization_id | `string` | Organization this user is assigned to
first_name | `string` |
last_name | `string` |
address | `Location` |
phone_number | `string` | Phone number in [RFC3966 format](https://www.ietf.org/rfc/rfc3966.txt).
email | `string` |
logo_token | `string` | Token for organization's logo in our [file system](#files-photos)
roles | `array<string>` | Valid roles are either "dispatcher" or "technician". Users can have both roles. Note that users with only the "technician" role cannot log in to the desktop application.
password | `string` | User's password. Only used for creation and updating. Not returned in `GET` requests.

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
  "id": 12834,
  "organization_id": 10,
  "first_name": "Jim",
  "last_name": "the Technician",
  "roles": ["technician"],
  "phone_number": "+15551234567",
  "email": "jim@joesplumbing.com"
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

## Delete a User

`DELETE /v3/users/:id`

# <a name="work-orders"></a> Work Orders
Work Orders represent all of the data needed to create an organization, job, customer, and (optionally) appointment in the Dispatch system. They are not business objects themselves, but rather used as a "factory" of sorts to create all of the underlying objects and apply any sort of orchestration rules like "round robin" and "jump ball". 

You can **create**, **update**, and **cancel** work orders. When you do that, business objects in the Dispatch system are created or updated appropriately. This makes it easy for your system to send data to Dispatch without having to worry about the underlying objects. However, if you would like, you can still access those objects individually to check their status, etc.

## Work Order Object

```json
{
  "title": "PLB 123: Fix the Toilet",
  "description": "## Job Info\n* Dog attacked the last guy",
  "service_type": "PLB",
  "orchestration": "direct_offer",
  "external_id": "AAA123",
  "location": {
    "street_1": "1234 Test Avenue",
    "street_2": "Apt 2",
    "city": "Boston",
    "state": "MA",
    "postal_code": "02115",
    "timezone": "America/New_York"
  },
  "appointment_windows": [
    {
      "start_time": "2017-01-01T05:00:00Z",
      "end_time": "2017-01-01T07:00:00Z"
    },
    {
      "start_time": "2017-01-01T11:00:00Z",
      "end_time": "2017-01-01T13:00:00Z"
    }
  ],
  "contacts": [
    {
      "first_name": "Testy",
      "last_name": "McGee",
      "company_name": "Widgets, Inc.",
      "external_id": "BBB456",
      "primary": true,
      "notes": "This person is really nice",
      "phone_numbers": [
        {
          "label": "mobile",
          "value": "+15551234567",
          "preferred": true
        }
      ],
      "email_addresses": [
        {
          "label": "work",
          "value": "testy.mcgee@widgets.com",
          "preferred": true
        }
      ]
    }
  ],
  "organizations": [
    {
      "name": "Joe's Plumbing",
      "address": {
        "street_1": "5555 Test Street",
        "city": "Boston",
        "state": "MA",
        "postal_code": "02114",
        "timezone": "America/New_York"
      },
      "phone_number": "+15558889999",
      "email": "joe@joesplumbing.com",
      "external_id": "CCC789"
    }
  ]
}
```

attribute | type | notes
--------- | ---- | -----
title | `string` | required
description | `string` | markdown is supported
service_type | `string` | Type of service to be performed
brand_id | `integer` | Optionally assign to a [brand](#brands) within your account. <br/>This is the brand's ID in the Dispatch system.
orchestration | `enum<string>` | See [orchestration algorithms](#orchestration-algorithms)
external_id | `string` | ID for the work order in your system. See [external ids](#external-ids)
location | `Location` | Location of the work. [Location entity schema](#location-schema)
appointment_windows | `array<AppointmentWindow>` | Optionally provide appointment windows for the <br/>organization to choose from. <br/>[AppointmentWindow entity schema](#appointment-window-schema)
contacts | `array<Contact>` | List of contacts for the work order. [Contact entity schema](#contact-schema)
organizations | `array<Organization>` | List of organizations to send/offer the work to. <br/>Note that in "direct_*" orchestration only a single <br />organization is permitted. [Organization entity schema](#organization-schema)

## <a name="orchestration-algorithms"></a>Orchestration Algorithms

Algorithm | Description
--------- | -----------
direct_offer | Offer the job to a single organization. Job starts in "offered" status.
direct_assign | Assign the job to a single organization. Job starts in "unscheduled" status.
round_robin | **Coming Soon!**
jump_ball | **Coming Soon!**

## Location Entity <a name="location-schema"></a>
Locations are not business objects in our system, but are attributes on several of our core business objects. 

Currently Dispatch only supports locations in the US and Canada.

attribute | type | notes
--------- | ---- | -----
street_1 | `string` | required
street_2 | `string` |
city | `string` | required
state | `enum<string>` | two-character abbreviation for the state. 
postal_code | `string` | 5-digit US or 6-character Canadian postal code
timezone | `enum<string>` | Timezone in [IANA](https://www.iana.org/time-zones) format. <br/>If not provided we will attempt to find the timezone from the provided postal code.
external_id | `string` | Your system's IDs for the location <br/>See [external ids](#external-ids)

## Appointment Window Entity <a name="appointment-window-schema"></a>

attribute | type | notes
--------- | ---- | -----
start_time | `string` | ISO8601 timestamp
end_time | `string` | ISO8601 timestamp

## Organization Entity <a name="organization-schema"></a>

We will attempt to look up the organization in our system using, in order:

* `id` (Dispatch's ID)
* `external_id` (your system's ID)
* name + address + email + phone number

If we find the organization, we will use the record we already have, since they may have already logged into Dispatch and be managing their jobs and we don't want to create another account for them. If we don't find the record, we will create a new one with the data you provide.

Note that if you are sure that the organization already exists in Dispatch, you **only** need to provide either the `id` or `external_id` fields. However, we recommend that you provide all data to avoid any errors.

attribute | type | notes
--------- | ---- | -----
name | `string` | Name of the organization.
address | `Location` |
email | `string` | Email address for the organization's office
phone_number | `string` | Phone number in [RFC3966 format](https://www.ietf.org/rfc/rfc3966.txt)
external_id | `string` | ID for the organization in your system. [external ids](#external-ids)
id | `int` | ID for the organization in Dispatch. Provide if you know it, otherwise omit.

## Contact Entity <a name="contact-schema"></a>

A contact is any person who the organization may need to get in touch with over the lifecycle of the work order.

attribute | type | notes
--------- | ---- | -----
first_name | `string` |
last_name | `string` |
company_name | `string` |
external_id | `string` | ID(s) for the contact in your system. See [external ids](#external-ids)
primary | `bool` | You must designate a single primary contact (set to `true`) <br/>per work order. This person will be able to log in to the customer  <br/> portal, suggest appointment windows, and provide feedback at <br/>the end of the appointment.
notes | `string` |
billing_address | `Location` | Optional billing address for this contact.
email_addresses | `array<ContactMethod>` | List of email addresses for this contact. [Contact method schema](#contact-method-schema) 
phone_numbers | `array<ContactMethod>` | List of phone numbers for this contact. [Contact method schema](#contact-method-schema) 

### Contact Method Entity <a name="contact-method-schema"></a>

attribute | type | notes
--------- | ---- | -----
label | `string` | E.g. "work", "fax", "mobile", "home"
value | `string` | Either the email address or phone number. <br />Phone number must be in [RFC3966 format](https://www.ietf.org/rfc/rfc3966.txt)
preferred | `bool` | If `true`, this phone number or email address will <br />be used to allow this person to log in to the Dispatch system.

## Create a Work Order
> Response

```json
{
  "id": 123,
  "created_at": "ISO8601 Timestamp",
  "updated_at": "ISO8601 Timestamp",
  "data": {
    "title": "",
    "location": {},
    "description": "",
    "service_type": "",
    "contacts": [...],
    "organizations": [
      {
        "id": 2,
        "customer_id": 4
      }
    ]
  }
}
```

`POST /v3/work_orders`

This process does the following:

* Create or identify the organization (see [organization entity documentation](#organization-entity))
* Create or identify the customer, using the contact marked as `primary: true`. When we support multiple contacts on a job, we will use all of the provided contacts instead of just the primary.
* Create a new job for that organization and customer.
* If `appointment_windows` are provided, adds customer suggested times to the job, which will appear to the dispatcher in our application.

Using the ID returned to you upon a successful `POST`, you can update or cancel the work order (see below)

## Update a Work Order
`PATCH /v3/work_orders/:id`

You can update individual fields, such as just the `location` or just the `description`, or provide the entire work order document as you would in a `POST`.

<aside class="warning">You cannot change the assigned organization of a Work Order after it is created. Instead, we recommend that you cancel the Work Order and create a new one with the correct assignment.</aside>

## Cancel a Work Order
`POST /v3/work_orders/:id/cancel`

This will cancel the underlying job and any appointments.

## Migrating from Job Offers

> Old Job Offer

```json
{
  "offer_strategy": "direct",
  "duration": -1,
  "job": {
    "title": "Fix the Sink",
    "description": "The sink needs fixing.",
    "service_type": "PLB",
    "external_id": "AAA123",
    "address": {
      "street_1": "1234 Test Avenue",
      "city": "Boston",
      "state": "MA",
      "postal_code": "02114"
    }
  },
  "appointment": {
    "window_start_time": ["2017-01-01T05:00:00Z", "2017-01-01T11:00:00Z"],
    "window_end_time": ["2017-01-01T07:00:00Z", "2017-01-01T13:00:00Z"]
  },
  "customer": {
    "first_name": "Joe",
    "last_name": "Shmo",
    "email": "joe.shmo@email.com",
    "phone_numbers": [
      {
        "number": "+15551234567",
        "type": "mobile",
        "primary": true
      },
      {
        "number": "+15559876543",
        "type": "home",
        "primary": false
      }
    ],
    "external_id": "BBB456",
    "billing_address": {
      "street_1": "5555 Test Avenue",
      "city": "Boston",
      "state": "MA",
      "postal_code": "02115"
    }
  },
  "entities": [
    {
      "type": "organization",
      "id": 123,
      "external_id": "CCC789",
      "data": {
        "name": "Jim's Plumbing",
        "email": "jim@jimsplumbing.com",
        "phone_number": "+15558887777",
        "address": {
          "street_1": "9876 Test Avenue",
          "city": "Boston",
          "state": "MA",
          "postal_code": "02113"
        }
      }
    }
  ]
}
```
> New Work Order

```json
{
  "orchestration": "direct_offer",
  "title": "Fix the Sink",
  "description": "The sink needs fixing",
  "service_type": "PLB",
  "external_id": "AAA123",
  "location": {
    "street_1": "1234 Test Avenue",
    "city": "Boston",
    "state": "MA",
    "postal_code": "02114"
  },
  "appointment_windows": [
    {
            "start_time": "2017-01-01T05:00:00Z",
            "end_time": "2017-01-01T07:00:00Z"
        },
        {
            "start_time": "2017-01-01T11:00:00Z",
            "end_time": "2017-01-01T13:00:00Z"
        }
  ],
  "organizations": [
    {
      "id": 123,
      "external_id": "CCC789",
      "name": "Jim's Plumbing",
      "address": {
        "street_1": "9876 Test Avenue",
        "city": "Boston",
        "state": "MA",
        "postal_code": "02113"
      },
      "phone_number": "+15558887777",
      "email": "jim@jimsplumbing.com"
    }
  ],
  "contacts": [
    {
      "first_name": "Joe",
      "last_name": "Shmo",
      "primary": true,
      "billing_address": {
        "street_1": "5555 Test Avenue",
        "city": "Boston",
        "state": "MA",
        "postal_code": "02115"
      },
      "external_id": "BBB456",
      "email_addresses": [
        {
          "preferred": true,
          "value": "joe.shmo@email.com"
        }
      ],
      "phone_numbers": [
        {
          "label": "mobile",
          "value": "+15551234567",
          "preferred": true
        },
        {
          "label": "home",
          "preferred": false,
          "value": "+15559876543"
        }
      ]
    }
  ]
}
```

Note that since we now support multiple contacts per work order, the single `customer` record on Job Offers can be mapped to the first member of the `contacts` array, marked as `"primary": true`.

job offer attribute | work order attribute | notes
------------------- | -------------------- | -----
`offer_strategy` | `orchestration` | See [orchestration algorithms](#orchestration-algorithms)
`duration` | - | Expiration is no longer supported
`duration_algorithm` | - | Expiration is no longer supported
`ui_options` | - | These are now configurable on a per-account basis rather than for each work order. Please speak with your account manager for details.
`entities.$.type` | - | Work orders can only be sent to organizations
`entities.$.id` | `organizations.$.id`
`entities.$.external_id` | `organizations.$.external_id` |
`entities.$.data.name` | `organizations.$.name` |
`entities.$.data.address` | `organizations.$.address` |
`entities.$.data.email` | `organizations.$.email` |
`entities.$.data.phone_number` | `organizations.$.phone_number`
`job.title` | `title` |
`job.description` | `description` |
`job.service_type` | `service_type` |
`job.external_id` | `external_id` |
`job.address` | `location` |
`customer.first_name` | `contacts[0].first_name` |
`customer.last_name` | `contacts[0].last_name` |
`customer.email` | `contacts[0].email_addresses.$.value` | Set this contact method to `preferred:true`
`customer.phone_numbers.$.number` | `contacts[0].phone_numbers.$.value` | 
`customer.phone_numbers.$.type` | `contacts[0].phone_numbers.$.label` |
`customer.phone_numbers.$.primary` | `contacts[0].phone_numbers.$.preferred` |
`customer.external_id` | `contacts[0].external_id` |
`customer.billing_address` | `contacts[0].billing_address` |
`customer.home_address` | - | No need for home address anymore - this is taken from the work order's location.
`appointment.window_start_time.$` | `appointment_windows.$.start_time` |
`appointment.window_end_time.$` | `appointment_windows.$.end_time` |


