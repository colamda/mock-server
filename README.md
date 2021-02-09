# Colamda mock-server

The mock-server is able to serve the contents of two distinct directories:

- `static`: contains static files and sub-folders that are returned directly and are separated by
  http request methods (`GET`, `POST`, ...)
- `dynamic`: contains java script files that are executed

File in the `static` directory MAY have a `.json` extension.
Files in the `dynamic` directory have a `.js` extensions.

Considering the following directory structure:

```
/static/
  GET/
    files/
      test.jpg
      README
    users/
      index.json
      234.json
  POST/
    users/
      234.json
/dynamic/
  users/
    567.js
  groups/
    index.js
  misc/
    readme.txt.js
```

- `GET /users/` will return the `/static/users/index.json` file
- `GET /files/test.jpg` will return the `/static/GET/files/test.jpg` file
- `GET /files/README` will return the `/static/GET/files/README` file
- `GET /users/234` will return the `/static/GET/users/234.json` file
- `POST /users/234` will return the `/static/POST/users/234.json` file
- `POST /users/567` will execute the `/dynamic/users/567.js` script
- `GET /users/567` will execute the `/dynamic/users/567.js` script
- `DELETE /groups` will execute the `/dynamic/groups/index.js` script
- `GET /misc/readme.txt` will execute the `/dynamic/misc/readme.txt.js` script

## Example scripts

Scripts must export a `handle` function that takes two parameters, a request and a response parameter.

```
exports.handle = function (req, res) {
  var password = req.body;

  if (password !== 'secret') {
    res.status(403).send('wrong password');
  } else {
    res.send({result: 'ok'});
  }
};
```

## Install and setup

```
yarn add @colamda/mock-server
```

In `package.json` add e.g.

```
"scripts": {
  "mock:serve": "mock-server --dynamic=mocks/dynamic --static=mocks/static --port=3000"
}
```

The mock server accepts the following parameters:

- `--dynamic=`: relative path to directory with dynamic scripts
- `--static=`: relative path to directory with static files
- `--port=`: server port (default is `3000`)

### Watch & Reload

If a script is changed (and was already used by the server), the server must be restarted.
This can be automated with `nodemon`.

First install `nodemon`:

```
yarn add nodemon
```

Then change the start script in `package.json` to e.g.:

```
"mock:serve": "nodemon --watch mocks/dynamic --exec mock-server --dynamic=mocks/dynamic --static=mocks/static --port=3000"
```

## Use cases

### Return a different http code than 200

Create a .js script and specify the response code.
