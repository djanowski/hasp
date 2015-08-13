Hasp
====

Half-assed CSS preprocessor.

Motivation
----------

Tired of all the slow and bloated preprocessors out there, I talked to
[@soveran][soveran] about writing my own take, which would address only basic
functionality:

- Be very fast.
- Support variables.
- Support includes.
- Implement media queries efficiently.

The last point is especially important to me. I want to define how a selector
behaves on all breakpoints in a single place.  On the other hand, producing
n `@media` declarations is wasteful. So my solution would buffer all rules
defined for a breakpoint, group them and print them at the end of the resulting
file.

[Michel][soveran] suggested I look at [M4][m4], a macro processor you already
have on your system.

The result is a 30-LOC minimal preprocessor. You know, "it's okay if it's
half-assed as long as it's the right half of the ass".


Usage
-----

Open `styles.hcss` (the extension doesn't matter, but you may want to name the file
something other than `.css`):

```
set(PRIMARY_COLOR, `#f00')

body {
  background: PRIMARY_COLOR;
}
```

Compile:

```
$ hasp styles.hcss
```

As expected, the variable was replaced.

Now let's use breakpoints for mobile-first development:

```
breakpoint(TABLET, min-device-width: 768px)

selector(.site-header) {
  height: 1rem;

  on(TABLET)
    height: 3rem;
  end
}

selector(.main-content) {
  padding: 0;

  on(TABLET)
    padding: 5rem;
  end
}
```

The output:

```css
.site-header {
  height: 1rem;
}

.main-content {
  padding: 0;
}

@media only screen and (min-device-width: 768px) {
  .site-header {
    height: 3rem;
  }

  .main-content {
    padding: 5rem;
  }
}
```

You'll see there's only one `@media` declaration. w00t!


But it's not a superset of CSS!
-------------------------------

Who cares? Do you use XSLT to render your HTML templates?


OK, should I use this in production?
------------------------------------

Yes, you can use it in production. If you do, please [let me know][djanowski].

If you're still not sold on the idea, and are using Less or Sass, make yourself
a favor and [switch to something better][postcss]. Anyway, they all require
Node.js, a huge dependency for this simple problem, in my opinion.


What about minification?
------------------------

Hasp assumes you'll minify its output, so it doesn't make any effort in
producing compressed (or pretty) output.


What about sourcemaps?
----------------------

`¯\_(ツ)_/¯`


How do I watch and rebuild?
---------------------------

In general, all other solutions use very high-level languages with slow
startup times, like Node.js or Java.  So they need to support file-watching
functionality in the same runtime, so as to avoid the slow startup.

M4 is so fast that there's no need to keep a runtime alive. Just use the beautiful
[entr][entr]:

```
$ while true; do find . -name '*.hcss' | entr -rd sh -c 'hasp styles.hcss'; done
```


Syntax reference
----------------

**`set(<name>, <value>)`**

Defines a variable. Per convention, variables names should be upper-case.

Example:

    set(BACKGROUND, white)

    body {
      background: BACKGROUND;
    }


**`include(<file>)`**

Includes another Hasp file.

Example:

    body {
      background: white;
    }

    include(path/to/other.hcss)


**`breakpoint(<name>, <expression>)`**

Defines a named breakpoint for later use.

Example:

    breakpoint(TABLET,  min-device-width: 768px)
    breakpoint(DESKTOP, min-device-width: 900px)


**`selector(<rule>)`**
**`on(<breakpoint>)`**

Declares a rule that will be affected by breakpoints.

Example:

    breakpoint(TABLET,  min-device-width: 768px)

    selector(.main-content) {
      width: 10rem;

      on(TABLET)
        width: 20rem;
      end
    }


Examples
--------

Check out the examples directory.


[djanowski]: https://twitter.com/djanowski
[entr]:      http://entrproject.org
[m4]:        http://www.gnu.org/software/m4/m4.html
[postcss]:   https://github.com/postcss/postcss
[soveran]:   https://twitter.com/soveran
