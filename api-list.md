# API

## Conventions

* Root resource names are plural
  * Related resources being acted on via a root resource are singular
    * e.x. `surveys/node`
* Resource updates are performed via `POST` with JSON input bodies
  * Avoid using URL query parameters
* Verb actions that update flag-type states can be included under resources
    * e.x. `node/publish`

## Resource Structure

```
API Root Path  
|-- users
|-- organizations
|   |-- nodes
|   |-- occasions
|   |-- containers
|   |-- surveys
|   |   |-- node
|-- offerings
|   |-- survey
|-- registrations
|   |-- node
|   |-- offering
|   |-- user
|   |-- offline
|-- payments
```
  
## Users

---

`GET /users`  - return all users. *This should not be implemented or only at super admin*

`GET /users/:id` - return single user

`PUT /users/:id` - update single user info

`POST /users` - create a new user, needs to be coordinated with Cognito process

`DELETE /users/:id` - delete a user, super admin level only? If we let users delete themselves, it would screw with existing and historical registrations.
`PUT /users/:id/deactivate` - alternative at user level?

## Organizations

---

`GET organizations/` - return all orgs (super user and/or implement filtering)

`GET organizations/:id` - return specific organization

`POST organizations/` - create a new organization

### Nodes

`GET organizations/:org_id/nodes` - return all nodes belonging to `:org_id`

`GET organizations/:org_id/nodes/:id` - return node and children if present (same as /offerings/:id)

`PUT organizations/:org_id/nodes/:id` - update node

`PUT organizations/:org_id/nodes/:id/publish` - set node as publicly visible
`PUT organizations/:org_id/nodes/:id/unpublish` - set node as invisible, will need lots of checking (can’t be done once offered, or part of an offered tree)

`PUT organizations/:org_id/nodes/:id/offer` - set node to an offering (can be registered for)
`PUT organizations/:org_id/nodes/:id/deoffer` - not implemented? Not implemented once there are registrations?

`DELETE organizations/:org_id/nodes/:id` - delete a node, only available prior to offering

#### Occasions and Containers

`POST organizations/:org_id/occasions` - create a new occasion

`POST organizations/:org_id/containers` - create a new container

`PUT organizations/:org_id/containers/:id?:id2` - add node `id2` to container with `id`

`DELETE organizations/:org_id/containers/:id?:id2` - remove node `id2` from container with `id`

#### Surveys

`POST organizations/:org_id/surveys` - create a new survey

`GET organizations/:org_id/surveys/:id` - return a specific survey

`PUT organizations/:org_id/surveys/:id` - update a specific survey

`PUT organizations/:org_id/surveys/:id/node/:id2` - attach survey `:id` to node `:id2`

`GET organizations/:org_id/surveys/node/:id` - return all attached surveys in tree starting at node `:id`



## Offerings

---

`GET /offerings` - super admin, retrieve all offerings

`GET /offerings/:region` - get offerings in a region

`GET /offerings/:id` - get a specific offering and all children (wraps /nodes/:id)

`PUT /offerings/:id/deoffer` - not implemented? (see above)

`GET /offerings/:id/survey` - get the group of surveys from all nodes on the tree below `:id`

## Registrations

---

`GET /registrations/:id` - get a specific registration

`GET /registrations/node/:id` - get all registrations beneath a specific node on a tree

`GET /registrations/user/:id` - get all registrations for a specific user

`POST /registrations/offering/:id` - attempt to register for offering `:id`

`POST /registrations/offline` - batch of registrations from offline source, admin only

`PUT /registrations/:id` - update limited set of information - likely only admin

`DELETE /registrations/:id` - delete a specific registration, admin only, possibly super admin only

## Payments

---

`GET /payments/:id` - get a specific payment receipt

`GET /payments/organizations/:id?` - get payment receipts belonging to an organization, implement sort parameters.

`POST /payments` - create a payment receipt

`PUT /payments/:id` - update a payment, super admin, possibly admin, would need to wrap specific payment processor API for full handling
