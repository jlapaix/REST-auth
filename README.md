REST-auth
=========

REST authentication for MagicStore

Installation
------------

After cloning, create a virtual environment and install the requirements. For Linux and Mac users:

    $ virtualenv venv
    $ source venv/bin/activate
    (venv) $ pip install -r requirements.txt

If you are on Windows, then use the following commands instead:

    $ virtualenv venv
    $ venv\Scripts\activate
    (venv) $ pip install -r requirements.txt

Running
-------

To run the server use the following command:

    (venv) $ python api.py
     * Running on http://127.0.0.1:5000/
     * Restarting with reloader

Then from a different terminal window you can send requests.

API Documentation
-----------------

- POST **/api/users**

    Register a new user.<br>
    The body must contain a JSON object that defines `username` and `password` fields.<br>
    On success a status code 201 is returned. The body of the response contains a JSON object with the newly added user. A `Location` header contains the URI of the new user.<br>
    On failure status code 400 (bad request) is returned.<br>
    Notes:
    - The password is hashed before it is stored in the database. Once hashed, the original password is discarded.
    - In a production deployment secure HTTP must be used to protect the password in transit.

- GET **/api/users/&lt;int:id&gt;**

    Return a user.<br>
    On success a status code 200 is returned. The body of the response contains a JSON object with the requested user.<br>
    On failure status code 400 (bad request) is returned.

- GET **/api/token**

    Return an authentication token.<br>
    This request must be authenticated using a HTTP Basic Authentication header.<br>
    On success a JSON object is returned with a field `token` set to the authentication token for the user and a field `duration` set to the (approximate) number of seconds the token is valid.<br>
    On failure status code 401 (unauthorized) is returned.

- GET **/api/resource**

    Return a protected resource.<br>
    This request must be authenticated using a HTTP Basic Authentication header. Instead of username and password, the client can provide a valid authentication token in the username field. If using an authentication token the password field is not used and can be set to any value.<br>
    On success a JSON object with data for the authenticated user is returned.<br>
    On failure status code 401 (unauthorized) is returned.

Example
-------

The following `curl` command registers a new user with username `test_user` and password `pwd`:

    $ curl -i -X POST -H "Content-Type: application/json" -d '{"username":"test_user","password":"pwd"}' http://127.0.0.1:5000/api/users
    HTTP/1.0 201 CREATED
    Content-Type: application/json
    Content-Length: 27
    Location: http://127.0.0.1:5000/api/users/1
    Server: Werkzeug/0.9.4 Python/2.7.3
    Date: Thu, 28 Nov 2013 19:56:39 GMT
    
    {
      "username": "test_user"
    }

Request an authentication token using user credentials:

    $ curl -u test_user:pwd -i -X GET http://127.0.0.1:5000/api/token
    HTTP/1.0 200 OK
    Content-Type: application/json
    Content-Length: 139
    Server: Werkzeug/0.9.4 Python/2.7.3
    Date: Thu, 28 Nov 2013 20:04:15 GMT
    
    {
      "duration": 600,
      "token": "eyJhbGciOiJIUzI1NiIsImV4cCI6MTM4NTY2OTY1NSwiaWF0IjoxMzg1NjY5MDU1fQ.eyJpZCI6MX0.XbOEFJkhjHJ5uRINh2JA1BPzXjSohKYDRT472wGOvjc"
    }

Save token (replace token parameter with token) to file:
    $ echo 'Authorization:Token <token>' > token.txt


These credentials can now be used to access protected resources:

    $ curl -i -H "$(< token.txt)" -XGET http://127.0.0.1:5000/api/resource
    HTTP/1.0 200 OK
    Content-Type: application/json
    Content-Length: 30
    Server: Werkzeug/0.9.4 Python/2.7.3
    Date: Thu, 28 Nov 2013 20:02:25 GMT
    
    {
      "data": "Hello, test_user!"
    }

To upload file:
curl -i -H "$(< token.txt)" -XPOST -F file=@<relative path to file> http://127.0.0.1:5000/

To download file:
    $ curl -i -H "$(< token.txt)" -GET http://127.0.0.1:5000/uploads/<filename>
    HTTP/1.0 200 OK
    Content-Type: application/json
    Content-Length: 30
    Server: Werkzeug/0.9.4 Python/2.7.3
    Date: Thu, 28 Nov 2013 20:05:08 GMT
    
    {
      "data": "Hello, test_user!"
    }

Once the token expires it cannot be used anymore and the client needs to request a new one. Each file uploaded is also associated with the uploading user and cannot be accessed by any other.
