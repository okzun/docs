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

## Responses

- any repeated action to a resource (eg: multiple `DELETE`s to the same path) will result in a `304 Not Modified`

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
|      |-- stripe
|      |-- paypal
|-- nodes (limited api)
|-- organizations
|   |-- nodes
|   |-- occasions
|   |-- containers
|   |-- groups
|   |-- permissions
|   |-- surveys
|   |   |-- node
|   |-- registrations
|   |   |-- node
|   |   |-- offering
|   |   |-- user
|   |   |-- offline
|   |-- payments
```

## Users
---
>NOTE: This may or not be wrapped in API. The functionality already exists in Cognito and we may just interact directly with Cognito for simplicity at the moment.

`GET /users`  - return all users. *This should not be implemented or only at super admin*

`GET /users/:id` - return single user

`PUT /users/:id` - update single user info

`POST /users` - create a new user, needs to be coordinated with Cognito process

`DELETE /users/:id` - deactivate a user

`GET /users/:id/registrations` - get all registrations for a specific user

### Payments
>Assume for now same verbs/functions between Stripe and PayPal - represented as `[service]`

`POST /users/:id/payments/[service]` - create a charge for a transaction


## Organizations

---

`GET /organizations` - return all orgs (super user and/or implement filtering)

`GET /organizations/:id` - return specific organization

`POST /organizations` - create a new organization

`PATCH /organizations` - update an organization(i.e. make active/inactive, change other attrs, etc)

`DELETE /organizations/:id` - deactivate organization

### Nodes
>Any object that is a node on functional tree - can be an occasion or container

`GET /organizations/:org_id/nodes` - return all nodes belonging to `:org_id`  
`GET /organizations/:org_id/nodes?isRoot=true` - return all root level nodes belonging to `:org_id`

`GET /organizations/:org_id/nodes/:id` - return node and children if present

`PUT /organizations/:org_id/nodes/:id` - update node
```
{
    "public": true // html body sets node to be public.
    "active": true // html body sets node to be active
    , "offering" : { // html body sets node to be an offering
       // with the following options
       "price": 1000 // $10
       , "startDate" : TIMESTAMP // when offering begins
       , "endDate" : TIMESTAMP // when it ends
       , "cancelationDeadline": TIMESTAMP // can't be canceled after this date
       , "modificationDeadline": TIMESTAMP // information (survey responses) can't be modified after this
       , "conditionalPrices": {} // future nice to have
       , ...
    }
    , "surveys": [ "surveyId1", "surveyId2", ... ] // set surveys
}
```

`DELETE /organizations/:org_id/nodes/:id` - delete a node, only available prior to offering

### Occasions and Containers
>Occasions are a node that represents one thing happening in one place (real or virtual) at one time.  
>Containers are nodes that contain other nodes.

`POST /organizations/:org_id/occasions` - create a new occasion

`POST /organizations/:org_id/containers` - create a new container

`PATCH /organizations/:org_id/containers/:id/children` - **ADD** nodes `childNodeId1` and `childNodeId2` to container with `id`
```
[ "childNodeId1", "childNodeId2", ... ]
```

`PUT organizations/:org_id/containers/:id/children` - **SET** nodes `childNodeId1` and `childNodeId2` to container with `id`
```
[ "childNodeId1", "childNodeId2", ... ]
```

`DELETE organizations/:org_id/containers/:id/children/:id2` - remove node `id2` from container with `id`

### Groups
>Groups are organizational containers, but cannot be used for functional node purposes such as registration.  
>Groups can be used for administrative grouping, such as for organization on the GUI.  
>Groups are a core component for assigning access control to users inside an organization.  
>Groups **cannot** be nested for initial implementation. Later implementation may allow for nesting of groups in the same manner as nodes are arranged in trees.  

`POST /organization/:org_id/groups` - create a new group

`PATCH /organizations/:org_id/groups/:id/nodes` - **ADD** nodes nodeId1 and nodeId2 to group with `id`
```
[ "nodeId1", "nodeId2", ... ]
```

`DELETE /organizations/:org_id/groups/:id` - remove or deactivate group with `id`

### Permissions
>Access control for organizations.  
>An `org_id`:`user_id` pair will contain a set of `group_id`:`access_level` pairs. The `group_id` can be ***** to represent an access level the user has on all groups in the organization.  
>For now, access levels will not be user defined - however applying multiple access levels to a user in a group will combine to the **least** restrictive combination of those access levels. We will define access levels such as "Owner", "Organization Admin", "Bookkeeper", "Event Admin", etc.

`POST organizations/:org_id/permissions/:user_id` - add user with `user_id` to set of authorized users for the organization. This will by default give them `[{ "*":"none" }]` access. Give option to include `[permissions]` inside body to set initial permissions.

`GET organizations/:org_id/permissions/:user_id` - return the `[permissions]` set for the user with `user_id`

`PATCH organizations/:org_id/permissions/:user_id/` - **ADD** the `group_id`:`access_level` pairs listed in `[permissions]` inside the request body to the user with `user_id`.  
**NOTE:** Multiple access levels can be applied to one user inside one group. Therefore there is only **ADD** and **DELETE** functionality. A **SET**, i.e. changing the access level would functionally be a **DELETE** of the existing pair, and an **ADD** of the new pair.

`PUT organizations/:org_id/permissions/:user_id` - **DELETE** an existing `group_id`:`access_level` pair in the user with `user_id` `[permissions]`.

`DELETE organizations/:org_id/permissions/:user_id` - remove the user with `user_id` from the set of authorized users for the organization.

### Surveys

`POST /organizations/:org_id/surveys` - create a new survey

`GET /organizations/:org_id/surveys/:id` - return a specific survey

`GET /organizations/:org_id/surveys?root={:id}` - get the group of surveys from all nodes on the tree below `:id`

`PUT /organizations/:org_id/surveys/:id` - update a specific survey


## Registrations

---

`GET /organizations/:org_id/registrations/:id` - get a specific registration

`GET /organizations/:org_id/registrations?root=:id` - get all registrations beneath a specific node on a tree

`POST /organizations/:org_id/registrations` - attempt to register for offering `:id`
```
{ "offering": "OFFERING_ID", ... }
```

`POST /organizations/:org_id/registrations` - batch of registrations from offline source, admin only, `content-type: text/csv`

`PUT /organizations/:org_id/registrations/:id` - update limited set of information - likely only admin

`DELETE /organizations/:org_id/registrations/:id` - cancel a specific registration, optionally issues refunds

## Payments

---

`GET /organizations/:org_id/payments/:id` - get a specific payment receipt

`PUT /organizations/:org_id/payments/:id` - update a payment. If it's associated with a vendor (stripe), this should be restricted to issuing partial refunds only.

`POST /organizations/:org_id/payments` - create a payment. restricted to "offline" payments (cash at door)

`DELETE /organizations/:org_id/payments/:id` - issue a refund for payment


# SUPER ADMIN

## Nodes

---

`GET /nodes` - super admin, retrieve all nodes. filter by `?offering`, `?isCollection=true`, etc

`GET /nodes?region=GEOCODE` - get node in a region
