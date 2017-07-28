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

# The Work Order Document

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

The Work Order document contains all of the information to create all objects in the Dispatch system needed to handle your workflow. See [Posting a Work Order to Dispatch](#posting-work-order)

attribute | type | notes
--------- | ---- | -----
title | `string` | required
description | `string` | markdown is supported
service_type | `string` | Type of service to be performed
brand_id | `integer` | Optionally assign to a [brand](#branding) within your account. <br/>This is the brand's ID in the Dispatch system.
orchestration | `enum<string>` | See [orchestration algorithms](#orchestration-algorithms)
external_id | `string` | ID for the work order in your system. See [external ids](#external-ids)
location | `Location` | Location of the work. [Location entity schema](#location-schema)
appointment_windows | `array<AppointmentWindow>` | Optionally provide appointment windows for the <br/>organization to choose from. <br/>[AppointmentWindow entity schema](#appointment-window-schema)
contacts | `array<Contact>` | List of contacts for the work order. [Contact entity schema](#contact-schema)
organizations | `array<Organization>` | List of organizations to send/offer the work to. <br/>Note that in "direct" orchestration only a single <br />organization is permitted. [Organization entity schema](#organization-schema)

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
external_ids | `array<string>` | ID(s) for the contact in your system. See [external ids](#external-ids)
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

# Interacting with Work Orders in Dispatch <a name="work-order-api"></a>

## Create a Work Order
> Response

```json
{
	"work_order": {
		"id": 1
		"created_at": "ISO8601 Timestamp",
		"updated_at": "ISO8601 Timestamp",
		"title": "",
		"location": {}
		"description": "",
		"service_type": "",
		"contacts": [...],
		"organizations": [
			{
				"id": 2,
				"customer_id": 3
				"status": "offered",
			}
		]
	}
}
```

`POST /v3/work_orders`

You will need to make a `POST` request to `https://api.dispatch.me/v3/work_orders`, using your current method of authentication (OAuth2 Bearer Token). The response will look much like your work order, but have additional data representing the work as seen by each organization in the list.

## Update a Work Order

> Request

```json
{
	"title": "New title",
	"description": "New description",
	"contacts": [
		{
			"first_name": "Different",
			"last_name": "Guy",
			"phone_numbers": [
				{
					"value": "+15556666666",
					"label": "mobile",
					"preferred": true
				}
			],
			"primary": true
		}
	]
}
```

> Response: 

```json
{
	"work_order": {...}
}
```

`PATCH /v3/work_orders/:id`


You can update the work order's details after it has been created. Since this is a PATCH, you should only provide the attributes that you want to change.

### Caveats
* You **cannot** update the organizations once a work order has been created. To accomplish this, you should **cancel** the original work order and create a new one with the correct organization assignments.
* If you're updating a contact, you must provide all of the information for that contact, so we know whether to create a new record in our system for that contact or simply update the existing one.

## Change the Status of a Work Order

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

## Common "Update" Workflows

### Accept on behalf of organization

### Reject on behalf of organization

### Cancel work order

### Complete on behalf of organization