---
layout: post
title: Determine Entropy of a Password
---

# Introduction

Entropy, as it relates to passwords, is a measure of how many attempts it will take during a search to exhaust all possibilities of your password. I have created pscalc to quickly let you calculate the entropy of a password given a few bits of information.

## Installation

Install [Perl], and then grab the code from <https://github.com/hadenpike/pscalc.git>. Make sure the script is in your PATH:

```
export PATH="/path/to/pscalc/:${PATH}"
```

in Bash, for example. Documentation for this script follows.

# NAME

pscalc --- Determine entropy of a password

# SYNOPSIS

$ pscalc [size_of_alphabet] [password_size]

# DESCRIPTION

The entropy of a password is determined by:
log(alphabet_size)/log(2)
where alphabet_size is equal to the sum of the allowed characters
(95 for lowercase, uppercase, digits, and the 33 special characters),
and n is the length of the password.

# EXAMPLE

Assume we are using the standard alphabet size (95), and we have a twelve character password. Using this tool:

```
$ pscalc 95 12
78.84
```

Thus it would take
2^79 = 6.04462909807315e+23
attempts to try all possibilities. Consider the speed of computers and increase the numbers passed to pscalc, and you'll see that's not actually a great result.

# REFERENCE

Thanks to Steve Gibson
<https://www.grc.com/>
for pointing this formula out on the Security Now podcast
<https://twit.tv/sn>.

[Perl]: https://www.perl.org/
