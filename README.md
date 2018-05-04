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


Uploading data logs on anemolab
-------------------------------

(note: this API is not yet functional)

| HTTP command                    | description                                    |
| ------------------------------- | ---------------------------------------------- |
| GET /api/files/boatid(?s=DATE) | List uploaded file. Optionally specify a date. 
                                   If specified, only file uploaded after the     
                                   given date will be returned.                   
                                   The date is specified in the following format: 
                                   YYYY-MM-DDTHH:MM:SS                            
                                   For example: 2018-02-16T09:01:48               |
| POST /api/files/boatid         | Upload a new data log file                     |


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
| author    | string   | userid of the author                                     |
| comment   | string?  | Comment (free text) typed by the user. Optional.         |
| photo     | string?  | Identifier of a photo, UUID format. Optional.            |
| when      | Date     | Date/time of when the note/comment has been taken.       |
| latitude  | number   | Coordinate of the boat when the event took place (deg).  |
| longitude | number   | Coordinate of the boat when the event took place (deg).  |

### Add a new Event

Call POST /api/events with an Event object containing at least:
- author: the userid of the author
- when: the date/time of event
- either comment or photo containing

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

