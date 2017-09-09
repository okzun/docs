# API Structure

## Errors

### 404 Not Found

Should be returned when:
- a resource is missing
- an incorrect url is hit
- a request is **unauthorized**. DO NOT USE 401 for Authorization feedback. This gives attackers too much information.

## Users

`GET /user/{id}` - gets user account data

**params**
- `id`: the user id


`PUT /user/{id}` - update user account data

**params**
- `id`: the user id

**body**
```json
{
  name: 'string',
  billing_address: {
    address_line: 'string',
    address_line2: 'string',
    city: 'string',
    region: 'string',
    country: 'string',
    postal_code: 'string'
  }
  ...
}
```
