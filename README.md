# Speca Web API specification format (DRAFT v0.0)

- [Привет](#Привет)
- [Concept](#concept)
- [File layout](#file-layout)
- [Index file](#index-file)
- [Documentation index file](#documentation-index-file)
- [Model index file](#model-index-file)
- [Methods index file](#methods-index-descriptor)

## Привет

## Concept

- MULTIPLE FILES -      _well organized files and directories layout. Main file - **index.yml**_
- YAML -                _both human and computer readable definitions, comments support, docmentation may be either inlined or moved to separate text file_
- MARKDOWN -            _rich documentation_
- JSON SCHEMA -         _data model definition_

add
## WEB API components

### Rich documentation

The primary goal of Speca platform is to create WEB API specifications full of rich human readable documentation.

Speca specification format is designed around this requirement - all descriptions and notes within specification can be written in **markdown**
and can be either inlined in a YAML file or moved to separate .md file and referenced in YAML.

Description or notes can be added not just to every method or data model but to literally every method definition part like query parameters or response.

### Method

Web API Method or operation is a main API component. Method definition includes:

- **name** - _Create a user_
- **HTTP method** - _GET, POST, PUT etc..._
- **URI path** - _/user/{id}_
- **path variables** - _`id` - user identifier or username_
- **query parameters** - query, page, size, etc...
- **request body** (if applicable)
- **successful response - http status, data type**
- **errors list**
- **examples**


### Group

API methods may be organized into groups, e.g. **Users**, **Movies**. Very often group can be used as a **Resource** in terms of **REST** but it is not restricted to.

### Envelope

Envelope is a special type of data model which is used to define generic response wrappers e.g.

`GET  /users` returns array of users - [user1,user2,...]
`GET  /pets` returns array of pets - [pet1,pet2,...]

Envelopes allow us to define common wrapper for both responses, consider the example:
```json
Envelope:
{
    "status" : "ok",
    "size"   : 5,
    "total"  : 500,
    "data"   : [DATA CONTAINER]
}
```
where `DATA CONTAINER` is special type defines that particular response data will be placed to `data` field of envelope object; the data could be either `LIST OF USERS` or `LIST OF PETS` or something else

Utilizing envelope feature we can define `GET  /users` response as: `Array` of `Users` wrapped with `Envelope`
and `GET  /pets` response as  `Array` of `Pets` wrapped with `Envelope`

Envelopes eliminate us from creation of separate data types for each business entity, e.g: ListUsersResponse or ListPetsResponse.

### Custom attributes

Custom attributes are designed to provide extension mechanism for API method definition. For example we want to mark few methods as deprecated because we are going to remove them in nearest six months.  We can walk through methods and add additional notes, like `Deprecated`.
  Speca provides alternative and generic approach - you need to define new custom attribute:
```yaml
  name: Deprecated
  type: Boolean
```
and add this attribute to desired methods. Speca provides methods filtering by custom attribute values so you will be able to easily find them and remove.

Another example could be implementation status - `Done`, `In progress`,`Initial`
// todo


## File layout

- index.yml            # or speca.yml
- api.json             # robots.txt for APIs
- documentation
  - docs.yml
  - intro.md
  - authentication.md
  - rate_limiting.md
  - ...
- model
  - model.yml
  - user.json
  - pet.json
  - ...

- methods              # resources/operations
  - methods.yml

  - user               # methods group *Users*
    - create_user.yml         # or folder
    - update_user.yml
    - notes
      - create_user_notes.md
      - create_user_param_notes.md
      - create_user_response_notes.md
      - update_user_notes.md
    - examples
      - create_user_example_1.yml
      - create_user_example_2.yml

  - pet                # methods group *Pets*
    - list_pets.yml


## Index file

Index file description...

```yaml
---
baseUrl: https://api.demo.com/{v1}
baseUrlVariables:
    - name : v1
      description : API version
      value: v1.0

mediaTypes:
  supported:
    - name: application/json
    - name: text/xml
  default:  application/json

authentication:
  supported:
    - name: basic
    - name: token
      description: header
      type: header  # header or parameter
      headerName: X-Auth
  default: all      # none, all, or one of supported

attributes:
  - name: Deprecated
    description: Mark methods as deprecated for usage because they will be removed in nearest future
    type: Boolean
  - name: Status
    description: Implementation status
    type: Enum
    values: Done, In Progress, None

params:
  - name : pretty
    description : |
                  **Pretty** print `json`
    type: Boolean
    example: true
    applied_to_all: true     # PARAMETER IS APPLIED TO ALL METHODS WITHIN SPECIFICATION

  - name : page
    description : Used in pair with `count` to navigate through big result sets
    type: Boolean
    required: false
    default: 1
    example: 10
  - name : count
    description : Used in pair with `page` to navigate through big result sets
    type: Boolean
    required: false
    default: 50
    example: 10


headers:
  - name : X-Auth
    description : Authentication header
    example: access_token
    applied_to_all: true     # HEADER IS APPLIED TO ALL METHODS WITHIN SPECIFICATION


errors:
  - name : Not Found
    status : 404
    description: Requested resource not found
    applied_to_all: true     # ERROR IS APPLIED TO ALL METHODS WITHIN SPECIFICATION
    type : $ErrorModel

  - name : Access Denied
    status : 403
    description: User is not uuthorized to access requested content
    applied_to_all: true     # ERROR IS APPLIED TO ALL METHODS WITHIN SPECIFICATION
    type : $ErrorModel


```

## Documentation index file

```yaml
---
title: Inline
content: |
  Markdown content
  ABCD
  **Markdown** content
---
title: Introduction
file: auth.md
---
title: Authentication
file: auth.md
---
title: Rate Limiting
file: rate-limiting.md
```

## Model index file

```yaml
---
id: Pet                                        # Data model identifier
description: Model description in `markdown`   # Data model description/notes, ether string or path to .md file
file: pet.json                                 # JSONSchema file
type: model                                    # model (Default), error, envelope

---
id: User
file: user.json

---
title: GenericResponse
type: envelope

---
id: ErrorModel
type: error            # Error type is used to define error responses types
schema: |
  {
    jsonSchema is here
  }
```


## Methods index file

Method definition description

```yaml
---
groups:
  - name: Pets
    description: Group description

  - name: Users
    description: Group description

---
file: status.yml
```

## Method descriptor


```yaml
---
id:    123
name:  Create a user
method: POST
path: /users/{domain}
pathVariables:
    - name: domain
      value: github
      description: |
                   Markdown is `here`
notes:  >
        Follow the **Yellow** Brick
        Road to the Emerald City.
        Pay no attention to the
        man behind the curtain.
params:
    notes: 123/param-notes.md
    list:
        - name: query
          description: Query param description in *markdown*
          type: -1
          value: cool
request:
    notes: 123/request-notes.md
    type: User
response:
    notes: 123/response-notes.md
    status: 200
    type: User
errors:
    notes: 123/error-notes.md
    list:
        - name: NOT_FOUND
          description: Error description in *markdown*
          status: 404
          model: ErrorModel
          example: >
              {
                  "cool":1
              }
```
