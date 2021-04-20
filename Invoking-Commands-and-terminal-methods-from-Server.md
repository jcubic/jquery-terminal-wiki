The echo command is quite powerful. With it, you can invoke the different commands from the server.

In JSON-RPC you can return the string `"[[ exit ]]"` and it will log out from the interpreter if you have a login or call pop if you're in the nested interpreter. There is also one feature that's disabled by default for security reasons (see 
[security section in API reference](https://terminal.jcubic.pl/api_reference.php#security)). To execute terminal method, like for instance shortcut CTRL+R from the server, you
can use this syntax:

```
[[ terminal::invoke_key("CTRL+R") ]]
```

the same goes to cmd instance methods, you can insert text and change cursor position to the beginning of the line, from server with this string:

`[[ cmd::set("hello") ]][[ cmd::position(0) ]]`

To enable this feature you need to set `invokeMethods: true` option.

The only limitations are that you can't return value from this method, you can't return object and invoke a method on it (so you can't disable history but this may change). The other limitation is that arguments need to be valid JSON so this will not work:

`[[ cmd::set('hello') ]]`

## Custom controller

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
