If you want to restrict the users that can use terminal you can use authentication, the simplest way is login option. If you have JSON-RPC interpreter you can set this
option to true or stings and it will call login RPC method or method specified by the option and if RPC return truthy value, the best is token hash as string, it
will log you in the the terminal. And from now own each call to RPC method will have token as first argument so each method on the server should check if token is valid.

You can also specify function as login, where you can use naive check for password in JavaScript or call the server to verify the user, which is proper way to do.

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

Here we use fetch API to send POST request to the server in auth.py URI and whatever JSON it return it will be used as token, the server need to return text "<generated token>" or null (false will also work).

If you want to have prompt like in one of the previous sections (different prompt for login in users and guests) then you can't use login option because this will not
allow to use the terminal without login. Instead you need to create a command that will call login method.

```javascript
$(function() {
    $('#terminal').terminal({
        login: function() {
            if (this.get_token()) {
                this.error("you're already logged in");
            } else {
                this.login(function(user, password, callback) {
                    // this is just example in real applications check need to be on the server
                    // and token should be uniq generated value that you can
                    // check if it's valid on the server
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

login method work the same as login option. Callback is old API that was not removed to not introduce breaking changes.