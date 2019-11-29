You can parse and process commands with options like in the *linux* command line or any other shell.
If you're using a function as interpreter, you can parse the commands to split the command into arguments.

There are two pairs of functions:

* split_arguments
* parse_arguments

and

* split_command
* parse_command

x_command will create an object from a parsed string

```
{name, args, rest, command, args_quotes}
```

* name - text before the first space
* args - the array of arguments parsed with x_arguments
* rest - the rest of the text without name and space as is
* command - the input string as is
* args_quotes - array of strings with quote used ' or "

x_arguments functions will create an array from the arguments splitted on space. x_command functions use x_arguments functions to process the arguments array.

The processing can handle spaces inside regular expressions and strings.

Another function was added in version *1.17.0*: `$.terminal.parse_options`. It parses the arguments and accepts two parameters either a string or an array and an object with options. Right now, there is only one option available which is `boolean`. It should be an array of strings, to indicate options that don't have argument.

`parse_options` is very simple and is very similar to [yargs parser](https://www.npmjs.com/package/yargs-parse). It returns an object with a field `_`as an array for the "free" options and each said "free" options is saved as a key with value set to `true`. The other options appear as key with their value as string:

```javascript
$.terminal.parse_options(["-x", "foo", "-aby", "bar"], {boolean: ['y']});
```

this will return

```
{_: ["bar"], x: "foo", y: true, a: true, b: true}
```

If you're using an object as interpreter, you can use the following **ES5** syntax to get an `options` array:
```javascript
$(function() {
    $('#terminal').terminal({
        fetch: function() {
            var args = Array.from(arguments);
            var options = $.terminal.parse_options(args);
            if (!options.url) {
                this.error("You need to specify the url");
            }
            if (options.method == 'POST') {
                 // do POST request
            } else {
                 // do GET request
            }
        }
    }, {checkArity: false});
});

Or the **ES6** version, if you like:
```

```javascript
$(function() {
    $('#terminal').terminal({
        fetch: function(...args) {
            var options = $.terminal.parse_options(args);
            if (!options.url) {
                this.error("You need to specify the url");
            }
            if (options.method == 'POST') {
                 // do POST request
            } else {
                 // do GET request
            }
        }
    }, {checkArity: false});
});
```

and you will get these responses for  these commands:

```
fetch --method "POST" - will show error that the URL is not specified
fetch --url https://example.com/file.txt --method POST - will fetch the file using POST
fetch --url https://example.com/file.txt - will fetch the file using GET method
```

Another usage example of this function can be found in the [API reference](https://terminal.jcubic.pl/api_reference.php#parse_options).