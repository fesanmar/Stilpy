.. |br| raw:: html

   <br />

=====
Usage
=====

To use Stilpy in a project:

>>> from stilpy import TimeGaps 

Minimal expample
-----------------

Suppose that you have serveral records of time stored in a list of dictionaries, like this:
:

>>> list_dict = [
...         {'t_dt':'start','dt': '2019-12-19 10:00:00'},
...         {'t_dt':'end', 'dt': '2019-12-19 13:30:20'},
...         {'t_dt':'start', 'dt': '2019-12-19 14:30:00'},
...         {'t_dt':'start', 'dt': '2019-12-19 15:30:00'},
...         {'t_dt':'end', 'dt': '2019-12-19 17:00:35'},
...         {'t_dt':'start', 'dt': '2019-12-19 09:00:00'}
...         ]

As you can see, this are time intervals. They have a start an an end point, but
the are not in the rigth order. The first two elements are allright. But then we
have to start points towether. And the last one y a start point interval that should
be on the top of the list. Wath Stilpy can do for us is to make an iterator with
those records, closing the start points with the rigth end, or ginving then an
unknown end y they don't have one of their own.

To do that we need to make an instance of the **TimeGaps** class.

**TimeGaps** recieves several parameters. Some of the are optionals. Let's see
wich of them we need for our example:

    **iterable:** 
        An iterable object that contains a list of items.
        Those items must be dicts or dictlike objects. Lists,
        tuples and objects with ``__dict__`` atribute are accepted as
        well. |br| Every item, must content itself the next items: |br| 
        1. A ``datetime`` object or a string format ``datetime``. In our
        case, we have the second option. |br|
        2. A item that defines if the first elemente that we just mentioned
        is an inicial or a final time point of a time interval.

    **tag_loc:** 
        It tells **TimeGaps** where to find the tag that tells if the
        item is a start point or an end.

    **i_tag:**
        The name of the initial time tag in the iterable. Default is ``'start'``.

    **f_tag:** 
        The name of the final time tag in the iterable. Default is ``'end'``.

    **dt_loc:**
        The location, inside each element of the iterable, of the
        ``datetime`` information. It can be a dictionary key or an index,
        depending on the collection.

The rest of parameters are optionals, and we're not needing them at the
moment.

Now we can call create the **TimeGaps** object, passing the arguments in 
order:

>>> ti = TimeGaps(list_dict, 't_dt', 'start', 'end', 'dt')

Now we have our iterator. Every item is a ``TimeInterval`` object with
some attributes like ``start``, ``end``, ``duration`` and ``is_perfect``.
You can add any other attribute to the ``TimeInterval`` objects, but we
are going to see it later. Now we are just goint to print each element. But
first we'll ask to our object for the sum or all their duration. At the
same time, we'll pass an argumen that will be returned if **TimeGaps** is
unable to make the sum because some interval hasn't a duration. By default
``None`` will be returned.

>>> ti.total_duration("there are some inperfect TimeInterval in your collection")
'there are some inperfect TimeInterval in your collection'
>>> for i in ti:
...     print(i)
TimeInterval(start=datetime.datetime(2019, 12, 19, 9, 0), end='', duration='')
TimeInterval(start=datetime.datetime(2019, 12, 19, 10, 0), end=datetime.datetime(2019, 12, 19, 13, 30, 20), duration=datetime.timedelta(seconds=12620))
TimeInterval(start=datetime.datetime(2019, 12, 19, 14, 30), end='', duration='')
TimeInterval(start=datetime.datetime(2019, 12, 19, 15, 30), end=datetime.datetime(2019, 12, 19, 17, 0, 35), duration=datetime.timedelta(seconds=5435))

You can see that, by default, an empty property is set to ``''``.

If we pass a perfect time intervals collection ``total_duration`` will return a
very different result. Let's see an example.

>>> perfect_list_dict = [
...         {'t_dt':'start','dt': '2019-12-19 10:00:00'},
...         {'t_dt':'end', 'dt': '2019-12-19 13:30:20'},
...         {'t_dt':'start', 'dt': '2019-12-19 14:30:00'},
...         {'t_dt':'end', 'dt': '2019-12-19 17:00:35'},
...         ]
>>> for t in ti_p:
...     print(t.duration)
3:30:20
2:30:35
>>> ti_p.total_duration()
datetime.timedelta(seconds=21655)
>>> print(ti_p.total_duration())
6:00:55

As you can see, this method returns a ``timedelta`` object with the sum
of the duration of every ``TimeInterval``.

Time intervals with groups
--------------------------

In the previous example, with just gotted records that need to be ordered
and putted together. But what happend if we have records that belong to
different groups, all together in the same collection. Well, for that we
have the ``group_by`` parameter.

Let's try another example.

Imagine we're working with the sign-in and sign-out of the employees from the 
company's web application. We should have something like this:

>>> keys_dicts = [
...     {
...         'name': 'Eve', 'surname': 'Palmer',
...         't_dt':'start', 'dt': '2019-12-19 10:00:00'
...     },
...     {
...         'name': 'Cecilia', 'surname': 'Park',
...         't_dt':'end', 'dt': '2019-12-19 11:00:05'
...     },
...     {
...         'name': 'Moses', 'surname': 'Farrel',
...         't_dt':'start', 'dt': '2019-12-19 10:00:05'
...     },
...     {
...         'name': 'Eve', 'surname': 'Palmer',
...         't_dt':'end', 'dt': '2019-12-19 13:30:20'
...     },
...     {
...         'name': 'Moses', 'surname': 'Farrel',
...         't_dt':'end', 'dt': '2019-12-19 13:45:15'
...     },
...     {
...         'name': 'Eve', 'surname': 'Palmer',
...         't_dt':'start', 'dt': '2019-12-19 14:30:00'
...     },
...     {
...         'name': 'Cecilia', 'surname': 'Park',
...         't_dt':'start', 'dt': '2019-12-19 15:30:00'
...     },
...     {
...         'name': 'Cecilia', 'surname': 'Park',
...         't_dt':'end', 'dt': '2019-12-19 17:00:35'
...     },
...     {
...         'name': 'Moses', 'surname': 'Farrel',
...         't_dt':'start', 'dt': '2019-12-19 09:00:00'
...     }
... ]

We can't just order these records according to their time value and their
condition of starting or ending point of a interval. If we do that, we'll
be ignoring that every record belongs to a different person. Son we have
to use the ``group_by`` parameter by saying wich keys should use **TimeGaps**
to order this records. Lets see how:

For our example we need to group the records by name and surname. ``group_by``
is a keyword argumen and it's expecting a single element or a collection,
preferred a tuple. So we do it like this:

>>> ti_g = TimeGaps(
...                     keys_dicts, 't_dt', 'start', 'end', 'dt',
...                     group_by=('name', 'surname')
...        )

But, adittionally maybe we want to store that pairs of keys and values 
of names and surnames inside of te ``TimeInterval`` objects, in order 
to differentiate some intervals from others. As we said before ``group_by`` 
is a keyword argumen. Any other positional argumen used to instanciate the 
``TimeGaps`` class different of ``iterable``, ``tag_loc``, ``i_tag``, ``f_tag``
and ``dt_loc`` we'll be treated as the key for creating the adittionall
attributes for the ``TimeInterval`` objects of a ``TimeGaps`` iterator.
So we can change the instanciation like this:

>>> ti_g = TimeGaps(
...                     keys_dicts, 't_dt', 'start', 'end', 'dt',
...                     'name', 'surname',
...                     group_by=('name', 'surname')
...        )

Now if we print every element we should see how the ``TimeIntervals`` has
being created by groups, and how they are ordered in the collection.

>>> for tg in ti_g:
...     print(tg)
TimeInterval(start=datetime.datetime(2019, 12, 19, 9, 0), end='', duration='', name='Moses', surname='Farrel')
TimeInterval(start=datetime.datetime(2019, 12, 19, 10, 0), end=datetime.datetime(2019, 12, 19, 13, 30, 20), duration=datetime.timedelta(seconds=12620), name='Eve', surname='Palmer')
TimeInterval(start=datetime.datetime(2019, 12, 19, 10, 0, 5), end=datetime.datetime(2019, 12, 19, 13, 45, 15), duration=datetime.timedelta(seconds=13510), name='Moses', surname='Farrel')
TimeInterval(start='', end=datetime.datetime(2019, 12, 19, 11, 0, 5), duration='', name='Cecilia', surname='Park')
TimeInterval(start=datetime.datetime(2019, 12, 19, 14, 30), end='', duration='', name='Eve', surname='Palmer')
TimeInterval(start=datetime.datetime(2019, 12, 19, 15, 30), end=datetime.datetime(2019, 12, 19, 17, 0, 35), duration=datetime.timedelta(seconds=5435), name='Cecilia', surname='Park')

But what happens if we want different iterators, one per element of the group.
Lets say that we want a iterator for every employee. You can easily have. In 
fact you will get a list of ``TimeGaps`` objects, one for every employee. You
just need to call the ``grouped_intervals`` property.

First lets see the groups that we have by calling the ``grouper_tags`` property.

>>> ti_g.grouper_tags
[{'name': 'Cecilia', 'surname': 'Park'}, {'name': 'Eve', 'surname': 'Palmer'}, {'name': 'Moses', 'surname': 'Farrel'}]

Now lets get a list of ``TimeGaps``, one per employee and see what it got
inside.

>>> grouped_ti = ti_g.grouped_intervals
>>> for group in grouped_ti:
...     print(group)
TimeGaps(TimeInterval(start='', end=datetime.datetime(2019, 12, 19, 11, 0, 5), duration='', name='Cecilia', surname='Park'), TimeInterval(start=datetime.datetime(2019, 12, 19, 15, 30), end=datetime.datetime(2019, 12, 19, 17, 0, 35), duration=datetime.timedelta(seconds=5435), name='Cecilia', surname='Park'))
TimeGaps(TimeInterval(start=datetime.datetime(2019, 12, 19, 10, 0), end=datetime.datetime(2019, 12, 19, 13, 30, 20), duration=datetime.timedelta(seconds=12620), name='Eve', surname='Palmer'), TimeInterval(start=datetime.datetime(2019, 12, 19, 14, 30), end='', duration='', name='Eve', surname='Palmer'))
TimeGaps(TimeInterval(start=datetime.datetime(2019, 12, 19, 9, 0), end='', duration='', name='Moses', surname='Farrel'), TimeInterval(start=datetime.datetime(2019, 12, 19, 10, 0, 5), end=datetime.datetime(2019, 12, 19, 13, 45, 15), duration=datetime.timedelta(seconds=13510), name='Moses', surname='Farrel'))

You can easily see that a ``TimeGaps`` iterator has being created for each 
employee with the same methods and properties as their ``TimeGaps`` object's
father. So, for example, you could call the ``total_duration`` method for each
``group`` in ``grouped_ti`` collection.
