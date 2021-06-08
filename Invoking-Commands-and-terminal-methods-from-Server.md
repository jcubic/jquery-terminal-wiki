The echo command is quite powerful. With it, you can invoke the different commands from the server. The mechanism is called **extended commands**.

You can call code like this:

```javascript
var term = $('body').terminal({
    foo: function(a, b, c) {
        console.log({a, b, c});
    }
});
term.echo('[[ foo 1 2 /bar/ ]]');
```

and foo command will be executed where `a` is number 1, `b` is number 2 and `c` is a regular expression.

this is the same as `term.exec('foo 1 2 /bar/');`, but the difference is that you can return the string `"[[ foo 1 2 3 ]]"` from JSON-RPC procedure. For this to work you'll need to create an interpreter that will have both JSON-RPC and object (or function).

```javascript
var term = $('body').terminal(["service.php", {
    foo: function(a, b, c) {
        console.log({a, b, c});
    }
}]);
```

This will create an interpreter with both JSON-RPC service in PHP file and custom object.

With this setup, you can create a procedure `"hello"` in JSON-RPC service that will return `"[[ foo 1 2 3 ]]"` and when you type "hello" it will run `console.log({a, b, c});`.

You can also return `"[[ exit ]]"` from JSON-RPC and when that procedure will be executed (you type the proper command) it will log out from the interpreter if you have a login or call pop if you're in the nested interpreter. There is also one feature that's disabled by default for security reasons (see 
[security section in API reference](https://terminal.jcubic.pl/api_reference.php#security)). To execute terminal method, like for instance shortcut CTRL+R from the server, you
can use this syntax:

```
[[ terminal::invoke_key("CTRL+R") ]]
```
You can execute any method on the jQuery Terminal instance. The same goes to cmd instance methods, you can insert text and change cursor position to the beginning of the line, from server with this string:

`[[ cmd::set("hello") ]][[ cmd::position(0) ]]`

To enable this feature you need to set `invokeMethods: true` option.

The only limitations are that you can't return value from this method, you can't return an object and invoke a method on it (so you can't disable history but this may change). The other limitation is that arguments need to be valid JSON so this will not work:

`[[ cmd::set('hello') ]]`

## Custom controller

With this feature you can write any missing function that will control the terminal and if you JSON-RPC service you can give your server a way to control the terminal in any way. Adding powerful behavior.

Example:

```javascript
$('body').terminal([
  "service.py", 
  {
    size: function(x) {
      this.css('--size', x); // custom properties works in jQuery 3.2
    }
  }
});
```

In the above example terminal create two interpreters two objects one from JSON-RPC service and one object with one command `size` (more about interpreters in [[Advanced jQuery Terminal Tutorial]]). With this setup, the server can return the string `[[ size 1.5 ]]` and change the size of the terminal. You can write any method you like and invoke it from the server.

### Animation from server

Having this feature at your disposal you can make the server do whatever you want. Just another example is an animation that is sent from the server.

```javascript
$('body').terminal([
  "service.py", 
  {
    typing: function(message, delay, continuation = null) {
      this.typing('echo', message, delay).then(() => {
        if (continuation !== null) {
          this.exec(continuation, true);
        }
      });
    }
  }
], { checkArity: false });
```

with this code, the server (JSON-RPC) can return the string `"[[ typing 'hello' 200 ]]"` to create typing animation of "hello" and you can also you can sue the last argument to command to execute the next command. For instance, you can add the command `"ask"` that you will need to write next to typing that will ask the user for his name and save it in localStorage for the retrival. To do that just return `"[[ typing 'hello' 200 "ask" ]]"` from the server.