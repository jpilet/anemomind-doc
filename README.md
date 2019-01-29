Anemomind API
=============

Anemolab is Anemomind's web platform to visualize navigation data.
This manual documents how to use the web API exposed by anemolab.com in order
to manage users and to upload navigation log files and comments.

Anemolab access control overview
--------------------------------

Users have to register and login.
Users can have read and write access to boats.
All navigation data is related to a boat.
Boat read access can be open to anyone (no login required) or restricted to users.

Anemolab authentication process
-------------------------------

1. Users have to register. This is done through HTTP POST on
   https://anemolab.com/api/users with a JSON object containing: 
  ```
  {
    "name":"User Name",
    "email":"user@email.address.com",
    "password":"clear text password"
  }
  ```
  Note that Content-Type should be set to application/json.

  In case of success, status will be 200 and the response will be similar to:
  ```
  {
   "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJfaWQiOiI1YWVjMTE2ZGVjOTM5OTA1NzYzZmUwNmUiLCJpYXQiOjE1MjU0MjAzOTcsImV4cCI6MTUyNTQzODM5N30.1NJ4q4gy1-ydhJ9NoFQVARIYWhw02ahhZdHctA_S_T0",
   "user": {
    "_id": "5aec116dec939905763fe06e",
    "name": "User Name",
    "role": "user"
   }
  }
  ```

  In case of error, status will be 422 and a JSON object describing the error
  will be returned.

2. Registered users can login with an HTTP POST on https://anemolab.com/auth/local
   with data:
  ```
  {
    "email":"user@email.address.com",
    "password":"clear text password"
  }
  ```

  In case of success, status will be 200 and the response will be the same as
  after creating a user.


  curl example:
  ```
  curl \
    -H "Content-Type: application/json"  \
    --request POST \
    --data '{"email":"user@domain.com","password":"XXX"}' \
    https://anemolab.com/auth/local
```


3. Registered users can use their token to do registered queries by adding the
   following header to the request:
   ``` Authorization: Bearer $token ```
   For example:
   ```
   Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJfaWQiOiI1YWVjMTE2ZGVjOTM5OTA1NzYzZmUwNmUiLCJpYXQiOjE1MjU0MjA5MTQsImV4cCI6MTUyNTQzODkxNH0.We4vnrAoTjr6xBhZD56qYPasRuuv_b5W4Kaw2k72uxI
   ```

4. To test the validity of a token, use https/anemolab.com/api/users/me
   Response example:
   ```
   {
     "_id":"5aec116dec939905763fe06e",
     "provider":"local",
     "name":"User Name",
     "email":"user@email.address.com",
   }
   ```

Boats API
---------

API entries:
- ```GET /api/boats```: lists all the boat the user has access to.
- ```GET /api/boats/boatid```: get boat details and sailing sessions for the given boat
- ```POST /api/boats```: creates a new boat
- ```PUT /api/boats/boatid```: updates an existing boat
- ```PUT /api/boats/boatid/invite```: invites a new member to get access to the boat.

The Boat data structure:

| Field        | Type       | Description                                        |
| ------------ | ---------- | -------------------------------------------------- |
| _id          | string     | the boat identifier (boatid)                       |
| name         | string     | boat name                                          |
| publicAccess | boolean    | true iif boat access is public. Otherwise, access  |
|              |            | is restricted                                      |
| anemobox     | string     | serial number of the connected anemobox            |
| length       | string     | boat length                                        |
| lengthUnit   | string     | units of the "length" field: "meter" or "feet"     |
| sailNumber   | string     | race identifier written in the sails               |
| type         | string     | boat type                                          |
| invited      | [string]   | list of invited users                              |
| readers      | [User]     | list of users with read-only access                |
| admins       | [User]     | list of users with read-write access               |
| sails        | [string]   | list of sail configurations                        |


Uploading data logs
-------------------

| HTTP command                     | description                                    |
| -------------------------------- | ---------------------------------------------- |
| POST /api/files/boatid           | Upload a new data log file (or files)          |
| GET /api/files/boatid            | List uploaded file. Optionally specify a date. |
| GET /api/files/boatid/filename   | Get details about <filename>                   |
| DELETE /api/files/boatid/filename| Delete <filename>                              |

POST to /api/files/boatid has to be made using Content-Type: multipart/form-data.

Uploads pictures and comments
-----------------------------

| HTTP command                    | description                                    |
| ------------------------------- | ---------------------------------------------- |
| GET /api/events/$boatid         | Get all Event objects for a given boat         |
| POST /api/events                | Add a new Event                                |
| GET /api/events/$eventid        | Get a particular event                         |
| POST /api/events/photo/$boatid  | Add a photo to the given boat                  |
| GET /api/events/photo/$photoid  | Get a photo or a thumbnail                     |
| POST /api/events/photo/$photoid | Upload a new photo                             |

### Event data structure

| Field     | type     | description                                              |
| --------- | -------- | -------------------------------------------------------- |
| boat      | string   | the boat id the comment is attached to                   |
| author    | string   | userid of the author                                     |
| comment   | string?  | Comment (free text) typed by the user. Optional.         |
| photo     | string?  | Identifier of a photo, UUID format. Optional.            |
| when      | Date     | Date/time of when the note/comment has been taken.       |
| latitude  | number   | Coordinate of the boat when the event took place (deg).  |
| longitude | number   | Coordinate of the boat when the event took place (deg).  |

### Add a new Event

Call POST /api/events with an Event object containing at least:
- boat: the boat id the comment should be attached to
- author: the userid of the user who wrote the comment
- when: the date/time of event. [Format][https://tools.ietf.org/html/rfc2822#page-14] UTC is assumed if no timezone is specified.
- comment: the text of the comment (optional)
- photo: a string containing the filename (UUID.jpg) of the picture (optional)

For example, to post "Hello world", you would do a request similar to this:
```
curl  \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJ...Yhw" \
  --request POST \
  --data '{"when": "2016-09-19T11:45:00Z", "comment":"Hello, world!", "author":"560d07bbcd1054542579f5a3", "boat":"5aabe889cda9b6b0de4ed194"}' \
  https://anemolab.com/api/events
```

### Get a photo or a thumbnail

To view a photo from an img tag, the authorization token can be passed in
the parameter:
```
https://anemolab.com/api/events/photo/[boat]/[picture].jpg?access_token=[token]
```
to get a 120x120 thumbnail, simply use:
```
https://anemolab.com/api/events/photo/[boat]/[picture].jpg?s=120x120&access_token=[token]
```
To make a thumbnail of a fixed width, but preserving the aspect ratio,
simply replace the height by '_':  ```s=240x_```
Similarly, to have a thumbnail of a fixed height but variable width, use:
```s=_x300```


### Upload a new photo

Call POST /api/events/photo/boatId with a multipart/form-data request containing one or more JPEG file.
Each file has to be named with a valid UUID, followed by '.jpg'.

Here's the regexp validating the filename:
```
[A-F0-9]{8}-[A-F0-9]{4}-4[A-F0-9]{3}-[89AB][A-F0-9]{3}-[A-F0-9]{12}\.jpg
```

Note: posting twice with the same UUID will overwrite the photo.
Note: UUID are considered case insensitive. Upper cases are equivalent to their lower case counterpart.
