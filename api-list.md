# API

## Verbs

- `GET` - Read
- `POST` - Create
- `DELETE` - remove/deactivate
- `PUT` - update by **setting** data
- `PATCH` - update by **adding** data

## HTTP Body

`accepts: application/json`

- any fields that are included (even if empty) will be used to set/overwrite data
- any fields that are not included will be ignored and unchanged on the resource

## Requests

**Filtering**

Requests can filter the results by parameters. eg: `?filter=active&filter=name%3Dswing`

- filter parameters can be poplated by just the field name (which tests for existance)
- they can also have a property name, followed by an operator

Operators (will be url encoded)
- "=" looks for exact match
- "<" ("<=") looks for a value that is less (or LE) than the one provided
- ">" (">=") looks for a value that is greater (or GE) than the one provided
- "~" string fuzzy match
- "^~" string starts with...
- "~$" string ends with

**Sorting**

Requests can sort the results by using the `sort` query parameter. If several sort parameters are provided then the order matters. The results should be sorted in order of precedence. eg: `?sort=-created&sort=name` The results are sorted first by date created (desc) and then by name (asc)

If a "-" is specified before the value of the sort query, then the sort order should be descending instead of ascending for that field.

**Pagination**

The api should have a maximum allowed number of resources returned per request. 

Results can be paginated by using two parameters: `page` and `pagesize`. eg: `?page=2&pagesize=50` will return the second page of 50 entries.

Information should be placed in the meta response property that relate to the total number of results, total pages, and current page

## Responses

- the api methods for any specific resource (excepting creation) should be idempotent. Any identical request body should result in the same response.
- any repeated action to a resource (eg: multiple `DELETE`s to the same path) will result in a `304 Not Modified`

A response should take the form:
```
{
    "meta": {
        // meta content... ex:
        "total": 40,
        "totalPages": 4,
        "page": 2
    },

    "data": [], // the actual data. Can be array or object

    "error": {} // data and error are mutually exclusive
}
```


## Conventions

* Root resource names are plural
  * Related resources being acted on via a root resource are singular
    * e.x. `surveys/node`
* Resource creations are performed via `POST` with JSON input bodies
  * Avoid using URL query parameters
* Verb actions that update flag-type states can be included under resources
    * e.x. `node/publish`

## Resource Structure

```
API Root Path  
|-- users
|   |-- payments
|-- nodes (limited api)
|-- organizations
|   | -- permissions
|-- groups
|-- folders
|-- occasions
|-- sureys
|-- registrations
```

## Users
---
> NOTE: This may or not be wrapped in API. The functionality already exists in Cognito and we may just interact directly with Cognito for simplicity at the moment.


### Payments
> Assume for now same verbs/functions between Stripe and PayPal - represented as `[service]`

`POST /users/:id/payments/[service]` - create a charge for a transaction


## Organizations

```
{
    "id": "79d79ccc-d424-43cd-b2e1-5087a5bc5180",
    "created": "2017-11-03T23:17:39.235Z",
    "modified": "2017-11-26T00:34:40.397Z",
    "rootFolder": "e4c96703-e6c0-4973-b41a-1eba95731031",
    "name": "Bahringer Group",
    "type": "organization",
    "slug": "nostrum-aut-dolore",
    "thumb": "https://placeimg.com/300/300?"
}
```

### Occasions

```
{
    "public": true, // html body sets node to be public.
    "active": true, // html body sets node to be active
    "surveys": [ "surveyId1", "surveyId2", ... ] // set surveys
}
```

### Offerings
```
{
    "price": 1000 // $10
    , "startDate" : TIMESTAMP // when offering begins
    , "endDate" : TIMESTAMP // when it ends
    , "cancelationDeadline": TIMESTAMP // can't be canceled after this date
    , "modificationDeadline": TIMESTAMP // information (survey responses) can't be modified after this
    , "conditionalPrices": {} // future nice to have
    , ...
}
```

### Folders
> Although folders represent a tree structure, they should not be organized in the api as a tree.
> All organizations have a single unique root folder (that is nameless) that is the top level of navigation for that organization.



### Groups
>Groups are organizational containers, but cannot be used for functional node purposes such as registration.  
>Groups can be used for administrative grouping, such as for organization on the GUI.  
>Groups are a core component for assigning access control to users inside an organization.  

### Permissions
>Access control for organizations.  
>An `org_id`:`user_id` pair will contain a set of `group_id`:`access_level` pairs. The `group_id` can be ***** to represent an access level the user has on all groups in the organization.  
>For now, access levels will not be user defined - however applying multiple access levels to a user in a group will combine to the **least** restrictive combination of those access levels. We will define access levels such as "Owner", "Organization Admin", "Bookkeeper", "Event Admin", etc.


