# NAME

Perl::Critic::Policy::RegularExpressions::ProhibitRegexToStripAffix - Take a known prefix or suffix off a string without a regex capture.

# VERSION

version 0.001

# Perl::Critic::Policy::RegularExpressions::ProhibitRegexToStripAffix

A match that anchors a run of literal text to one end of the string and
captures the rest is taking a prefix or a suffix off.  `substr` does that
without compiling a pattern, running it, and copying out a capture:

```perl
my ($name) = $file =~ m/\A(\w+)\.pm\z/;                       # reported

my $name = substr( $file, -3 ) eq '.pm' ? substr( $file, 0, -3 ) : undef;

my ($rest) = $line =~ m/\Aprefix-(.*)\z/;                     # reported

my $rest = index( $line, 'prefix-' ) == 0 ? substr( $line, 7 ) : undef;
```

## PROHIBITED

```
m/\A(\w+)\.pm\z/              # a suffix, anchored at both ends
m/^(.*)\.tar\.gz$/            # $ and ^ as well as \A and \z
m/(.+)\.bak\z/                # .* or .+ reaches the start on its own
m/\Aprefix-(.*)\z/            # a prefix
m/\Afoo:(.*)/                 # a prefix, and .* reaches the end on its own
m/\A(\w+) \. pm\z/x           # whitespace under /x is not text
```

## ALLOWED

```perl
m/(\w+)\.pm\z/                # unanchored: the last run of word characters
m/\A(\w+)\.(pm|pl)\z/         # alternation, and two captures
m/\A(\w+)\.pm\z/i             # /i: substr's comparison is case sensitive
m/^(.*)\.pm$/m                # /m: ^ and $ are line anchors
m/\A(\w+)\.pm\z/g             # /g
m/\A(\w+)[.]pm\z/             # a character class is not literal text
m/\A(\w+)$ext\z/              # interpolation
m/\A(\w+)\z/                  # nothing to strip
s/\.pm\z//                    # a substitution
qr/\A(\w+)\.pm\z/             # a compiled regex, which may be used elsewhere
```

## CAVEATS

A capture of anything but `.*` or `.+` also checks what it captures:
`m/\A(\w+)\.pm\z/` refuses `foo-bar.pm`, and `substr` takes the suffix off
whatever is in front of it.  When that check is the point, keep the regex and
say `## no critic (ProhibitRegexToStripAffix)` and why.

`$` and `\Z` also match before a newline at the end of the string, and `.`
does not match a newline without `/s`.  `substr` knows nothing of lines, so
for text that can end in a newline, `chomp` it first.

A modifier turned on by `use re` is not seen, only one written on the match.

## METHODS

### supported\_parameters

### default\_severity

### default\_themes

### applies\_to

### violates

# AUTHORS

Current Maintainers:

- George S. Baugh <george@troglodyne.net>

# COPYRIGHT AND LICENSE

Copyright (c) 2026 Troglodyne LLC

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:
The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
