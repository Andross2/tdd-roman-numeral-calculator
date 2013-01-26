Chapter 1 - A simple TDD example - the Roman Calculator
-------------------------------------------------------

I wanted to start with a simple example. Everyone always uses the same
one, which is a Roman Numerals, but I’m going to give it a little twist,
which is that I’ll try and use TDD to make a Roman numeral *calculator*
- not a Roman Numeral converter. Along the way, we’ll discover something
I thought was quite cool too.

### A first test for a minimum viable calculator

In order for our development to be truly test-driven, tests have to come
first - before we write any application code, we first write a test for
it.

We start with the simplest possible thing — TDD is very much an agile
methodology, so we start to think about what the "minimum viable Roman
numeral calculator" might do… What would be just slightly better than
not having a calculator at all? How about one that can add 1 + 1?

Here’s the simplest possible test for that:

    from rome import add

    assert add('I', 'I') == 'II'

I’m going to save that as `tests.py`. Let’s try running it. “What?”, I
hear you say, “that’s totally pointless — we haven’t even made a
rome.py, or an add function, it’s just a waste of time to run the test
now!”

The reason I can hear you saying that is because it’s exactly what I
said when I first started learning TDD at PythonAnywhere. Believe me
when I say that I was dragged incredibly reluctantly into the world of
TDD. I dragged my heels, raged at every "pointless" bit of testing,
moaned about the ridiculously pernickety procedure we would
follow — “but it’s obvious where we want to be, why don’t we just skip
to the end”.

There are different justifications for it. One is that, even though we
could easily skip a bunch of steps in these easy cases, we just do it as
practice, because it’s a great skill to have for when we get into the
not-so-easy cases. Another is that, in purely practical terms, it really
doesn’t waste any serious time, and it might even help us catch some
early bugs — if there’s a syntax error in our tests, we may as well
catch it right now rather than later.

    Traceback (most recent call last):
      File "tests.py", line 1, in <module>
        from rome import add
    ImportError: No module named rome

### Baby steps: small, incremental code changes, driven by the tests

Once we have a failing test, the next thing to do is make the smallest
possible code change that will get the tests closer to passing. The
"Driven" bit in TDD means we let the tests tell us what to do. In this
case, they’re complaining of an import error, so let’s fix that - and
only that.

We can do it by creating an empty file called rome.py

    touch rome.py

(if you’re on Windows, you probably won’t have the "touch" command. I’m
sure you can figure out some other way of making an empty file called
rome.py. In general, I’ll try and make the commands I use OS-agnostic…)

Now we can re-run our test!

    Traceback (most recent call last):
      File "tests.py", line 1, in <module>
        from rome import add
    ImportError: cannot import name add

Progress! We’ve moved from one kind of ImportError to another one. Let’s
fix that! Open up `rome.py` and add 1 line of code:

    add = None

And re-run the test

    Traceback (most recent call last):
      File "tests.py", line 3, in <module>
        assert add('I', 'I') == 'II'
    TypeError: 'NoneType' object is not callable

Yeah! Seriously! It’s perverse, isn’t it? Now, I’ll admit, perhaps in
real life I would have jumped straight to defining a function called
"add".

Kent Beck put it this way: “Do I really expect you to always write code
like this? No. But I want you to always be *able* to”. These sorts of
baby steps seem nonsensical when we’re dealing with an easy example like
this, but once you start talking about making changes to a complex
application, being able to use baby steps makes for a much less
stressful way of coding. Taking small steps, with constant reference to
tests, gives us reassurance that we’ll go from code that works to code
that still works without breaking anything.

By only making minimal changes, and justifying each change with
reference to getting a test to pass, we make sure that every bit of your
code is tested. It also tests the tests - running them frequently and
seeing that they fail in the way you expect, makes sure that the tests
really work the way you think they do.

So let’s get them a little further along:

    def add():
        pass

Now we expect a moan about arguments:

    Traceback (most recent call last):
      File "tests.py", line 3, in <module>
        assert add('I', 'I') == 'II'
    TypeError: add() takes no arguments (2 given)

Let’s fix that:

    def add(augend, addend):
        pass

(Hat tip to Kent Beck once again for teaching me the word
"augend" — amongst many other things!)

    Traceback (most recent call last):
      File "tests.py", line 3, in <module>
        assert add('I', 'I') == 'II'
    AssertionError

“AssertionError” isn’t very helpful - let’s make our tests a bit more
verbose in their failure message:

    from rome import add

    result = add('I', 'I')
    assert result == 'II', 'add did not return II, it returned %s' % result

That’s a bit better:

    Traceback (most recent call last):
      File "tests.py", line 4, in <module>
        assert result == 'II', 'add did not return II, it returned %s' % result
    AssertionError: add did not return II, it returned None

Writing our own error messages is a bit tedious though. Time to bring in
some testing tools:

### The Python `unittest` module

    import unittest
    from rome import add

    class AdditionTest(unittest.TestCase):

        def test_adding_Is(self):
            self.assertEqual(add('I', 'I'), 'II')


    if __name__ == '__main__':
        unittest.main()

`unittest` is Python’s main built-in testing tool; it has lots of tools
to help structure our tests, tools to run them and report passes and
failures, and dozens of useful functions like `assertEqual`, which can
help us to make assertions about our code, and report back with useful
messages:

    F
    ======================================================================
    FAIL: test_adding_Is (__main__.AdditionTest)
    ----------------------------------------------------------------------
    Traceback (most recent call last):
      File "tests.py", line 7, in test_adding_Is
        self.assertEqual(add('I', 'I'), 'II')
    AssertionError: None != 'II'

    ----------------------------------------------------------------------
    Ran 1 test in 0.001s

    FAILED (failures=1)

Tests in `unittest` are structured as methods on classes - the rule is
that any method whose name starts with `test` will get run.
`unittest.main()` will run all the tests in the current module, and
print out any errors. You can see in the output that the failure message
and traceback is neatly preceded by the class name (`AdditionTest`) and
the test method name (`test_adding_Is`).

Now let’s see what we can do to get these tests passing:

    def add(augend, addend):
        return 'II'

    .
    ----------------------------------------------------------------------
    Ran 1 test in 0.000s

    OK

Well, OK, the test is passing, but it’s pretty clear that we’ve cheated.
Still, returning a hard-coded value like this is often a useful first
step. If you’re dealing with complex code that’s built into a huge
application, a function that “cheats” is still useful, because we can
then check that it integrates with the rest of the application.

We know we’ve cheated, and it’s definitely nagging at us though. But, in
order to justify changing the code, we have to write another test:

    def test_adding_Is(self):
        self.assertEqual(add('I', 'I'), 'II')
        self.assertEqual(add('I', 'II'), 'III')

That gives us a failing test:

    F
    ======================================================================
    FAIL: test_adding_Is (__main__.AdditionTest)
    ----------------------------------------------------------------------
    Traceback (most recent call last):
      File "tests.py", line 8, in test_adding_Is
        self.assertEqual(add('I', 'II'), 'III')
    AssertionError: 'II' != 'III'

    ----------------------------------------------------------------------
    Ran 1 test in 0.001s

    FAILED (failures=1)

Which lets us have another crack at it, and write a slightly better
implementation of add:

    def add(augend, addend):
        return augend + addend

    .
    ----------------------------------------------------------------------
    Ran 1 test in 0.000s

    OK

Hooray! A passing test. You might think it’s a slightly weird calculator
so far, and that using string concatenation to implement addition of
numbers isn’t likely to be a viable solution long-term, but for now, it
works fine.

So that’s our first chapter. It covered:

-   Using TDD to write the minimum viable Roman Numeral calculator

-   the First rule: Test First!

-   how to code with TDD: minimal, incremental changes.

-   the Python `unittest` module

In the next chapter we’ll take our calculator a little further, and
learn about testing exceptions and using loops to structure test data.

* * * * *

Last updated 2013-01-26 20:36:42 GMT
