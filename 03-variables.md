---
layout: page
title: Automation and Make
subtitle: Automatic variables
minutes: 0
---

> ## Learning Objectives {.objectives}
>
> * Use Make automatic variables to remove duplication in a Makefile.
> * Use `$@` to refer to the target of the current rule.
> * Use `$^` to refer to the dependencies of the current rule.
> * Use `$<` to refer to the first dependency of the current rule.
> * Explain why bash wild-cards in dependencies can cause problems.

After the exercise at the end of the previous part, our Makefile look like this:

~~~ {.make}
# Generate summary table.
results.txt : isles.dat abyss.dat last.dat
        python zipf_test.py abyss.dat isles.dat last.dat > results.txt

# Count words.
.PHONY : dats
dats : isles.dat abyss.dat last.dat

isles.dat : books/isles.txt
        python wordcount.py books/isles.txt isles.dat

abyss.dat : books/abyss.txt
        python wordcount.py books/abyss.txt abyss.dat

last.dat : books/last.txt
        python wordcount.py books/last.txt last.dat

.PHONY : clean
clean :
        rm -f *.dat
        rm -f results.txt
~~~

Our Makefile has a lot of duplication. For example, the names of text
files and data files are repeated in many places throughout the
Makefile. Makefiles are a form of code and, in any code, repeated code
can lead to problems e.g. we rename a data file in one part of the
Makefile but forget to rename it elsewhere.

> ## D.R.Y. (Don't Repeat Yourself) {.callout}
>
> In many programming languages, the bulk of the language features are there  to allow the programmer to describe long-winded computational routines as short, expressive, beautiful code.
> Features in Python or R or Java like user-defined variables and functions are useful in part because they mean we don't have to write out (or think about) all of the details over and over again.
> This good habit of writing things out only once is known as the "Don't Repeat Yourself" principle or D.R.Y.

Let us set about removing some of the repetition from our Makefile.

In our `results.txt` rule we duplicate the data file names and the name of the results file name:

~~~ {.make}
results.txt : isles.dat abyss.dat last.dat
        python zipf_test.py abyss.dat isles.dat last.dat > results.txt
~~~

Looking at the results file name first, we can replace it in the action
with `$@`:

~~~ {.make}
results.txt : isles.dat abyss.dat last.dat
        python zipf_test.py abyss.dat isles.dat last.dat > $@
~~~

`$@` is a Make [automatic variable](reference.html#automatic-variable)
which means 'the target of the current rule'. When Make is run it will
replace this variable with the target name.

We can replace the dependencies in the action with `$^`:

~~~ {.make}
results.txt : isles.dat abyss.dat last.dat
        python zipf_test.py $^ > $@
~~~

`$^` is another automatic variable which means 'all the dependencies
of the current rule'. Again, when Make is run it will replace this
variable with the dependencies.
 
Let's update our text files and re-run our rule:

~~~ {.bash}
$ touch books/*.txt
$ make results.txt
~~~

We get:

~~~ {.output}
python wordcount.py books/isles.txt isles.dat
python wordcount.py books/abyss.txt abyss.dat
python wordcount.py books/last.txt last.dat
python zipf_test.py isles.dat abyss.dat last.dat > results.txt
~~~

We can use the bash wild-card in our dependency list:

~~~ {.make}
results.txt : *.dat
        python zipf_test.py $^ > $@
~~~

Let's update our text files and re-run our rule:

~~~ {.bash}
$ touch books/*.txt
$ make results.txt
~~~

We get the same as above.

Now let's delete the data files and re-run our rule:

~~~ {.bash}
$ make clean
$ make results.txt
~~~

We get:

~~~ {.error}
make: *** No rule to make target `*.dat', needed by `results.txt'.  Stop.
~~~

As there are no files that match the pattern `*.dat` the name `*.dat`
is used as a file name itself, and there is no file matching that, nor
any rule so we get an error. We need to explicitly rebuild the `.dat`
files first:

~~~ {.bash}
$ make dats
$ make results.txt
~~~

> ## Update dependencies {.challenge}
> 
> What will happen if you now execute:
> 
> ~~~ {.bash}
> $ touch *.dat
> $ make results.txt
> ~~~
> 
> 1. nothing
> 2. all files recreated
> 3. only `.dat` files recreated
> 4. only `results.txt` recreated

As we saw, `$^` means 'all the dependencies of the current
rule'. This works well for `results.txt` as its action
treats all the dependencies the same - as the input for the `zipf_test.py` script.

However, for some rules, we may want to treat the first dependency
differently. For example, our rules for `.dat` use their first (and
only) dependency specifically as the input file to `wordcount.py`. If
we add additional dependencies (as we will soon do) then we don't want
these being passed as input files to `wordcount.py` as it expects only
one input file to be named when it is invoked.

Make provides an automatic variable for this, `$<` which means 'the
first dependency of the current rule'. 

> ## Rewrite `.dat` rules to use automatic variables {.challenge}
>
> Rewrite each `.dat` rule to use the automatic variables `$@` ('the
> target of the current rule') and `$<` ('the first dependency of the
> current rule').
