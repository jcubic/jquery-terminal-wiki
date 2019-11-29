Terminal have a bunch of events, you can specify them in options, most events have only terminal as with few exceptions like `keypress` or `keydown`, which was old API before
`keymap` was added, that have event as first argument and send is terminal. There are also events that have current command as first argument or error object.

[List of all events can be found in API reference](https://terminal.jcubic.pl/api_reference.php#onAfterCommand)
