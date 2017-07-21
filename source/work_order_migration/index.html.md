---
title: Dispatch Job Offer -> Work Order Migration Guide

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
We have begun the deprecation process for our **Job Offer** entity. Below you will find instructions on how to migrate to our new, simpler, more robust **Work Order**.

The data on both the Job Offer and Work Order are relatively similar. We have restructured the document to make it easier to follow as well as to support new features like **different workflows**, **multiple contacts per job**, and **new orchestration algorithms** like round robin and jump ball.

The biggest change is that Work Orders are now used only to **create and orchestrate entities inside of Dispatch**, and are not accessible after the initial request. In other words, they are the simplest way to create all of the necessary objects in the Dispatch platform to service a customer. As such, there is no request to `GET /v3/workorder`. Instead, you should query our API for information about the [Job](#jobs), [Appointment](#appointments), or [Service Provider](#service-provider) that are created when you `POST` the Work Order.

# The Work Order Document

```json
{
	"title": "PLB 123: Fix the Toilet",
	"description": "## Job Info\n* Dog attacked the last guy",
	"service_type": "PLB",
	"workflow": "service",
	"orchestration": "direct",
	"status": "offered",
	"external_ids": ["AAA123"],
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
			"external_ids": ["BBB456"],
			"primary": true,
			"notes": "This person is really nice",
			"contact_methods": [
				{
					"method": "phone",
					"type": "mobile",
					"value": "+15551234567",
					"preferred": true
				},
				{
					"method": "email",
					"type": "work",
					"value": "testy.mcgee@widgets.com",
					"preferred": true
				}
			]
		}
	],
	"service_providers": [
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
			"external_ids": ["CCC789"]
		}
	]
}
```

The Work Order document contains all of the information to create all objects in the Dispatch system needed to handle your workflow. See [Posting a Work Order to Dispatch](#posting-work-order)

attribute | type | notes
--------- | ---- | -----
title | `string` | required
description | `string` | markdown is supported
service_type | `string` | Type of service to be performed
workflow | `enum<string>` | "sales" or "service". See [workflow types](#workflow-types)
brand_id | `integer` | Optionally assign to a [brand](#branding) within your account. <br/>This is the brand's ID in the Dispatch system.
orchestration | `enum<string>` | See [orchestration algorithms](#orchestration-algorithms)
external_ids | `array<string>` | ID(s) for the work order in your system. See [external ids](#external-ids)
status | `enum<string>` | Starting status for the work order. See [state machine by workflow](#state-machine)
location | `Location` | Location of the work. [Location entity schema](#location-schema)
appointment_windows | `array<AppointmentWindow>` | Optionally provide appointment windows for the <br/>service provider to choose from. <br/>[AppointmentWindow entity schema](#appointment-window-schema)
contacts | `array<Contact>` | List of contacts for the work order. [Contact entity schema](#contact-schema)
service_providers | `array<ServiceProvider>` | List of service providers to send/offer the work to. <br/>Note that in "direct" orchestration only a single <br />service provider is permitted. [ServiceProvider entity schema](#service-provider-schema)

## Location Entity <a name="location-schema"></a>
Currently Dispatch only supports locations in the US and Canada.

attribute | type | notes
--------- | ---- | -----
street_1 | `string` | required
street_2 | `string` |
city | `string` | required
state | `enum<string>` | two-character abbreviation for the state. 
postal_code | `string` | 5-digit US or 6-character Canadian postal code
timezone | `enum<string>` | Timezone in [IANA](https://www.iana.org/time-zones) format. <br/>If not provided we will attempt to find the timezone from the provided postal code.
external_ids | `array<string>` | Your system's IDs for the location <br/>See [external ids](#external-ids)

## Appointment Window Entity <a name="appointment-window-schema"></a>

attribute | type | notes
--------- | ---- | -----
start_time | `string` | ISO8601 timestamp
end_time | `string` | ISO8601 timestamp

## Service Provider Entity <a name="service-provider-schema"></a>

We will attempt to look up the service provider in our system using, in order:

* `id` (Dispatch's ID)
* `external_ids` (your system's IDs)
* name + address + email + phone number

If we find the service provider, we will use the record we already have, since they may have already logged into Dispatch and be managing their jobs and we don't want to create another account for them. If we don't find the record, we will create a new one with the data you provide.

Note that if you are sure that the service provider already exists in Dispatch, you **only** need to provide either the `id` or `external_ids` fields. However, we recommend that you provide all data to avoid any errors.

attribute | type | notes
--------- | ---- | -----
name | `string` | Name of the provider.
address | `Location` |
email | `string` | Email address for the service provider's office
phone_number | `string` | Phone number in [RFC3966 format](https://www.ietf.org/rfc/rfc3966.txt)
external_ids | `array<string>` | ID(s) for the service provider in your system. [external ids](#external-ids)
id | `int` | ID for the service provider in Dispatch. Provide if you know it, otherwise omit.

## Contact Entity <a name="contact-schema"></a>

A contact is any person who the service provider may need to get in touch with over the lifecycle of the work order.

attribute | type | notes
--------- | ---- | -----
first_name | `string` |
last_name | `string` |
company_name | `string` |
external_ids | `array<string>` | ID(s) for the contact in your system. See [external ids](#external-ids)
primary | `bool` | You must designate a single primary contact (set to `true`) <br/>per work order. This person will be able to log in to the customer  <br/> portal, suggest appointment windows, and provide feedback at <br/>the end of the appointment.
notes | `string` |
billing_address | `Location` | Optional billing address for this contact.
contact_methods | `array<ContactMethod>` | [Contact methods](#contact-method-schema) for this contact

### Contact Method Entity <a name="contact-method-schema"></a>

Contacts can have one or more contact methods. Note that you can not have more than 1 preferred "email" method or more than 1 preferred "phone" method. (You can, however, have 1 of each).

attribute | type | notes
--------- | ---- | -----
method | `enum<string>` | Either "phone" or "email"
type | `string` | E.g. "work", "fax", "mobile", "home"
value | `string` | Either the email address or phone number. <br />Phone number must be in [RFC3966 format](https://www.ietf.org/rfc/rfc3966.txt)
preferred | `bool` | If `true`, this phone number or email address will <br />be used to allow this person to log in to the Dispatch system.

# Posting a Work Order to Dispatch <a name="posting-work-order"></a>

> Response

```json
{
	"service_provider_id": 123,
	"primary_contact_id": 456,
	"job_id": 789,
	"appointment_id": null
}
```

You will need to make a `POST` request to `https://api.dispatch.me/v3/workorder`, using your current method of authentication (OAuth2 Bearer Token). The response will tell you which objects were created in the Dispatch system to support your work order.

* A **Job** is created and will be accessible via `GET` to `/v3/jobs/:id`
* A **Customer** (designated as "Primary Contact") is created or matched to an existing record and will be accessible via `GET` to `/v3/customers/:id`
* A **Service Provider** is created or matched to an existing record and will be accessible via `GET` to `/v3/service_providers/:id`
* If the `appointment_windows` attribute is provided, an **Appointment** is created and will be accessible via `GET` to `/v3/appointments/:id`

# Migrating from Job Offers

## Attribute Mapping

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
	"workflow": "service",
	"orchestration": "direct",
	"title": "Fix the Sink",
	"description": "The sink needs fixing",
	"service_type": "PLB",
	"external_ids": ["AAA123"],
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
	"service_providers": [
		{
			"id": 123,
			"external_ids": ["CCC789"],
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
			"external_ids": ["BBB456"],
			"contact_methods": [
				{
					"method": "email",
					"preferred": true,
					"value": "joe.shmo@email.com"
				},
				{
					"method": "phone",
					"preferred": true,
					"type": "mobile",
					"value": "+15551234567",
					"preferred": true
				},
				{
					"method": "phone",
					"type": "home",
					"preferred": false,
					"value": "+15559876543"
				}
			]
		}
	]
}
```

Note that since we now support multiple contacts per work order, the single `customer` record on Job Offers can be mapped to the first member of the `contacts` array, marked as `"primary": true`.

Also note that previous job offers were always inferred to be in the "service" workflow and "direct" orchestration, so those values should now be explicitly provided to get the same behavior.

job offer attribute | work order attribute | notes
------------------- | -------------------- | -----
`offer_strategy` | `orchestration` | See [orchestration algorithms](#orchestration-algorithms)
`duration` | - | Expiration is no longer supported
`duration_algorithm` | - | Expiration is no longer supported
`ui_options` | - | These are now configurable on a per-account basis rather than for each work order. Please speak with your account manager for details.
`entities.$.type` | - | Work orders can only be sent to service providers
`entities.$.id` | `service_providers.$.id`
`entities.$.external_id` | `service_providers.$.external_ids` | Add a single member to the `external_ids` array
`entities.$.data.name` | `service_providers.$.name` |
`entities.$.data.address` | `service_providers.$.address` |
`entities.$.data.email` | `service_providers.$.email` |
`entities.$.data.phone_number` | `service_providers.$.phone_number`
`job.title` | `title` |
`job.description` | `description` |
`job.service_type` | `service_type` |
`job.external_id` | `external_ids` | Add a single member to the `external_ids` array.
`job.address` | `location` |
`customer.first_name` | `contacts[0].first_name` |
`customer.last_name` | `contacts[0].last_name` |
`customer.email` | `contacts[0].contact_methods.$.value` | Set this contact method to `method:"email"` and `preferred:true`
`customer.phone_numbers.$.number` | `contacts[0].contact_methods.$.value` | 
`customer.phone_numbers.$.type` | `contacts[0].contact_methods.$.type` |
`customer.phone_numbers.$.primary` | `contacts[0].contact_methods.$.notify` | Also set to `preferred:true`
`customer.external_id` | `contacts[0].external_ids` | Add a single member to the `external_ids` array
`customer.billing_address` | `contacts[0].billing_address` |
`customer.home_address` | - | No need for home address anymore - this is taken from the work order's location.
`appointment.window_start_time.$` | `appointment_windows.$.start_time` |
`appointment.window_end_time.$` | `appointment_windows.$.end_time` |

## Webhooks

Pending new webhooks project.

## Common "Update" Workflows

See examples on the right.

> Accept on behalf of Service Provider: If there is no appointment, `PATCH /v3/jobs/:id`

```json
{
	"status": "unscheduled"
}
```

> Accept on behalf of Service Provider: If there is an appointment, create it, and the job's status will change automatically to "scheduled". `POST /v3/appointments`:

```json
{
	"job_id": 123,
	"time": "ISO8601 Timestamp",
	"duration": 3600,
	"status": "scheduled"
}
```

> Cancel Work Order: `PATCH /v3/jobs/:id`

```json
{
	"status": "canceled"
}
```

> Complete Work Order: `PATCH /v3/jobs/:id`

```json
{
	"status": "complete"
}
```

### Accept on Behalf of Service Provider
Previously, to accept a job offer on behalf of a service provider, you would `PATCH /v1/organizations/:id/job_offers/:id`.

Now, you just need to change the status of that service provider's job or create an appointment for that job, using the job ID you received back from us when you created the work order.

!!!NEED TO EXPOSE WOGS AND HAVE THEM HIT THAT FOR ACCEPT/REJECT.!!!

### Canceling a Work Order

To cancel a work order, just change the status of its job(s) to "canceled".

### Completing a Work Order

If the work was completed outside of Dispatch but you still want to use our survey feature, you can update it in the Dispatch system by just changing the job's status to "complete".