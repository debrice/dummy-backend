# Dummy-backend

A ready-to-go REST API compliant server so you can prototype your front end. Just call the endpoint you wish you had and it’s available on the fly.

- No database to setup
- No proxy server
- No VM
- No authentication

The server will create and persist tables and records as you need them. You can
create, update, delete, get, list (including query, sorting, pagination) using the standard CRUD requests you know (see below).

When you're satified with your solution, you can start building the backend that matches your feature and not the opposite.

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

### Arguments

The start call accepts the following arguments:

| argument | alias | description                              | default       |
| -------- | ----- | ---------------------------------------- | ------------- |
| --port   | -p    | set the serving port                     | 8080          |
| --file   | -f    | set the filename where data is persisted | database.json |
| --delay  | -d    | set the delay prior to sending response  | 0             |

So if you wanted to load the data stored in `foo.json` on port `8000` you'd use the following command:

```
$ yarn start -d foo.json -p 8000
4/2/2020, 8:11:45 PM Serving request at http://localhost:8000
4/2/2020, 8:11:45 PM Loading data from foo.json ...
4/2/2020, 8:11:45 PM Loading data done.
```

## Interactions

By default the server will run on port 8080 on your localhost. The convention is having the resource name as the first part of the URL followed by its optional ID:

```
http://localhost:8080/<resource>/<resource ID>
```

For example, if you wanted to interact with a users resource, you would use the following URLs:

| Method | URL        | description                         |
| ------ | ---------- | ----------------------------------- |
| GET    | /users/123 | return the user with ID 123         |
| PUT    | /users/123 | updates the user with ID 123        |
| DELETE | /users/123 | delete the user with ID 123         |
| POST   | /users     | create a user (ID is autogenerated) |
| GET    | /users/    | return a list of users (more below) |

## Creation

Every object created will be augmented with 3 attributes:

- `id`: The uniq identifier, here as UUID
- `createdAt`: A ISO date string (so it's sortable and comparable)
- `updatedAt`: A ISO date string (so it's sortable and comparable)

## Update

Update will also update the `updatedAt` to the current date.
Note that, when updating, the object needs to exists or you'll get a 404 response.

## Listing

Listing often allows for a few additional actions like, pagination, filtering and sorting. So, here you go...

### Listing query arguments

Every listing endpoint accepts the following query arguments:

| argument      | description                                | example             | default |
| ------------- | ------------------------------------------ | ------------------- | ------- |
| page          | The page number starting at 0              | ?page=0             | 0       |
| pageSize      | the number of items per page               | ?pageSize=5         | 10      |
| sortBy        | The attribute to sort the listing by       | ?sortBy=createdAt   | -       |
| sortDirection | Sorting direction when sortBy is provided  | ?sortDirection=DESC | ASC     |
| filter_xxx    | Filter record that must contains the value | ?filter_name=john   | -       |

_Note regarding the filters_: you can combine them as long as they target different attributes. Filter values are interpreted as a regular expression.

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
