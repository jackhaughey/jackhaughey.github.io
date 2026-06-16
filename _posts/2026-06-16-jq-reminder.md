# jq filtering

Today I'm reminded of how useful jq is for filtering/displaying the contents of a json file, so that it is readable.

It's nice to be reminded of these things from time to time.

```
cat file.json | jq "."
```

The will "filter" the entire file.
