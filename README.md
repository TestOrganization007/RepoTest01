Node+Express+Gulp+Sass+Livereload Starter Project
========

This is a starter project with Node, Express, Gulp, Sass, Livereload,
and a few other things like Redis and Mongo.

| Tables   |      Are      |  Cool |
|----------|:-------------:|------:|
| col 1 is |  left-aligned | $1600 |
| col 2 is |    centered   |   $12 |
| col 3 is | right-aligned |    $1 |

Prerequisites:
-------------
- [Vagrant](https://www.vagrantup.com/)
  - Then, do `vagrant plugin install vagrant-pristine`.
- [VirtualBox](https://www.virtualbox.org/)

Boot the vagrant box
--------------
- Boot it with `vagrant up`. Or smarter, do `vagrant pristine -f` (this destroys existing VM if any, and then boots it).
  The first time, this will download the base box which may take 10 minutes. Then some provisioning scripts will run,
  and you know it's done when it says "VAGRANT IS READY".
- SSH into it: `vagrant ssh`

Install packages (inside vagrant box)
----------------
- `npm install` - This has a post-install step which installs bower packages.
- `npm run bower_install` - If you ever change bower and you just want to reinstall packages.

Run (inside vagrant box)
----
- `USE_LIVERELOAD=yes npm start`
- You will now have the server running on http://localhost:3000 accessible from the host machine.

Sass, Livereload
--------------
- Frontend files are in frontend/, and if you modify HTML/CSS while the server is running, they are live-reloaded.
- The frontend/styles/css folder is generated from frontend/styles/scss contents. Don't modify anything in frontend/styles/css.

DevOps (Docker)
------
- docker-compose build
- docker-compose up
- docker-compose run -e BASE_URL=https://whatever.com web

Unit Tests
----------
- `npm test` runs unit tests with normal/fast behavior.
- `LARGE_TESTS=1 npm test` runs tests with longer/deeper checks.

QA Accounts
-----------
- See `test/data/seed_data.json`
- For testing purposes, the password for most QA accounts is **Passw0rd**. This password should of course never be used in production.

API
=====
General notes:
- Unless otherwise noted, GET params are on querystring, POST are in JSON body.
- Any API call can return error 500 (with explanation) if an unexpected error occurs.

Allowed only when **not** logged in
------
Note: If any of these routes are requested while logged in, it will result in an **error 403**.

**Authentication**
POST /api/login {email, password} **("Login to the application")**
- Returns: { user: {id, email, first_name, last_name } }, also sets the "logged_in" cookie to "1".
- Error 404 if no such email found
- Error 401 if wrong password
POST /api/invitation_redeem {email, password, code} **("Redeem an invitation")**
    - Returns: nothing. The user's account is now complete, with password
    - Error 401 if the error code does not match
    - Error 401 if the code is expired.
    - Error 403 if the password did not meet requirements
- POST /api/forgot_password {email} **("Forgot my password, request reset link, not logged in")**
    - Returns: nothing. The link to reset password (with secure code) is emailed to the user.
    - Error 404 if a user with that email address was not found.
    - Error 403 if the user with that email has not accepted their invitation yet.
- POST /api/reset_password {email, code, new_password} **("Use reset link and change my password")**
    - Returns: nothing. The user's password is now changed
    - Error 401 if the error code does not match
    - Error 401 if the code is expired.
    - Error 403 if new_password did not meet requirements

Allowed only when logged in:
--------
Note: If any of these routes are requested while **not** logged in, it will result in an **error 403**.

**Authentication**
- POST /api/logout **("Logout of application")**
    - Authorization: none.
    - Returns: nothing. The "logged_in" cookie will now be deleted.
- POST /api/invite {email, first_name, last_name, role} **("Invite a new user")**
    - Authorization: Must be "admin" or "hr".
    - Note: roles is a string, i.e. "admin" or "hr" or "none". Only "admin" can invite another "admin".
    - Returns: nothing. The link to accept the invitation (with secure code) is emailed to the user.
    - Error 403 if the email address already exists
    - Error 403 if the role doesn't match an expected value.
- POST /api/change_password {old_password, new_password} **("Change my password while logged in")**
    - Authorization: none.
    - Returns: nothing. The user's password is now changed. Like logout, the "logged_in" cookie will now be deleted.
    - Error 401 if the old password did not match.
    - Error 403 if the new password does not meet requirements.

**Users**
- GET /api/user/{id} **("Get a user")**
    - Authorization: none.
    - Returns: JSON object with user fields.
    - Error 404 if no user was found with that ID.
- GET /api/user/?q={search}&p={page} **("Search users")**
    - Authorization: none.
    - Note: "p" is optional, defaults to first page if not provided.
    - Returns: JSON object: {results:(array of objects with user fields), summary:{total_items, total_pages}}
- PATCH /api/user/{id} **("Update a user")**
    - Authorization: Must be "admin" or "hr".
    - Body: JSON object with fields to be updated. Omitted fields are not modified.
    - Returns: nothing. The user's fields are now updated.
    - Error 403 if an unexpected field or value was passed in. Note: Not allowed to change email or password fields.
- POST /api/user/{id}/deactivate **("Deactivate a user")**
    - Authorization: Must be "admin" or "hr".
    - Body: JSON object {new_password}
    - Returns: nothing. The user is now deactivated, password changed.
    - Error 403 if an unexpected field or value was passed in.
    - Error 403 if the new password does not meet requirements.
    - Error 403 if the new password is the same as the old password (it has to be different).

**Expo Permissions**
- GET /api/expo_perms/user/{id} **("What expos can this user access, and how?")**
    - Authorization: none.
    - Returns: JSON array of {user_id, expo_id, role} triplets.
    - Error 404 if no user was found with that ID.
- GET /api/expo_perms/expo/{id} **("What users can access this expo, and how?")**
    - Authorization: Must be "admin" or ExpoPerm="manager" for that expo.
    - Returns: JSON array of {user_id, expo_id, perm} triplets.
    - Error 404 if no expo was found with that ID.
- PUT /api/expo_perms/user/{user_id}/expo/{expo_id}/role/{expo_role} **("Add/update permission for user/expo/role")**
    - Authorization: Must be "admin" or ExpoPerm="manager" for that expo.
    - Returns: nothing. The permission is updated.
    - Error 404 if no user was found with that ID.
    - Error 404 if no expo was found with that ID.
- DELETE /api/expo_perms/user/{user_id}/expo/{expo_id} **("Remove permission for user/expo")**
    - Authorization: Must be "admin" or ExpoPerm="manager" for that expo.
    - Returns: nothing. The permission for this combo is removed.
    - Error 404 if no user was found with that ID.
    - Error 404 if no expo was found with that ID.

**Expos**
- GET /api/expo/{id} **("Get an expo")**
    - Authorization: "admin" or ExpoPerm=* for that expo (must exist)
    - Returns: JSON object with expo fields.
    - Error 404 if no expo was found with that ID.
- GET /api/expo/?q={search} **("Search expos")**
    - Authorization: none, but if not "admin", you will only see expos you have ExpoPerms for.
    - Note: "p" is optional, defaults to first page if not provided.
    - Returns: {results:(array of objects with expo fields), summary:{total_items, total_pages}}
- POST /api/expo **("Create a new expo")**
    - Authorization: Must be "admin"
    - Body: JSON object with fields.
    - Returns: nothing. The expo is now added to the system.
    - Error 403 if an unexpected field or value was passed in.
- PATCH /api/expo/{id} **("Update an expo")**
    - Authorization: Must be "admin" or ExpoPerm="manager" for that expo.
    - Body: JSON object with fields to be updated. Omitted fields are not modified.
    - Returns: nothing. The expo's fields are now updated.
    - Error 403 if an unexpected field or value was passed in.

**Synchronization**
- POST /upload **("Upload a file to S3 and return the key")**
    - Authorization: none.
    - Body: The uploaded data, encoded as multipart, and named "file"
    - Returns: {key: s3key}, the s3 key of the completed upload

**QA**
- POST /reseed {password} **("Drop database and reseed with QA data. Requires password confirm.")**
    - Authorization: "admin"
    - Returns: nothing. The database is now reseeded.
    - Error 401 if wrong password