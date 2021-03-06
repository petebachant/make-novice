---
layout: page
title: Automation and Make
subtitle: Functions
minutes: 0
---

> ## Learning Objectives {.objectives}
>
> * Use Make's `wildcard` function to get lists of files matching a
>   pattern. 
> * Use Make's `patsubst` function to rewrite file names.

At this point, we have the following Makefile:

~~~ {.make}
include config.mk

# Generate summary table.
results.txt : *.dat $(ZIPF_SRC)
        $(ZIPF_EXE) *.dat > $@

# Count words.
.PHONY : dats
dats : isles.dat abyss.dat last.dat

%.dat : books/%.txt $(COUNT_SRC)
        $(COUNT_EXE) $< $*.dat

.PHONY : clean
clean :
        rm -f *.dat
        rm -f results.txt
~~~

Make has many [functions](reference.html#function) which can be used to
write more complex rules. One example is `wildcard`. `wildcard` gets a
list of files matching some pattern, which we can then save in a
variable. So, for example, we can get a list of all our text files
(files ending in `.txt`) and save these in a variable by adding this at
the beginning of our makefile:

~~~ {.make}
TXT_FILES=$(wildcard books/*.txt)
~~~

We can add `.PHONY` target and rule to show the variable's value:

~~~ {.make}
.PHONY : variables
variables:
	@echo TXT_FILES: $(TXT_FILES)
~~~

> ## @echo {.callout}
>
> Make prints actions as it executes them. Using `@` at the start of
> an action tells Make not to print this action. So, by using `@echo`
> instead of `echo`, we can see the result of `echo` (the variable's
> value being printed) but not the `echo` command itself.

If we run Make:

~~~ {.bash}
$ make variables
~~~

We get:

~~~ {.output}
TXT_FILES: books/abyss.txt books/isles.txt books/last.txt books/sierra.txt
~~~

Note how `sierra.txt` is now included too.

The following figure shows the dependencies embodied within our Makefile, involved in building the `results.txt` target, once we have introduced our function:

![results.txt dependencies after introducing a function](img/07-functions.png "results.txt dependencies after introducing a function")

`patsubst` ('pattern substitution') takes a pattern, a replacement string and a
list of names in that order; each name in the list that matches the pattern is 
replaced by the replacement string. Again, we can save the result in a
variable. So, for example, we can rewrite our list of text files into
a list of data files (files ending in `.dat`) and save these in a
variable:

~~~ {.make}
DAT_FILES=$(patsubst books/%.txt, %.dat, $(TXT_FILES))
~~~

We can extend `variables` to show the value of `DAT_FILES` too:

~~~ {.make}
.PHONY : variables
variables:
        @echo TXT_FILES: $(TXT_FILES)
        @echo DAT_FILES: $(DAT_FILES)
~~~

If we run Make,

~~~ {.bash}
$ make variables
~~~

then we get:

~~~ {.output}
TXT_FILES: books/abyss.txt books/isles.txt books/last.txt books/sierra.txt
DAT_FILES: abyss.dat isles.dat last.dat sierra.dat
~~~

Now, `sierra.txt` is processed too.

With these we can rewrite `clean` and `dats`:

~~~ {.make}
.PHONY : dats
dats : $(DAT_FILES)

.PHONY : clean
clean :
        rm -f $(DAT_FILES)
        rm -f results.txt
~~~

Let's check:

~~~ {.bash}
$ make clean
$ make dats
~~~

We get:

~~~ {.output}
python wordcount.py books/abyss.txt abyss.dat
python wordcount.py books/isles.txt isles.dat
python wordcount.py books/last.txt last.dat
python wordcount.py books/sierra.txt sierra.dat
~~~

We can also rewrite `results.txt`:

~~~ {.make}
results.txt : $(DAT_FILES) $(ZIPF_SRC)
        $(ZIPF_EXE) *.dat > $@
~~~

If we re-run Make:

~~~ {.bash}
$ make clean
$ make results.txt
~~~

We get:

~~~ {.output}
python wordcount.py books/abyss.txt abyss.dat
python wordcount.py books/isles.txt isles.dat
python wordcount.py books/last.txt last.dat
python wordcount.py books/sierra.txt sierra.dat
python zipf_test.py *.dat > results.txt
~~~

We see that the problem we had when using the bash wild-card, `*.dat`,
which required us to run `make dats` before `make results.txt` has
now disappeared, since our functions allow us to create `.dat` file
names from those `.txt` file names in `books/`.

Let's check the `results.txt` file:

~~~ {.bash}
$ cat results.txt
~~~

~~~ {.output}
Book	First	Second	Ratio
abyss	4044	2807	1.44
isles	3822	2460	1.55
last	12244	5566	2.20
sierra	4242	2469	1.72
~~~

So the range of the ratios of occurrences of the two most frequent words in our books is indeed around 2, as predicted by Zipf's law.

most frequently-occurring word occurs approximately twice as often as the second most frequent word

Here is our final Makefile:

~~~ {.make}
include config.mk

TXT_FILES=$(wildcard books/*.txt)
DAT_FILES=$(patsubst books/%.txt, %.dat, $(TXT_FILES))

# Generate summary table.
results.txt : $(DAT_FILES) $(ZIPF_SRC)
	$(ZIPF_EXE) *.dat > $@

# Count words.
.PHONY : dats
dats : $(DAT_FILES)

%.dat : books/%.txt $(COUNT_SRC)
	$(COUNT_EXE) $< $*.dat

.PHONY : clean
clean :
	rm -f $(DAT_FILES)
	rm -f results.txt

.PHONY : variables
variables:
	@echo TXT_FILES: $(TXT_FILES)
	@echo DAT_FILES: $(DAT_FILES)
~~~

Remember, the `config.mk` file contains:

~~~ {.make}
# Count words script.
COUNT_SRC=wordcount.py
COUNT_EXE=python $(COUNT_SRC)

# Test Zipf's rule
ZIPF_SRC=zipf_test.py
ZIPF_EXE=python $(ZIPF_SRC)
~~~
