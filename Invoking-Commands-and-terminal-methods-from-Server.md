echo command is quite powerful with it you can execute different method and because of this you can invoke different command from server.

In JSON-RPC you can return string "[[ exit ]]" and it will logout from the interpreter if you have login or call pop if you're in nested interpreter. There is also one feature that's disabled by default for security reasons (see 
[security section in API reference](https://terminal.jcubic.pl/api_reference.php#security)). To execute terminal method, like for instance shortcut CTRL+R from server, you
can use this syntax:

```
[[ terminal::invoke_key("CTRL+R") ]]
```

the same goes to cmd instance methods, you can insert text and change cursor position to the beginning of the line, from server with this string:

`[[ cmd::set("hello") ]][[ cmd::position(0) ]]`

To enable this feature you need to set `invokeMethods: true` option.

The only limitations is that you can't return value from this method, you can't return object and invoke method on it (so you can't disable history but this may change). The other limitation is that arguments need to be valid JSON so this will not work:

`[[ cmd::set('hello') ]]`