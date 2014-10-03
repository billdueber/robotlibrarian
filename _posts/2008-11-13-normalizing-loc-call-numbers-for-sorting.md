---
title: "Normalizing LoC Call Numbers for sorting"
date: 2008-11-13
tags: "callnumbers, code"
layout: post

---

**Updated**: <em>I missed a '?' in the original code that pushed a single cutter into the second-cutter position. Fixed below.</em>

<strong>Crap. Update 2</strong>: Initial letters can be three characters long. Regexp and output changed.

LoC Call numbers tend to be a mess, and I've been working this morning trying to normalize them for easy string comparison.

The perl function below takes a call number (with some level of sloppiness) and returns a string suitable for comparisons with other strings returned by the function. It outputs stuff like this:

~~~

E                          E 0000.0000  0000  0000
E 184 .A1 G78              E 0184.0000A 1000G 7800
E184.A2 G78 1967           E 0184.0000A 2000G 7800 1967
E184.A2 G78 1970           E 0184.0000A 2000G 7800 1970
EA                         EA0000.0000  0000  0000
EA 10                      EA0010.0000  0000  0000
EA 10 1970                 EA0010.0000  0000  0000 1970
EA10 B7                    EA0010.0000B 7000  0000
EA 10.B7.G8                EA0010.0000B 7000G 8000
EA10.5                     EA0010.5000  0000  0000

~~~~

The code, in perl, follows:

~~~perl

sub normalizeLC {
  my $lc = uc(shift);
  $lc =~ /^
          \s*
          ([A-Z]{1,3})  # alpha
          \s*
          (         # optional numbers
            \d+
            (?: \s*\.\s*\d+)?  # ...with optional decimal point
          )?
          \s*
          (?:               # optional cutter
            \.? \s*
            ([A-Z]+)      # cutter letter
            \s*
            (\d+)?        # cutter numbers
          )?
          (?:               # optional cutter
            \.? \s*
            ([A-Z]+)      # cutter letter
            \s*
            (\d+)?        # cutter numbers
          )?
          \s*
          (.*?)            # everthing else
          \s*$
        /x;
  my ($alpha, $num, $c1alpha, $c1num, $c2alpha, $c2num, $extra) = ($1, $2, $3, $4, $5, $6, $7);
  $c1num .= 0 x (4 - length($c1num)); # Pad out to four decimal places
  $c2num .= 0 x (4 - length($c2num)); # ditto
  $extra = ' ' . $extra if ($extra);
  return sprintf("%-3s%09.4f%-2s%4s%-2s%4s%s", $alpha, $num, $c1alpha, $c1num, $c2alpha, $c2num, $extra);
}
~~~
