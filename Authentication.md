If you want to restrict the users that can use the terminal, you can use authentication. The way jQuery Terminal implements authentication is using a token. The token is a way to validate if the client and server are the same person. The token is any true value, but ideally it should be a unique identifier that can be validated on the server. Example result of any hash function like md5 (I think you don't need to worry about collisions in the token, but you can use a more secured hash if you want) on current time saved in a safe place, like the database or a session.

The simplest way of authentication is `login` option. If you have JSON-RPC interpreter you can set this option to `true` or stings and it will call login RPC method or method specified by the option and if RPC return truthy value, the best is token hash as string, it
will log you in the the terminal. And from now own each call to RPC method will have token as first argument so each method on the server should check if token is valid.

Examples
```javascript
$('body').terminal('service.php', {
  login: true
});
```

```javascript
$('body').terminal('service.php', {
  login: "auth"
});
```


and JSON-RPC service requires each procedure to have a token as the first argument and validate it. It should throw an error if the token is invalid. When this happens the command will show an error on the terminal. You can use any implementation of JSON-RPC in your server-side language of choice.

You can also specify a function as login, where you can use naive check for the password in JavaScript or call the server to verify the user (you don't need to use JSON-RPC but it requires more code), which is a proper way to do.

```javascript
$(function() {
    $('#terminal').terminal({}, {
        login: function(user, password) {
            return fetch('auth.py', {
                method: 'POST',
                body: JSON.stringify({user, password}),
                headers:{
                    'Content-Type': 'application/json'
                }
            }).then(res => res.json());
        }
    });
});
```
and `auth.py` should read JSON as POST data validate that this is valid user and if so generate a token (any value) and save it in safe place (that can't be accessed from the web).

Here we use fetch API to send POST request to the server in `auth.py` URI and whatever JSON it returns it will be used as a token, the server needs to return JSON string (texts `"<generated token>"`) or null (false will also work). `auth.py` should read JSON as POST data validate that this is a valid user and if so generate a token (any value) and save it in safe place (that can't be accessed from the web).

If you want to have a prompt like in one of the previous sections (different prompt for login in users and guests) then you can't use login option because this will not
allow using the terminal without login. Instead, you need to create a command that will call login method.

```javascript
$(function() {
    $('#terminal').terminal({
        hello: function() {
            // client side only validation if token exists
            // to protect the app you should validate
            // the token on the server
            if (this.get_token()) {
               this.echo('Welcome');
            } else {
               this.error("You need to login first");
            }
        },
        login: function() {
            if (this.get_token()) {
                this.error("you're already logged in");
            } else {
                this.login(function(user, password, callback) {
                    // this is just example in real applications check need to be on the server
                    // and token should be unique value generated on the server that you can
                    // be checked if it's valid by server
                    if (user === 'john' && password === 'connor') {
                        callback("SKYNET");
                    } else {
                        callback(null);
                    }
                });
            }
        }
    }, {
        prompt: function(callback) {
            if (this.get_token()) {
                callback(this.login_name() + '@server$ ');
            } else {
                callback('guest@server$ ');
            }
        }
    });
});
```

Here is alterative version using nested interpreter (`push` method) that you can exit from (with `exit` command or with `CTRL+D`):

```javascript
  $('body').terminal(function(command, term) {
    var cmd = $.terminal.parse_command(command);
    if (cmd.name === 'login') {
      term.push({
        hello: function() {
          this.echo('hello there');
        }
      }, {
        prompt: 'obi> ',
        login: function(username, password) {
          if (username === 'obi' && password === 'one') {
            return Promise.resolve('TOKEN');
          }
          return Promise.reject();
        }
      });
    } else {
      this.error('Invalid Command.');
    }
  });
```

login method works the same as the login option. The callback is an old API that was not removed to not introduce breaking changes.

> In every API that have callback you can return a promise.

## Custom Login Function

Bult-in login function test both username and password and give a single error that user or password is invalid.
You can create your own login function where you first check if the user exists and if not show an error about a wrong user.
Then you can check the user and password is valid and if not you can only show an error about a wrong password.

```javascript
var users = [
    {
        user: 'jon',
        password: 'secret'
    },
    {
        user: 'jane',
        password: 'monty'
    }
];

function valid_user(username) {
    return users.find(user => user.user == username);
}
function valid_user_password(username, password) {
    var user = users.find(user => user.user == username);
    return user && user.password === password;
}

function login(term, interpreter, options) {
    var history = term.history();
    history.disable();
    term.push(function(user) {
        if (!valid_user(user)) {
            term.pop().error('Invalid Username');
        }
        term.set_mask(true).push(function(pass) {
            term.set_mask(false).pop().pop();
            if (!valid_user_password(user, pass)) {
                term.error('Invalid Password');
            } else {
                term.push(interpreter, $.extend(options, { prompt: user + '@~ ' }));
            }
        }, { prompt: 'password: ' });
    }, { prompt: 'user: ' });
}
```

> Note that if you want your application to be secured you should never save user and password in JS. This is just an example to simplify the code. In a real application user and password should be saved on the server and you should use AJAX to send a request to the server that will check if the user is valid and then if the user/password is valid.

This code doesn't have any token or login name that is used by the default in jQuery Terminal. So if you want to use token-based authentication you can get the token from the server and save it with `term::set_token`. and use `term::get_token` to see if the token is set and validate it on the server before executing the commands.