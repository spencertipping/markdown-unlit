# Markdown literate compiler
```perl
#!/usr/bin/env perl
use strict;
use warnings;
```

## Parser
This parser takes Markdown text and returns a list of parsed elements in order,
each of which is one of the following:

- `"link destination"`: the `Y` part of `[X](Y)`, as a string
- `[line_number, "fence_language", "block_text"]`: a description of each fenced
  code block within the markdown

```perl
sub parse($)
{ my @lines   = ('', split /\n/, $_[0]);
  my @toggles = (0, grep $lines[$_] =~ /^\s*\`\`\`/, 1..$#lines);
  (map((join("\n", @lines[$toggles[$_-2]..$toggles[$_-1]]) =~ /\]\(([^)]+\.md)\)/g,
        [$toggles[$_-1] + 1,
         $lines[$toggles[$_-1]] =~ /^\s*\`\`\`(.*)$/,
         join "\n", @lines[$toggles[$_-1]+1..$toggles[$_]-1]]),
       grep !($_ & 1), 2..$#toggles),
   join("\n", @lines[$toggles[-1]..$#lines]) =~ /\]\(([^)]+\.md)\)/g) }
```

## Compiler
Returns a list of block text in the targeted language(s), in the order they
appeared in the Markdown files. Link ordering is also respected, so a link
between two blocks will do what you expect.

The compiler automatically inserts `#line` directives for Perl; this results in
the Perl interpreter giving warnings and errors in terms of the original
Markdown files and line numbers.

```perl
our %languages;

sub compile($);
sub compile($)
{ my ($f) = @_;
  open my $fh, "< $f" or die "open $f: $!";
  map ref() ? $languages{$$_[1]}
              ? $$_[1] =~ /^pl|^perl/ ? "#line $$_[0] \"$f\"\n$$_[2]\n" : $$_[2]
              : ()
            : compile $f =~ s|[^/]*$||r . $_,
      parse join '', <$fh> }

$languages{$1} = shift @ARGV while @ARGV && $ARGV[0] =~ /^--(.*)/;
print for map compile($_), @ARGV;
```
