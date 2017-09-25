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
|-- nodes (limited api)
|-- organizations
|   |-- nodes
|   |-- occasions
|   |-- containers
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

`GET /users`  - return all users. *This should not be implemented or only at super admin*

`GET /users/:id` - return single user

`PUT /users/:id` - update single user info

`POST /users` - create a new user, needs to be coordinated with Cognito process

`DELETE /users/:id` - deactivate a user, super admin level only? If we let users delete themselves, it would screw with existing and historical registrations.
`PUT /users/:id/deactivate` - alternative at user level?

## Organizations

---

`GET /organizations` - return all orgs (super user and/or implement filtering)

`GET /organizations/:id` - return specific organization

`POST /organizations` - create a new organization

`DELETE /organizations/:id` - deactivate organization

### Nodes

`GET /organizations/:org_id/nodes` - return all nodes belonging to `:org_id`  
`GET /organizations/:org_id/nodes?isRoot=true` - return all root level nodes belonging to `:org_id`

`GET /organizations/:org_id/nodes/:id` - return node and children if present (same as /offerings/:id)

`PUT /organizations/:org_id/nodes/:id` - update node
```
{ 
    "published": true // html body sets node to be published.
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

### Surveys

`POST /organizations/:org_id/surveys` - create a new survey

`GET /organizations/:org_id/surveys/:id` - return a specific survey

`GET /organizations/:org_id/surveys?root={:id}` - get the group of surveys from all nodes on the tree below `:id`

`PUT /organizations/:org_id/surveys/:id` - update a specific survey


## Registrations

---

`GET /organizations/:org_id/registrations/:id` - get a specific registration

`GET /organizations/:org_id/registrations/node/:id` - get all registrations beneath a specific node on a tree

`GET /organizations/:org_id/registrations/user/:id` - get all registrations for a specific user

`POST /organizations/:org_id/registrations/offering/:id` - attempt to register for offering `:id`

`POST /organizations/:org_id/registrations/offline` - batch of registrations from offline source, admin only

`PUT /organizations/:org_id/registrations/:id` - update limited set of information - likely only admin

`DELETE /organizations/:org_id/registrations/:id` - delete a specific registration, admin only, possibly super admin only

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

