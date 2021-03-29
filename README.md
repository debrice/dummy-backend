# Dummy-backend

A ready-to-go REST API compliant server so you can prototype your front end. Just call the endpoint you wish you had and it’s available on the fly.

- No database to setup
- No proxy server
- No VM
- No authentication

The server will create and persist tables and records as you need them. You can
create, update, delete, get, list (including query, sorting, pagination) using the standard CRUD requests you know (see below).

When you're satified with your solution, you can start building a backend that matches your features and not the opposite.

## Install

Clone this repository

```sh
git clone git@github.com:debrice/dummy-backend.git
```

Install the dependencies using `yarn`

```sh
cd dummy-backend
yarn
```

Run the server using `yarn start`

```sh
yarn start
```

### Options

The start call accepts the following arguments:

| argument   | alias | description                      | default       |
| ---------- | ----- | -------------------------------- | ------------- |
| --port     | -p    | serving port                     | 8080          |
| --file     | -f    | filename where data is persisted | database.json |
| --delay    | -d    | delay prior to sending response  | 200           |
| --hostname | -n    | hostname for serving             | localhost     |

So if you wanted to load the data stored in `foo.json` on port `8000` you'd use the following command:

```
$ yarn start -d 150 -n 192.168.1.130 -f foo.json -p 8000
yarn run v1.12.3
...
[4/17/2020, 12:56:52 PM] Using port 8000
[4/17/2020, 12:56:52 PM] Database file is foo.json
[4/17/2020, 12:56:52 PM] Response delay is set to 150 ms
[4/17/2020, 12:56:52 PM] Serving hostname is set to 192.168.1.130
[4/17/2020, 12:56:52 PM] Serving request at http://192.168.1.130:8000
[4/17/2020, 12:56:52 PM] Loading data from foo.json ...
[4/17/2020, 12:56:52 PM] Loading data done.
```

## Interactions

By default the server will run on port 8080 on your localhost. The convention is having the resource name as the first part of the URL followed by its optional ID:

```
http://localhost:8080/<resource>/<resource ID>
```

For example, if you wanted to interact with a `user` resource, you would use the following URLs:

| Operation                      | Method | URL        | description                         |
| ------------------------------ | ------ | ---------- | ----------------------------------- |
| [Get](#get-a-single-record)    | GET    | /users/123 | return the user with ID 123         |
| [Replace](#replace-a-record)   | PUT    | /users/123 | replaces the user with ID 123       |
| [Update](#update-a-record)     | PATCH  | /users/123 | updates the user with ID 123        |
| [Delete](#delete-a-record)     | DELETE | /users/123 | delete the user with ID 123         |
| [Create](#create-a-new-record) | POST   | /users     | create a user (ID is autogenerated) |
| [Get many](#listing)           | GET    | /users/    | return a list of users (more below) |

There is also additional "feature" endpoints. Feature endpoints don't have a `<resource>` root and start with a `_`.

| Operation  | Method | URL                      | description                         |
| ---------- | ------ | ------------------------ | ----------------------------------- |
| Upload     | PUT    | /\_upload                | uploads a file to the upload folder |
| Get a File | GET    | /\_upload/upload_123.jpg | retrieve a previously uploaded file |

## Create a New Record

This is a cURL request creating a new user

```sh
curl --request POST "http://localhost:8080/users" \
  --header "Content-Type: application/json" \
  --data '{"name": "John Doe"}'
```

And the created user as a JSON response (`HTTP/1.1 200 OK`):

```json
{
  "status": "success",
  "data": {
    "name": "Jonh Doe",
    "id": "9b3521a4-0e08-471b-8434-19fcb2cc5899",
    "createdAt": "2021-03-29T14:03:42.293Z",
    "updatedAt": "2021-03-29T14:03:42.293Z"
  }
}
```

Every object created will be augmented with 3 attributes:

- `id`: The uniq identifier, a UUID string like `"d7699f21-902e-4d10-bf3c-e6c74c99f310"`
- `createdAt`: A ISO date string like `"2021-03-29T13:28:59.013Z"`
- `updatedAt`: A ISO date string like `"2021-03-29T13:28:59.013Z"`

## Update a Record `PATCH`

Update will combine the data provided with the existing record.
It will also update the `updatedAt` to reflect current date.
Note that, when updating, the object needs to exist or you'll get a 404 response.

If we wanted to only update a given record (here the previously created user),
we could use PATCH as follow:

```sh
curl --request PATCH "http://localhost:8080/users/9b3521a4-0e08-471b-8434-19fcb2cc5899" \
  --header "Content-Type: application/json" \
  --data '{"age": "32"}'
```

JSON response would be:

```json
{
  "status": "success",
  "data": {
    "name": "John Doe",
    "id": "9b3521a4-0e08-471b-8434-19fcb2cc5899",
    "createdAt": "2021-03-29T14:03:42.293Z",
    "updatedAt": "2021-03-29T14:08:44.050Z",
    "age": "32"
  }
}
```

## Replace a Record `PUT`

Replaces an existing record (beside it's ID) with the data provided. Every missing
attribute will be removed from the record.
It will also update the `updatedAt` to the current date.
Note that, when updating, the object needs to exists or you'll get a 404 response.

## Get A Single Record

Returns a given record, the object needs to exists or you'll get a 404 response.

To retrieve a user:

```sh
curl --request GET "http://localhost:8080/users/9b3521a4-0e08-471b-8434-19fcb2cc5899" \
  --header "Content-Type: application/json"
```

JSON response would be:

```json
{
  "status": "success",
  "data": {
    "name": "John Doe",
    "id": "9b3521a4-0e08-471b-8434-19fcb2cc5899",
    "createdAt": "2021-03-29T14:03:42.293Z",
    "updatedAt": "2021-03-29T14:08:44.050Z",
    "age": "32"
  }
}
```

## Delete a Record `DELETE`

Deletes a given record. When deleting, the object needs to exists or you'll get a 404 response.

Example, deleting a record that does not exist (HTTP/1.1 404 Not Found):

```sh
curl --request GET "http://localhost:8080/users/does-not-exist" \
  --header "Content-Type: application/json"
```

```json
{
  "status": "error",
  "error": "not_found",
  "message": "[GET] /users/does-not-exist does not exist."
}
```

Example when the record exist:

```sh
curl --request GET "http://localhost:8080/users/9b3521a4-0e08-471b-8434-19fcb2cc5899" \
  --header "Content-Type: application/json"
```

JSON response contains the deleted data:

```json
{
  "status": "success",
  "data": {
    "name": "John Doe",
    "id": "9b3521a4-0e08-471b-8434-19fcb2cc5899",
    "createdAt": "2021-03-29T14:03:42.293Z",
    "updatedAt": "2021-03-29T14:08:44.050Z",
    "age": "32"
  }
}
```

## Listing `GET`

Listing often allows for a few additional actions like, pagination, filtering and sorting. So, here you go...

### Listing query arguments `GET`

Every listing endpoint accepts the following query arguments:

| argument      | description                               | example                                            | default |
| ------------- | ----------------------------------------- | -------------------------------------------------- | ------- |
| page          | page number starting at 0                 | ?page=0                                            | 0       |
| pageSize      | number of items per page                  | ?pageSize=5                                        | 10      |
| sortBy        | attribute to sort the listing by          | ?sortBy=createdAt                                  | -       |
| sortDirection | sorting direction when sortBy is provided | ?sortDirection=DESC                                | ASC     |
| filter        | filter records                            | ?filter=name:john&filter=createdAt[gte]:2020-05-01 | -       |

### Listing filters

You can add as many filter as you want on the query. Every filter is composed as follow:

```

filter=<attribute>[operator]:<value>&filter=<attribute>[operator]:<value>&...

```

| operator | description                                 |
| -------- | ------------------------------------------- |
| eq       | equal, case sensitive (default)             |
| gt       | greater than                                |
| gte      | greater than or equal                       |
| lt       | lesser than                                 |
| lte      | lesser than or equal                        |
| reg      | regular expression                          |
| has      | contains specified value (case insensitive) |

### Listing Response

Since the listing endpoint accepts arguments for pagination, it makes sense to receive a response that allows you to display proper pagination.

A call to a [listing endpoint of users](http://localhost:8080/users/?pageSize=2&sortBy=id&sortDirection=DESC) would return:

```json
{
  "status": "success",
  "data": [
    {
      "email": "wat@example.com",
      "name": "John what",
      "id": "f3a0c7c7-b9b0-4f91-a485-a643d653508a",
      "createdAt": "2020-04-02T19:58:26.021Z",
      "updatedAt": "2020-04-02T19:58:26.021Z"
    },
    {
      "email": "jane@example.com",
      "name": "Jane Doe",
      "id": "f0ccc71e-9411-454d-895a-3dc528e78872",
      "createdAt": "2020-04-02T19:58:38.601Z",
      "updatedAt": "2020-04-02T19:58:38.601Z"
    }
  ],
  "total": 10,
  "pageSize": 2,
  "page": 0,
  "sortBy": "id",
  "sortDirection": "DESC"
}
```

## Persistence

By default the database is stored in memory and is backed up to every one seconds in a
file (default to `database.json`).
The database is loaded back in memory when the server starts.
