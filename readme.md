## koa-scheme

koa-scheme is a parameter validation middleware for koa.

### Install

    npm i koa-scheme --save
    
### Usage

    scheme(config, options)

- config: {Object} scheme object.
- options: {Object}
  - debug: {Boolean} whether to print compiled `config`.

**app.js**

```
var koa = require('koa');
//var bodyParser = require('koa-bodyparser');
var scheme = require('koa-scheme');
var conf = require('./scheme');
var route = require('./route');

var app = koa();
//app.use(bodyParser());
app.use(scheme(conf));

route(app);

app.listen(3000, function() {
  console.log("listening on 3000")
});
```

**scheme.json**

```
{
  "/(.*)": {
    "request": {
      "header": {
        "version": "[1-9]+"        
      }
    }
  },
  "/": {
    "response": {
      "status": 200
    }
  },
  "GET /user/:username": {
    "response": {
      "body": {
        "name": /[a-z]+/i,
        "age": "[0-9]{1,3}"
      }
    }
  },
  "/user/:username": {
    "request": {
      "method": "(POST|patch)",
      "body": {
        "name": "[a-zA-Z]+",
        "age": /[0-9]{1,3}/
      }
    },
    "response": {
      "status": 200
    }
  },
  "(delete|OPTIONS) /user/:username": {
    ...
  }
}
```

see [path-to-regexp](https://github.com/pillarjs/path-to-regexp).

**NB**: when use body-parser middleware (like: koa-bodyparser) before koa-scheme, you could configure 'body' field in 'request'.

use function:

**scheme.js**

```
module.exports = {
  "/users": {
    "request": {
      "method": "POST",
      "body": {
        "nameArr": testRequestNameArr
      }
    }
  }
}

function testRequestNameArr(arr) {
  if (arr.length === 3 && arr[1] === 'nswbmw') {
    return true;
  } else {
    return false;
  }
}
```

with [validator](https://github.com/chriso/validator.js):

**scheme.js**

```
var validator = require('validator');

module.exports = {
  "GET /user/:username": {
    "response": {
      "body": {
        "age": validator.isNumeric,
        "email": validator.isEmail,
        "webset": validator.isURL,
        ...
      }
    }
  }
}
```

Even you could write flat object like this:

```
{
  "GET /user/:username": {
    "response": {
      "body.age": validator.isNumeric,
      "body.email": validator.isEmail,
      "body.family.mother.age": validator.isNumeric
    }
  }
}
```

### Example

**scheme.js**

```
var validator = require('validator');

module.exports = {
  "/(.*)": {
    "request": {
      "header.version": "[1-9]+"
    }
  },
  "/users": {
    "request": {
      "method": "GET"
    },
    "response": {
      "body": testRes
    }
  },
  "GET /user/:username": {
    "response": {
      "body": {
        "name": "[a-zA-Z]+",
        "age": validator.isNumeric,
        "email": validator.isEmail
      }
    }
  },
  "/user/:username": {
    "request": {
      "method": "(POST|PATCH)",
      "body.name": /[a-zA-Z]+/,
      "body.age": "[0-9]{1,3}",
      "body.email": validator.isEmail,
      "password": checkPassword,
      "re_password": checkPassword
    }
  },
  "(delete|OPTIONS) /user/:username": {
    "response": {
      "status": 200
    }
  }
}

function testRes(arr) {
  if (arr && Array.isArray(arr) && arr.some(function (user) {return user.name === 'nswbmw'})) {
    return true;
  } else {
    return false;
  }
}

function checkPassword(password) {
  return function(re_password) {
    return password === re_password;
  }
}

```

### Test

   npm test

### License

MIT