# jq filtering

Today I'm reminded of how useful jq is for filtering/displaying the contents of a json file, so that it is readable.

It's nice to be reminded of these things from time to time.

```
cat file.json | jq "." > newfile.json
```

The will "filter" the entire file, and redirect it to a new file.

```
jq "." file.json
```
displays the entire json file on screen in a readable format.

Reminder: The filter can be used to display sections of the file.
