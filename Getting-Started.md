### Table of Contnent
1. [Introduction](#user-content-introduction)
2. [Unpkg CDN](#user-content-unpkg-cdn)
3. [Html page](#user-content-html-page)
4. [Initialization](#user-content-initialization)
5. [Accessing Terminal Object](#user-content-accessing-terminal-object)
6. [Creating interpreter](#user-content-creating-interpreter)
7. [What you can echo?](#user-content-what-you-can-echo)
8. [Greetings](#user-content-greetings)
9. [Parsing commands](#user-content-parsing-commands)
10. [Formatting](#user-content-formatting)
11. [Syntax highlighting](#user-content-syntax-highlighting)
12. [Less command](#user-content-less-command)
13. [ANSI Escape codes](#user-content-ansi-escape-codes)
14. [Custom Syntax highlighting](#user-content-custom-syntax-highlighting)
15. [Tab completion](#user-content-tab-completion)
16. [Command line history](#user-content-command-line-history)
17. [Reverse history search](#user-content-reverse-history-search)
18. [Chinese and Japanese character support](#user-content-chinese-and-japanese-character-support)
19. [Prompt](#user-content-prompt)
20. [Reading text from user](#user-content-reading-text-from-user)
21. [Masking password](#user-content-masking-password)
22. [Authentication](#user-content-authentication)
23. [Key Shortcuts](#user-content-key-shortcuts)
24. [Events](#user-content-events)
25. [Asynchronous Commands](#user-content-asynchronous-commands)
26. [Saving State](#user-content-saving-state)
27. [Executing commands from JavaScript](#user-content-executing-commands-from-javascript)
28. [Invoking Commands and terminal methods from Server](#user-content-invoking-commands-and-terminal-methods-from-server)
29. [Updating lines](#user-content-updating-lines)

### Introduction

jQuery Terminal is JavaScript library that require, to work properly, other library which is jQuery. To create app using jQuery Terminal, first you need to create
html page, then include jQuery and both JavaScript and CSS file from jQuery Terminal. You can also use webpack or other bundler but we will not discuss this here.
I this tutorial you will know all the basics of creating your own Terminal looking webpage as well as explanations of all the features.

### Unpkg CDN

The best way to get jQuery Terminal is from [unpkg.com](https://unpkg.com/#/) that serve files from [npm repository](https://npmjs.com). With it you don't need to host the files yourself. It's useful if you want to create quick Proof of Concept or if you want to use
one of demo services like Codepen, JSFiddle.
With unpkg.com CDN you can specify that you always want latest version but not if they have breaking changes. For instance if latest version is 1.22.1, you can specify that you want version 1.22 and all the bug fixes using url:

```
https://unpkg.com/jquery.terminal@1.22.x
```

this url will redirect to file /js/jquery.terminal.js with proper version number. If you want to have all the bug fixes to version 1 and all the features you can use 1.x.x instead of 1.22.x. And you if you want minified version of the library you can specify this url:

```
https://unpkg.com/jquery.terminal@1.x.x/js/jquery.terminal.min.js
```

same for css file, you can use url:

```
https://unpkg.com/jquery.terminal@1.x.x/css/jquery.terminal.min.css
```

### Html page

Basic html page using unpkg look like this

```html
<!DOCTYPE html>
<html>
<head>
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<script src="https://unpkg.com/jquery.terminal@1.x.x/js/jquery.terminal.min.js"></script>
<link href="https://unpkg.com/jquery.terminal@1.x.x/css/jquery.terminal.min.css"/>
</head>
<body>
</body>
</html>
```

Of course you can get the files using download option, install it using npm or bower or even by cloning the git repository.

### Initialization

To initialize the plugin and create your terminal you need to invoke the plugin. To do that you need to select some element of the page, use CSS selector with jQuery.
to Select element with specific ID you can use `$('#terminal')` or if you want to attach terminal to body you can use `$('body')` if you have your script using

```html
<script>
/* your code */
</script>
```

or

```html
<script src="script.js"></script>
```

inside head then you need to wrap your code inside jQuery Ready function that will be invoked when DOM (Document Object Model - html representation in memory) will be
ready so jQuery will find `#terminal` or `body`. to do that you need this code

```javascript
$(function() {
   $('#terminal');
});
```

if you have your code after the body (or before closing `</body>`) you don't need to wrap it with `$(function() {});`

when you select your jQuery object you need to invoke terminal plugin:

```javascript
$(function() {
   $('#terminal').terminal();
});
```

### Accessing Terminal Object

Terminal object have useful methods that you can invoke, whole list of methods in API documentation. To access this object, which is jQuery object with more methods, you can use:

* value returned by the plugin:

```javascript
var term = $('#terminal').terminal(function(command) {
    
});
term.echo('foo');
```

* you can use this in every function (if you can't you can report a bug).
```javascript
$('#terminal').terminal(function(command) {
    this.echo('foo');
});
```

* the third options is that some functions have it as last parameter (it should be in all of them), which is legacy to not break the API.

```javascript
$('#terminal').terminal(function(command, term) {
    term.echo('foo');
});
```

this is useful for example when you use ES5 and `setTimeout` and want to call echo after delay (with ES6 you can use arrow function).

### Creating interpreter

You know now how to invoke the plugin now you need to specify the interpreter (first plugin argument) and options, which are optional, but you probably want at least to remove default greetings, which discussed later in this document.

There are 3 types of interpreters: function, object and strings.

* function - is created when you want whole command as is passed to you

```javascript
$(function() {
   $('#terminal').terminal(function(command) {
    
   });
});
```

* object

```javascript
$(function() {
   $('#terminal').terminal({
      open: function(value) {
         
      },
      sub: {
         get: function(arg) {
         }
      }
      close: function(arg1, arg2) {
      }
   });
});
```

this will create terminal with three commands: open, close and sub. If you type `open something` the open function will be invoked and value parameter will equal to something.
Terminal will parse all your commands, convert number like strings into numbers and Regular Expressions into RegExp objects. You can disable this behavior using
[processArguments](https://terminal.jcubic.pl/api_reference.php#processarguments) option. This option can be boolean and you can set it to false or it can be a function where you
can process the value in your own way (in this function for instance you can use JSON.parse to get objects out of strings). By default if you have object like this and you type `close foo` terminal will throw error because you didn't specify second argument.
To prevent this behavior you can use option [checkArity](https://terminal.jcubic.pl/api_reference.php#checkarity) set to false.

In this example you have sub which don't have a function but another object, when you type `sub` in terminal the prompt will change to `sub> ` and you will be able to type
get, open and close will not work. This is called nested interpreter, which can also be created when you have function as interpreter using [push method](https://terminal.jcubic.pl/api_reference.php#push).

*  third option is string which point to [JSON-RPC service](https://en.wikipedia.org/wiki/JSON-RPC), if service is on the same domain you can use it as is, but if it's on different domain or port then you need
to use mechanism called [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing).

```javascript
$(function() {
   $('#terminal').terminal("service.py");
});
```

* There is also one more type of interpreter which is array that can combine all the above types but with limitations, you can have only one function that is executed when no other command is found. You can also have single JSON-RPC service without system.describe (or if you turn it off using [describe option](https://terminal.jcubic.pl/api_reference.php#describe)) which was part of the historical 1.1 version of JSON-RPC specification. It's special method that should
return different object then normal JSON-RPC, the spec for the object is in [system.describe section in API reference page](https://terminal.jcubic.pl/api_reference.php#system.describe). JSON-RPC is converted to simple function while JSON-RPC with system.describe is converted to object.

If you want to have automatic completion for JSON-RPC and only want normal JSON-RPC methods you can specify option [describe](https://terminal.jcubic.pl/api_reference.php#describe) as string with dot separated values that point to property procs from the spec, which should be array of object with name and params. (`describe: "result"` will work if proc array is normal JSON-RPC response).

Here is how the array should look like:

```javascript
[
   {"name":"echo","params":["string"]}
]
```

Here how it looks to invoke terminal with array:

```javascript
$(function() {
   $('#terminal').terminal([
      {
         open: function() {
         },
      },
      "service.py",
      function(command) {
          this.echo("You've typed " + command);
      }
   ]);
});
```

### What you can echo?

echo method on terminal instance if versatile, you can call echo with following:
* string
* array of strings
* any other value that can have toString method (so also numbers or boolean that are boxed in Number object or any custom object that have toString method).
* Promise that resolve to any of the above

Limitations you can't call echo with array of other values only array of strings and JavaScript objects are printed as `[object Object]` (Because this what's returned by toString method), but this may change and new printing objects may be introduced in the future.

### Greetings

Second argument to terminal plugin is object where you put your options. Greetings is one of them, is the first text that is echo into terminal, you also can use
term.echo to have the same effect but to have the same behavior as with greetings you will
need to put it in `onInit` event, because greetings is showing after login. More on login and `onInit` in later sections.

```javascript
$(function() {
   $('#terminal').terminal("service.py", {
      greetings: "Welcome to Python Service"
   });
});
```


NOTE: if you want to create ASCII logo as greetings, you will need to escape special characters. If you create string with double quotes "" you need to escape double quote `"` and slash `\` with `\"` and `\\`. The same with strings with ``` ` ``` and `'`, you will need to escape those with slashes.

### Parsing commands

To process command with options like in linux command line you can parse it and process it.
If you're using function as interpreter you can parse the commands to split the command into arguments.

There are two pairs of functions:

* split_arguments
* parse_arguments

and

* split_command
* parse_command

x_command will create object from string

```
{name, args, rest, command, args_quotes}
```

* name - text before first space
* args - array of arguments
* rest - text without name and space as is
* command - the same as input string.
* args_quotes - array of strings with quote used ' or "

x_arguments functions will create array from arguments split on space. x_command functions use x_arguments functions to get arguments array.

The processing handle spaces inside regular expressions and strings.

Another functions added in version 1.17.0 is $.terminal.parse_options parse arguments if accept two arguments string or array and object with options. Right now there is only one option which is boolean which should be array of strings, that indicate that option don't have argument.

parse_options is very simple and works similar to [yargs parser](https://www.npmjs.com/package/yargs-parse). It returns object
with field `_` that is array of free options and each option is saved as key with value be true or string:

```javascript
$.terminal.parse_options(["-x", "foo", "-aby", "bar"], {boolean: ['y']});
```

this will return

```
{_: ["bar"], x: "foo", y: true, a: true, b: true}
```

If you're using object as interpreter you can use ES6 syntax to get option array:

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

and if you will get this respons on this commands

```
fetch --method "POST" - will show error that url is not specified
fetch --url https://example.com/file.txt --method POST - will fetch the file using POST
fetch --url https://example.com/file.txt - will fetch the file using GET method
```

Another usage example of this function can be found in [API reference](https://terminal.jcubic.pl/api_reference.php#parse_options).

### Formatting

Formatting is special syntax that can be used to format your text, you can make your text bold italic or underline or even glow with this syntax.

```javascript
term.echo('[[b;red;white]hello world]');
```

this will create bold text with red color and white background more about the <a href="api_reference.php#echo">syntax of Terminal formatting in API reference</a>.

If you need to echo brackets, best way is to use $.terminal.escape_brackets function.

### Syntax highlighting

If you want to define syntax highlighting for instance taken from Python or SQL, there is option to not have to specify this formatting on your own but use [PrismJS library](https://prismjs.com/). This will also work while you type. To use Prism you first need to include files needed for the library:

```
https://unpkg.com/prismjs/prism.js
https://unpkg.com/prismjs/themes/prism.css
```

by default PrismJS only include html css and JavaScript syntax if you need python or SQL you need to include one of the [component files](https://unpkg.com/prismjs/components/) they are also on npm and on [GitHub](https://github.com/PrismJS/prism).

After you include PrismJS files you need to include one more thing which is jQuery Terminal Prism wrapper:

```
https://unpkg.com/jquery.terminal@1.x.x/js/prism.js
```

Then you can call this function somewhere after terminal files and prism. it don't need to be in $(function() {});

```javascript
$.terminal.syntax('python')
```

to have python highlighting. There is on additional syntax add by Terminal prism wrapper which is website that highlight html, css and javascript.
Where css need to be in style tag and JS in script tag.

Syntax highlighters are also useful if you want to have command like cat or less where

### Less command

If you want to have less like command you can use additional file:

```
https://unpkg.com/jquery.terminal@1.x.x/js/less.js
```

that add new jQuery plugin that can be invoked on Terminal instance which is also jQuery Object.

```javascript
$('body').terminal({
    less: function(file) {
       $.get(file, (text) => this.less(text));
    }
});
```

this will invoke less with any syntax highlighting you have, if you don't want to have formatting only for less, then you should not use function
syntax but use function $.terminal.prism:

```javascript
$('body').terminal({
    less: function(file) {
       var language = {html: 'website': js: 'javascript', css: 'css'};
       var ext = file.match(/\.([^.]+)$/)[1];
       $.get(file, (text) => this.less($.terminal.prism(language[ext], text)));
    }
});
```

### ANSI Escape codes

ANSI escape codes is a way to format text in unix terminal. The formatting look like this:

```
\x1b[32mHello\x1b[m
```

This will display text Hello in gren. 1B is hex for 27 which is escape key.

Explanation of the [ANSI ESCAPE codes can be found on Wikipedia](https://en.wikipedia.org/wiki/ANSI_escape_code).
To use ANSI formatting in jQuery Terminal you only need to include one file:

```
https://unpkg.com/jquery.terminal@1.x.x/js/unix_formatting.js
```

The file also hadle what's called overtyping (which is output from man command on Linux) where you have text like this
```
A\x08AB\x08BC\x08C
```

which will display text ABC in bold text, which in Terminal will be also white (like in Linux). `\x08` is backspace key so you simple write two times
the same character to make it bold, if you use `A\x08_` you will get underline. Backspaces (`\x08` characters) should work the same
as in Linux terminal.

### Custom Syntax highlighting

You can also create your own formatters (this is how terminal's prism.js work). Low level mechanism for formatters consist of array:

```javascript
$.terminal.defaults.formatters
```

which can be array of functions and arrays with regular expression and string. In function you can do any text replacement and return string with terminal formatting.
To add new formatter you can push value to this array, overwrite it with new array or use helper function `$.terminal.new_formatter` which will inject your formatter,
before nested formatting formatter that's default formatter, it allows to have nested formatting like in html `[[;red;]foo [[;blue;]bar] baz]`,
If you us new_formatter function your formatter also can have nesting.

For instance if you want text hello from user to be red and text world to be green you an use this formatter:

```javascript
$.terminal.new_formatter(function(string) {
    return string.replace(/hello/g, '[[;red;]hello]').replace(/world/g, '[[;green;]world]');
});
```

You can also put two arrays:

```javascript
$.terminal.new_formatter([/hello/g, '[[;red;]hello]']);
$.terminal.new_formatter([/world/g, '[[;green;]world]']);
```

You should use Regular expression with g to replace all instances you if you string or regex without g it will replace only first instance of the string.

Because formatters are in `$.terminal` namespace they are global for whole page, so you can't have two terminals on the page and each have different formatters (yes you can have multiple terminals on on page).
This may change in the future but it will be breaking change so it will be in next major version, so if you're using 1.x.x version, from unpkg.com, you will not be affected.

NOTE: if you need to create formatter that change length of the string like replace foo with hello world then you need to use special function ([$.terminal.tracking_replace](https://terminal.jcubic.pl/api_reference.php#tracking_replace)) that will track
cursor position after replacement, which was mostly created by [Stack Overflow user T.J. Crowder](https://stackoverflow.com/a/46756077/387194).
The cursor movement will be little weird because the cursor will be at the beginning of the result string or after while you move virtual cursor on original string.
So you will be able to remove any character in input string.

To have correct cursor position, with above limitations, You need to use function like this:

```javascript
$.terminal.new_formatter(function(string, position) {
    return $.terminal.tracking_replace(string, /hello/g, '[[;red;]hello world]', position);
});
```

this will be handled for you if you use simple array. But this is the only option if you need to put some kind of logic in replacement.

This will be the same as above:

```javascript
$.terminal.new_formatter([/hello/g, '[[;red;]hello world]']);
```

Note: one more limitations of formatters is that they are executed on strings between formatting so above formatter will not create infinite loop of replacement.
With exception of formatter like `$.terminal.nested_formatting` which is always added to the list when including the library.

If you want to have formatters based on interpreters, like for instance you want mysql command that have sql syntax and js command to have javascript formatting etc. then you can create a stack of formatters (Stack is the data structure [more on Wikipedia](https://en.wikipedia.org/wiki/Stack_(abstract_data_type)). Example how to do that is in [example section on a web site](https://terminal.jcubic.pl/examples.php#syntax_highlight).

### Tab completion

Tab completion is another useful feature of jQuery Terminal by default if you press tab it will insert it into terminal (it will be replaced by 4 spaces but it will
be treated as one character). To change default behavior you need to use [completion option](https://terminal.jcubic.pl/api_reference.php#completion).

it you have object as first argument or JSON-RPC string with system.describe

```javascript
$('#terminal').terminal({
  open: function() {
  },
  close: function() {
  },
  'open-foo': function() {
  }
}, {
  completion: true
});
```

if you type op and press TAB it will insert open then if you press TAB twice it will list open and open-foo

Second option is if you have function as first argument, then you can use array:

```javascript
$('#terminal').terminal(function(command) {
    
}, {
  completion: ['pwd', 'pwdhash', 'pwdhash-bin', 'pwdx']
});
```

and if you need logic in completion for instance if you have command git and want to complete git clone and git checkout you can use this:

```javascript
$('#terminal').terminal(function(command) {
    
}, {
  completion: function(string, callback) {
      if (this.get_command().match(/^git /)) {
         callback(['checkout', 'clone', 'pull', 'push', 'commit']);
      } else {
        callback(['ls', 'cat', 'git']);
      }
  }
});
```

Additionally you can return a promise from completion, you can use Promise.resolve, but more useful is if you have your completion on the server:

```javascript
$('body').terminal(function(command) {
    
}, {
  completion: function(string, callback) {
      return fetch('completion.py', {string, command: this.get_command()}).then(res => res.json());
  }
});
```

### Command line history

Each time you press enter, your command is saved in history (you can disable this by using history option). History is saved in localStorage or in cookies. You can move through history commands using up and down arrows. You can filter which commands should go to history and which don't using historyFilter option. It can be regular expression that if match it will not add command to history (you can use regex /^\s/ to not put commands that start with space like in bash) you can also specify a function where you can use some logic and return true or false.
You can also limit number of commands in history using historySize option, if you set it to falsy value it will not restrict the size (but it may fill up your localStorage or cookies).

If want history to be present only in one session you can use history in memory, instead of localStorage or cookies, using memory option set to true, but this will make
all data saved in memory (the same will happen with all data require by terminal like authentication token or interpreter names).

### Reverse history search

The feature related to history is shortcut CTRL+R it's reverse search and it work the same like in bash, you can type text and it will scan your history to find proper
command. if you press enter you will exit search the current interpreter will be invoked. You will also exit if you use arrow keys, the command will be selected.
If you wan to cancel search you can press CTRL+G the same as in bash.

### Chinese and Japanese character support

With special <a href="https://github.com/timoxley/wcwidth">wcwidth</a> library you can have proper text wrapping on terminal with wider characters like those from
Chinese or Japanese. The package is on npm but it don't include the file proper for browsers (no UMD - Universal Module Definition), you can spam the author
on this <a href="https://github.com/timoxley/wcwidth/issues/2">issue on github</a>.

temporarily you can get it from rawgit.com, but the service will be shut down in 2019.

https://rawgit.com/jcubic/leash/master/lib/wcwidth.js

If the author don't create UMD module that will allow to use it in browser, I will probably publish some npm package to be able to use it from unpkg.com.

### Prompt

The prompt is the string in beginning of the command line, you can specify the prompt using:
* prompt option
* if you have nested interpreter created from nested object
* if you use push to create new interpreter
* if call term.set_prompt method

the prompt can be string (it can have formatting) or function with single callback as argument that should be called with new prompt.

```javascript
$(function() {
   $('#terminal').terminal("service.py", {
      prompt: "[[;red;]>>> ]"
   });
});

term.set_prompt(function(set_prompt) {
    if (this.get_token()) {
        set_prompt(this.login_name() + '@server$ ');
   } else {
        set_prompt('guest@server$ ');
   }
});
```

this will display different prompt to login in users and normal users, about authentication later.

### Reading text from user

If you're in interpreter function and want to ask user for input and then process it (like in C scanf) you can use function read:

```javascript
$('#terminal').terminal({
   name: function(name) {
      this.read('last name: ', last_name => {
         this.echo('Your name is ' + name + ' ' + last_name);
      });
   }
});
```

read function also return a promise so you can use this code instead:

```javascript
$('#terminal').terminal({
   name: function(name) {
      this.read('last name: ').then(last_name => this.echo('Your name is ' + name + ' ' + last_name));
   }
});
```

This will change the prompt to last name: as wait until user press enter, then it will resolve the promise, so you will see text on thee screen with
your name.

Unfortunately you can return the promise and in then return the string, because when you return a promise terminal will be paused, so you will not
see last name: prompt.

Other option to read text from user is to use nested interpreter using push where you can have logic and ask for more then one thing.
This is the code that will work the sam as read:

```javascript
$('#terminal').terminal({
    name: function(name) {
        this.push(function(last_name) {
            if (last_name) {
                this.echo('Your name is ' + name + ' ' + last_name).pop();
            }
        }, {
            prompt: 'last name: '
        });
    }
});
```

But there are differences, by default read will disable history with push you can have history if you want but you still can disable it using
history `this.cmd().history().disable();` cmd is another plugin that is used internally but you can also use it on your own, it have simplified API.
Here is [demo if simple terminal created using cmd plugin](https://codepen.io/jcubic/pen/XaoqGp). The other difference that read will invoke callback and promise with empty last name if user press enter here it will only work if last_name is not empty.

### Masking password

If you want to read password from the user you can use method set mask you can be any string and each character you type will be replaced with this
character, you can also use empty string to have password like in some unix commands like mysql.

```javascript
$('#terminal').terminal(function() {
    this.set_mask('-').read('pass: ').then(pass => this.echo("Your password is " + pass));
});
```

this will set mask to - character if you use true it will us asterisk.

### Authentication

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

### Key Shortcuts

The best way to add key shortcuts is to use keymap option which accept object with shortcut as strings and function that will be invoked on this shortcut:

```javascript
$(function() {
    $('#terminal').terminal({}, {
        keymap: {
            'CTRL+P': function(e, original) {
                 this.echo('my shortcut');
            }
       }
    });
});
```

the original argument is original function from terminal if set, for instance you can change behavior of enter key using:

```javascript
$(function() {
    var count = 0;
    $('#terminal').terminal({}, {
        keymap: {
            'ENTER': function(e, original) {
                 original();
                 this.set_prompt(this.get_prompt().replace(/[0-9]*> /, ++count + '> '));
            }
       }
    });
});
```

this will change the prompt with counter for each enter.

The order of mnemonics look like this CTRL+META+SHIFT+ALT and at the begining is special modifier HOLD the shortcut will be invoked when you hold the key.

### Events

Terminal have a bunch of events, you can specify them in options, most events have only terminal as with few exceptions like `keypress` or `keydown`, which was old API before
`keymap` was added, that have event as first argument and send is terminal. There are also events that have current command as first argument or error object.

[List of all events can be found in API reference](https://terminal.jcubic.pl/api_reference.php#onAfterCommand)

### Asynchronous Commands

If you have Async commands, which mean commands that execute later in time you should pasue the terminal and resume it after you've done processing, this will ensure
that your command will be invoked properly (mostly because of one feature which is invoking commands from URL hash or using exec method, discussed later in this document).
If you don't care about order of commands you don't need to pause the terminal.

To pause the terminal and don't accept input from user, you can use pause() method and to resume the state you can use `resume()` method you can also return promise
from interpreter, but the resolved value will be printed on terminal, to avoid this you can resolve to undefined.

```javascript
$('#terminal').terminal(function(command, term) {
    return new Promise(function(resolve) {
        setTimeout(function() {
            term.echo('delay command ' + command);
            resolve();
        }, 1000);
    });
});

$('#terminal').terminal(function(command) {
    this.pause();
    setTimeout(() => {
        this.echo('delay command ' + command).resume();
    }, 1000);
});
```

### Saving State

You can save two types of states:

1. You can save state of the whole terminal including prompt, cursor position, lines on terminal or your interpreter.
to save state you can use function
```javascript
var view = term.export_view(); 
```

it will return object that can be use to restore the state of the terminal:

```javascript
term.import_view(view); 
```

NOTE: The object that is returned by export_view will not work with JSON.stringify unless you have a way to serialize functions. But it
will work with few limitations if you strip out functions.

2. Another type of state is history of commands you type in single session that is saved in url hash (text after # that is used to jump
to location with same id in html).

To start saving commands in hash you can invoke function:

```javascript
term.history_state(true);
```

Saving to hash is controlled by option historyState so you can set it to true but be careful because there is limit of URL length.

You can also add command to hash using

```javascript
term.save_state('foo');
```

If you refresh the browser and have option execHash set to true it will execute all commands in order. And you can share the link with commands
to your app.

But there is limitation different browsers have different max URL limit, so best is only save few commands. Setting option `historyState: true`. Also can make your app break because of this limit. So best is to have some command that toggle saving in hash using `term.history_state` function.

### Executing commands from JavaScript

There are few different types of executing commands, you can use method term.exec to invoke different command:

```javascript
$('#terminal').terminal({
    open: function() {
        this.echo('you try to open').exec('close');
    },
    close: function() {
        this.echo('you closed');
    }
});
```

if you type open you will see:

```
> open
you try to open
> close
you closed
```

If you don't want that > close to be echo (by default exec act like if you type that command and press enter) you can call:

```javascript
term.exec('close', true);
```
You can also put array in exec:
```javascript
term.exec([
   'git clone https://github.com/jcubic/jquery.terminal.git',
   'cd jquery.terminal',
   'git checkout devel'
]);
```

and it will invoke commands in order, so if you have async commands it's important to have pause/resume or to returning promises from interpreter.

Second type is invoking commands using url, if you saved some commands in url hash and if you refresh the browser, all the commands will be
executed in order the same as exec with array.

Last type is invoking terminal shortcuts, you can do that with term.invoke_key method. 
```javascript
term.invoke_key("CTRL+R");
```

this will invoke control + R key which by default is bind to reverse search. This will invoke any shortcut added by keymap option as well as
build in keymap options.

### Invoking Commands and terminal methods from Server

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

### Updating lines

You can also update the lines that gets echo to the terminal using update method.

```javascript
term.update(0, "hello world")
```

this will replace the greetings, you can also put negative values into first argument. To update last line you can use:

```javascript
term.echo("hello");
term.update(-1, "hello world");
```

there is also function that return index of last command which is term.last_index. This function is better if you need to update last line
and you know that it's last so you can use last_index and you need to call update later when there are already some text added after.

```javascript
term.update(term.last_index(), "hello")
```

this will be the same as -1.