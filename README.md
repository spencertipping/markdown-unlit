# Literate programming in Markdown
`unlit` looks for fenced code segments and follows Markdown links to construct
a single stream of source code.

## Usage
```
$ unlit --perl file.md | perl
```

`--perl` tells `unlit` to consider `perl` fenced blocks as source; others will
be ignored. You can use any language, or multiple languages.
