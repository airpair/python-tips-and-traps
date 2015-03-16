<div style="max-width:640px;background-color:#f5f5f5;padding:10px;">
<b>TL;DR: </b>
Python is a (powerful) general purpose language in broad use, let's dive in and
learn some control flow tips, standard library tricks, and common pitfalls.
</div>

## 1 Introduction

Python (and its libraries) are enormous. It is used for system automation, web
applications, big data, analytics, and security software. This article aims to
show off some lesser-known tricks to put you on the path to faster development,
easier debugging, and general fun.

As with every language, the real resource you get once you learn it isn't a
language-related superpower. It's the ability to use the idioms, libraries, and
shared knowledge of the Python community.

## Exploring Standard Data Types

### The Humble `enumerate`

Iterating over the contents of anything in Python is simple, just `for foo in
bar:` and you're off and running.

<!--code lang=python linenums=true-->

    drinks = ["coffee", "tea", "milk", "water"]
    for drink in drinks:
        print("thirsty for", drink)
    #thirsty for coffee
    #thirsty for tea
    #thirsty for milk
    #thirsty for water

But it's common to also want the index of items as well as the items
themselves. It's common to see programmers use `len()` and `range()` to iterate
over a list by index, but there's an easier way.

<!--code lang=python linenums=true-->

    drinks = ["coffee", "tea", "milk", "water"]
    for index, drink in enumerate(drinks):
        print("Item {} is {}".format(index, drink))
    #Item 0 is coffee
    #Item 1 is tea
    #Item 2 is milk
    #Item 3 is water

The `enumerate` builtin yields both the index and the item itself.

### A member of `set`
A surprising number of concepts can boil down to operations on a set. Need to
make sure a list doesn't has duplicates? Need to see what two lists have in
common? Python comes with a `set` type to make these operations fast and
readable.

<!--code lang=python linenums=true-->

    # deduplicate a list *fast*
    print(set(["ham", "eggs", "bacon", "ham"]))
    # {'bacon', 'eggs', 'ham'}


<!--code lang=python linenums=true-->

    # compare lists to find differences/similarities
    # {} without "key":"value" pairs makes a set
    menu = {"pancakes", "ham", "eggs", "bacon"}
    new_menu = {"coffee", "ham", "eggs", "bacon", "bagels"}

    new_items = new_menu.difference(menu)
    print("Try our new", ", ".join(new_items))
    # Try our new bagels, coffee

    discontinued_items = menu.difference(new_menu)
    print("Sorry, we no longer have", ", ".join(discontinued_items))
    # Sorry, we no longer have pancakes

<!--code lang=python linenums=true-->

    old_items = new_menu.intersection(menu)
    print("Or get the same old", ", ".join(old_items))
    # Or get the same old eggs, bacon, ham

    full_menu = new_menu.union(menu)
    print("At one time or another, we've served:", ", ".join(full_menu))
    # At one time or another, we've served: coffee, ham, pancakes, bagels, bacon, eggs


The `intersection` function compares all the items and returns only the items
both sets have in common. In this case, the breakfast staples of bacon, eggs,
and ham.

### collections.namedtuple

When you don't need to attach methods to a class, but still want the
convenience of `foo.prop`, look no further than namedtuple. You define the
fields ahead of time, then can instantiate a lightweight class that takes less
memory than a full object.

<!--code lang=python linenums=true-->

    LightObject = namedtuple('LightObject', ['shortname', 'otherprop'])
    m = LightObject()
    m.shortname = 'athing'
    > Traceback (most recent call last):
    > AttributeError: can't set attribute

You can't set attributes of a `namedtuple`, just like you can't change members
of a tuple. You need to set attributes when you instantiate your `namedtuple`.

<!--code lang=python linenums=true-->

    LightObject = namedtuple('LightObject', ['shortname', 'otherprop'])
    n = LightObject(shortname='something', otherprop='something else')
    n.shortname # something

### collections.defaultdict

It's not uncommon to see logic like this in a Python app, where it's expected
that a key won't exist initially.

<!--code lang=python linenums=true-->

    login_times = {}
    for t in logins:
        if login_times.get(t.username, None):
            login_times[t.username].append(t.datetime)
        else:
            login_times[t.username] = [t.datetime]

With `defaultdict` you can skip this logic by making any access to an undefined
key return an empty list (or any other type).

<!--code lang=python linenums=true-->

    login_times = collections.defaultdict(list)
    for t in logins:
        login_times[t.username].append(t.datetime)

You can even use custom classes, given a callable to build the class.

<!--code lang=python linenums=true-->

    from datetime import datetime
    class Event(object):
        def __init__(self, t=None):
        if t is None:
            self.time = datetime.now()
        else:
            self.time = t

    events = collections.defaultdict(Event)

    for e in user_events:
        print(events[e.name].time)

To go beyond what defaultdict offers and to set nested keys as attributes,
check out [addict](https://github.com/mewwts/addict)

<!--code lang=python linenums=true-->

    normal_dict = {
        'a': {
            'b': {
                'c': {
                    'd': {
                        'e': 'really really nested dict'
                    }
                }
            }
        }
    }

    from addict import Dict
    addicted = Dict()
    addicted.a.b.c.d.e = 'really really nested'
    print(addicted)
    # {'a': {'b': {'c': {'d': {'e': 'really really nested'}}}}}

This snippet is *way* easier to write than it would be with the standard
`dict`, but what about `defaultdict`? Seems like it would be easy enough.


<!--code lang=python linenums=true-->

    from collections import defaultdict
    default = defaultdict(dict)
    default['a']['b']['c']['d']['e'] = 'really really nested dict' # fails

That looks ok, but it will actually throw a `KeyError` exception because
`default['a']` is a `dict`, not a `defaultdict`. Let's make a defaultdict that
defaults to defaulted dictionaries (say *that* a couple times fast).

If you just need a defaulted counter, you can use the
[collections.Counter][counter] class which provides some convenience functions
like `most_common`.

## Control Flow

When learning control structures in Python, it's common to go over `for`,
`while`, `if-elif-else`, and `try-except`. Properly used, those few control
structures can handle most every case. There's a reason equivalents exist in
almost every language you run across. Python also offers some additions to the
basic structures that aren't often used, but can make your code more readable
and easier to maintain.

### Great Exceptations
Exceptions as flow control is a common pattern when dealing with databases,
sockets, files, or any resource that is likely to fail. With the standard `try`
and `except` something simple like working with a database might look like
this.

<!--code lang=python linenums=true-->

    try:
        # get API data
        data = db.find(id='foo') # may raise exception
        # manipulate the data
        db.add(data)
        # save it again
        db.commit() # may raise exception
    except Exception:
        # log the failure
        db.rollback()

    db.close()

Can you spot the problem here? There are *two* possible exceptions that will
trigger the same `except` block. Meaning that failure to *find* the data (or to
connect to find the data) would cause a rollback attempt. This almost
definitely isn't what we want, because a failure at that point wouldn't have
even begun a transaction yet. A rollback also probably isn't the right response
to a connection failure, so let's break these cases apart.

First, we'll handle finding the data.

<!--code lang=python linenums=true-->

    try:
        # get API data
        data = db.find(id='foo') # may raise exception
    except Exception:
        # log the failure and bail out
        log.warn("Could not retrieve FOO")
        return

    # manipulate the data
    db.add(data)

Now that the data retrieval has its own try-except we can take whatever action
makes sense if we don't have any data to work with. It's not likely our code
will do anything useful without data, so we'll just exit the function. Instead
of exiting you could also make a default object, retry the query, or kill the
entire program.

Now let's wrap the `commit` so it fails gracefully as well.

<!--code lang=python linenums=true-->

    try:
        db.commit() # may raise exception
    except Exception:
        log.warn("Failure committing transaction, rolling back")
        db.rollback()
    else:
        log.info("Saved the new FOO")
    finally:
        db.close()

We've actually added two clauses here. First, let's look at the `else`, which
runs if no exception occurs. In our example, all it does is log that the
transaction succeeded, but you could put more interesting actions in as needed.
One potential use would be to fire off a background job or notification.

The `finally` clause is there to make it clear that the `db.close()` will
always run. Looking back, we can see that all the code related to persisting
our data ended up in a nice logical grouping at the same indentation level.
Editing this code later, it will be easy for us to see that all these lines are
tied to the `commit`.

### Context and Control

We've seen control flow using exceptions before. In general, the steps are
something like:

1. Attempt to acquire a resource (file, network connection, whatever)
1. If it fails, clean up anything left behind
1. Otherwise, perform actions on the resource
1. Log what happened
1. Program complete

With that in mind, let's take a second look at the database example from the
last section. We used try-except-finally to make sure that any transaction we
began was either committed or rolled back.

<!--code lang=python linenums=true-->

    try:
        # attempt to acquire a resource
        db.commit()
    except Exception:
        # If it fails, clean up anything left behind
        log.warn("Failure committing transaction, rolling back")
        db.rollback()
    else:
        # If it works, perform actions
        # In this case, we just log success
        log.info("Saved the new FOO")
    finally:
        # Clean up
        db.close()
    # Program complete

Our previous example mapped to the steps above almost exactly. But how much of
this logic ever changes? Not very much.

Just about every time we save data, we'll do these exact same steps. We could
pull this logic into a method, or we could use a context manager.

<!--code lang=python linenums=true-->

    db = db_library.connect("fakesql://")
    # as a function
    commit_or_rollback(db)

    # context manager
    with transaction("fakesql://") as db:
        # retrieve data here
        # modify data here

A context manager makes it easy to protect some block by setting up resources
(context) that the block needs at runtime. In our example, we need a database
transaction that will be:

1. Connected to a database
1. Started at the beginning of the block
1. Committed or rolled back at the end of the block
1. Cleaned up at the end of the block

Let's build a context manager that will hide all this database setup for us.
The `contextmanager` interface is simple. The object is required to have a
`__enter__()` method to set up whatever context is needed and a
`__exit__(exc_type, exc_val, exc_tb)` method that will be called at the end of
the block. If there was no exception, then all three of the `exc_*` arguments
will be `None`.

The `__enter__` method will be pretty simple, so let's start with that.

<!--code lang=python linenums=true-->

    class DatabaseTransaction(object):
        def __init__(self, connection_info):
            self.conn = db_library.connect(connection_info)

        def __enter__(self):
            return self.conn

The `__enter__` method actually does nothing except return the database
connection, which we can use inside the block to retrieve or save data. The
`__init__` method is where the connection is actually made, and if it fails the
block won't run at all.

Now let's define how the transaction will be finished in the `__exit__` method.
This has a lot more to it, since it has to handle any exceptions thrown in the
block and close out the transaction.

<!--code lang=python linenums=true-->

        def __exit__(self, exc_type, exc_val, exc_tb):
            if exc_type is not None:
                self.conn.rollback()

            try:
                self.conn.commit()
            except Exception:
                self.conn.rollback()
            finally:
                self.conn.close()

Now we can use our `DatabaseTransaction` as the context manager for our block
of actions. Under the hood, the `__enter__` and `__exit__` methods will run and
handle setting up the database connection and tear it down when we're through.

<!--code lang=python linenums=true-->

    # context manager
    with DatabaseTransaction("fakesql://") as db:
        # retrieve data here
        # modify data here

To improve our (primitive) transaction manager, we could add handling for
different exception types. Even in its current state, this hides a *ton* of
complexity that you don't need to be worrying about every time you pull in
something from the database.

### Generators

Introduced in Python 2, generators are a simple way to implement an iterator
that doesn't hold all its values at once. Typically a function in Python starts
its execution, does some operations, and returns the result (or nothing).

Generators are different.

<!--code lang=python linenums=true-->

    def my_generator(v):
        yield 'first ' + v
        yield 'second ' + v
        yield 'third ' + v

    print(my_generator('thing'))
    # <generator object my_generator at 0x....>

Instead of `return` we use the `yield` keyword, which is what makes a generator
special. When calling `my_generator('thing')` instead of getting the result of
the function we get a generator object, which can be used anywhere you could
use a list or other iterable.

Most often, you'll use generators as part of a loop as below. The loop will
continue until the generator stops `yield`ing values.

<!--code lang=python linenums=true-->

    for value in my_generator('thing'):
        print value

    # first thing
    # second thing
    # third thing

    gen = my_generator('thing')
    next(gen)
    # 'first thing'
    next(gen)
    # 'second thing'
    next(gen)
    # 'third thing'
    next(gen)
    # raises StopIteration exception

After being instantiated, a generator doesn't do anything until it is asked for
a value. It will execute until the first `yield` and pass that value to the
caller, then wait (saving its state) until another value is requested.

Now let's make a generator that's a bit more useful than just giving back 3
hard-coded items. The classic generator example is an endless fibonacci
generator, so let's give that a try. It will start at 1 and give the sum of the
prior two numbers for as long as you ask it to.

<!--code lang=python linenums=true-->

    def fib_generator():
        a = 0
        b = 1
        while True:
            yield a
            a, b = b, a + b

A `while True` loop in a function would normally be something to avoid because
the function would never return, but for a generator it's fine as long as
there's a `yield` in the loop. We do need to be careful to have an end
condition when we use this generator, because it will happily add numbers
forever.

Now let's use our generator to calculate the first fibonacci number that's
greater than 10,000.

<!--code lang=python linenums=true-->

    min = 10000
    for number in fib_generator():
        if number > min:
            print(number, "is the first fibonacci number over", min)
            break

That was pretty easy, and we can make that number as large as we want and it
will still (eventually) come up with the first number larger than X in the
fibonacci sequence.

Let's try out a more practical example. Paginating APIs is common practice to
limit usage and avoid sending 50 megabytes of JSON (!!!) to a mobile device.
First, we'll define the API we're using and then we'll write a generator around
it to hide the paging from our code.

The API we're using is called Scream, a place where users can argue about
restaurants they've eaten at or want to eat at. Their API for searching is
pretty simple, and looks like this.

<!--code lang=markup linenums=true-->

    GET http://scream-about-food.com/search?q=coffee
    {
        "results": [
            {"name": "Coffee Spot",
             "screams": 99
            },
            {"name": "Corner Coffee",
             "screams": 403
            },
            {"name": "Coffee Moose",
             "screams": 31
            },
            {...}
        ]
        "more": true,
        "_next": "http://scream-about-food.com/search?q=coffee?p=2"
    }

Neat! They embedded the link to the next page in the API response so it'll be
extremely easy to get the next page when it's time. We can also leave off the
page number to just get the first page. To get the data, we'll use the
always-handy [requests][requests] library and wrap it in a generator to display
our search results.

The generator will handle pagination and have limited retry logic, and will
work something like:

1. Receive search term
1. Query the scream-about-food API
1. Try again if the API fails
1. Yield the results from the page it gets one at a time
1. Get the next page if there is one
1. Exit when there are no more results

Easy enough. To start with, we'll implement the generator without retries to
keep the code simple.

<!--code lang=python linenums=true-->

    import requests

    api_url = "http://scream-about-food.com/search?q={term}"

    def infinite_search(term):
        url = api_url.format(term)
        while True:
            data = requests.get(url).json()

            for place in data['results']:
                yield place

            # end if we've gone through all the results
            if not data['more']: break

            url = data['_next']

When you create a generator, you only need to pass in search terms and the
generator will build the query and get results as long as they exist. There are
(of course) some rough edges here. Exceptions aren't handled at all, and if the
API fails or returns unexpected JSON the generator will raise an exception.

Despite these rough spots, we can still use it to find out what number our
restaurant is in the search results for the term "coffee".

<!--code lang=python linenums=true-->

    # pass a number to start at as the second argument if you don't want
    # zero-indexing
    for number, result in enumerate(infinite_search("coffee"), 1):
        if result['name'] == "The Coffee Stain":
            print("Our restaurant, The Coffee Stain is number ", number)
            return
    print("Our restaurant, The Coffee Stain didnt't show up at all! :(")

The generator handles iterating over each page of search results, so all we
have to do is use the `enumerate` builtin from earlier in the article to keep
track of the number of results and print them when we find our shop.

As an exercise, go ahead and add a counter to the `infinite_search` generator
so we can write code like this instead.

<!--code lang=python linenums=true-->

    for result in infinite_search("coffee"):
        if result['name'] == "The Coffee Stain":
            print("Our restaurant, The Coffee Stain is number ", result['number'])
            return
    print("Our restaurant, The Coffee Stain didn't show up at all! :(")

If you write Python 3, you already use generators when you use the standard
library. Calls like `dict.items()` now return generators instead of lists. To
get this behavior in Python 2 `dict.iteritems()` was added, but isn't as
frequently used.

## Python 2 and 3 compatibility

Moving from Python 2 to Python 3 can be an undertaking for any codebase (or any
developer) but it's possible to write code that runs in both. Support for
Python 2.7 will continue until 2020, but it's unlikely that many new features
will be backported. For now, it's recommended to support Python 2.7 and 3+
unless it's feasible for you to drop Python 2 support entirely.

For a comprehensive guide on supporting both versions, see the
[Porting Python 2 Code](https://docs.python.org/3.5/howto/pyporting.html) guide
from python.org.

Let's look over the most common things you'll run into when trying to write
compatible code, and how to use `__future__` to work around them.

### print or print()

Just about every developer who has switched from Python 2 to 3 has typed the
wrong `print` statement. Fortunately, you can standardize on using print as a
function (Python 3 style) instead of a keyword by just importing
`print_function`.

<!--code lang=python linenums=true-->

    print "hello"  # Python 2
    print("hello") # Python 3

    from __future__ import print_function
    print("hello") # Python 2
    print("hello") # Python 3

### Divided Over Division

The default behavior for division in Python also changed between 2 and 3. In
Python 2, dividing integers would perform integer-only division, chopping off
any trailing decimals. This wasn't what most users expected, so it was changed
in Python 3 to use floating point division even when dividing integers.

<!--code lang=python linenums=true-->

    print(1 / 3) # Python 2
    # 0
    print(1 / 3) # Python 3
    # 0.3333333333333333
    print(1 // 3) # Python 3
    # 0

This sort of behavior change brings in a bunch of subtle bugs when writing code
to run in both major versions. Again, we're saved by the `__future__` module.
Importing `division` makes these behaviors identical in both versions.

<!--code lang=python linenums=true-->

    from __future__ import division
    print(1 / 3)# Python 2
    # 0.3333333333333333
    print(1 // 3)# Python 2
    # 0
    print(1 / 3) # Python 3
    # 0.3333333333333333
    print(1 // 3)# Python 3
    # 0

## Fin - Thanks for Reading

Thanks for reading, I hope you learned at least one thing. If you have
something to add (or correct, no writer is perfect) I'll be checking the
comments section frequently. If you enjoyed this article, you might want to
check out this one on [`list` and `dict` comprehensions][comprehensions] or
a more in-depth treatment of [Python 2 and 3][py2]

Thanks to commenters dalke (on HackerNews), György Kiss, mikemikemikemikemike,
Karl-Aksel Puulmann, Bartłomiej "furas" Burek, and Peter Venable for
finding errors and omissions in this article.

[comprehensions]: https://www.airpair.com/python/posts/python-comprehension-syntax
[requests]: http://docs.python-requests.org/en/latest/
[py2]: https://www.airpair.com/python/posts/python-2-vs-python-3
[counter]: https://docs.python.org/3.4/library/collections.html#collections.Counter
