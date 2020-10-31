# isoduration: Operations with ISO 8601 durations.

[![PyPI Package](https://img.shields.io/pypi/v/isoduration?style=flat-square)](https://pypi.org/project/isoduration/)

## What is this.

ISO 8601 is most commonly known as a way to exchange datetimes in textual format. A
lesser known aspect of the standard is the representation of durations. They have a
shape similar to this:

```
P3Y6M4DT12H30M5S
```

This string represents a duration of 3 years, 6 months, 4 days, 12 hours, 30 minutes,
and 5 seconds.

The state of the art of ISO 8601 duration handling in Python is more or less limited to
what's offered by [`isodate`](https://pypi.org/project/isodate/). What we are trying to
achieve here is to address the shortcomings of `isodate` (as described in their own
[_Limitations_](https://github.com/gweis/isodate/#limitations) section), and a few of
our own annoyances with their interface, such as the lack of uniformity in their
handling of types, and the use of regular expressions for parsing.

## How to use it.

This package revolves around the [`Duration`](src/isoduration/types.py) type.

Given a ISO duration string we can produce such a type by using the `parse_duration()`
function:

```py
>>> from isoduration import parse_duration
>>> duration = parse_duration("P3Y6M4DT12H30M5S")
>>> duration.date
DateDuration(years=Decimal('3'), months=Decimal('6'), days=Decimal('4'), weeks=Decimal('0'))
>>> duration.time
TimeDuration(hours=Decimal('12'), minutes=Decimal('30'), seconds=Decimal('5'))
```

The `date` and `time` portions of the parsed duration are just regular
[dataclasses](https://docs.python.org/3/library/dataclasses.html), so their members can
be accessed in a non-surprising way.

Besides just parsing them, a number of additional operations are available:

- Durations can be compared and negated:
  ```py
  >>> parse_duration("P3Y4D") == parse_duration("P3Y4DT0H")
  True
  >>> -parse_duration("P3Y4D")
  Duration(DateDuration(years=Decimal('-3'), months=Decimal('0'), days=Decimal('-4'), weeks=Decimal('0')), TimeDuration(hours=Decimal('0'), minutes=Decimal('0'), seconds=Decimal('0')))
  ```
- Durations can be added to, or subtracted from, Python datetimes:
  ```py
  >>> from datetime import datetime
  >>> datetime(2020, 3, 15) + parse_duration("P2Y")
  datetime.datetime(2022, 3, 15, 0, 0)
  >>> datetime(2020, 3, 15) - parse_duration("P33Y1M4D")
  datetime.datetime(1987, 2, 11, 0, 0)
  ```
- Durations are hashable, so they can be used as dictionary keys or as part of sets.
- Durations can be formatted back to a ISO 8601-compliant duration string:
  ```py
  >>> from isoduration import parse_duration, format_duration
  >>> format_duration(parse_duration("P11YT2H"))
  'P11YT2H'
  >>> str(parse_duration("P11YT2H"))
  'P11YT2H'
  ```

## How to improve it.

These steps, in this order, should land you in a development environment:

```sh
git clone git@github.com:bolsote/isoduration.git
cd isoduration/
python -m venv ve
. ve/bin/activate
pip install -U pip
pip install -e .
pip install -r requirements/dev.txt
```

Adapt to your own likings and/or needs.

Testing is driven by [tox](https://tox.readthedocs.io). The output of `tox -l` and a
careful read of [tox.ini](tox.ini) should get you there.

## FAQs.

### How come `P1Y != P365D`?
Some years have 366 days. If it's not always the same, then it's not the same.

### Why do you create your own types, instead of somewhat shoehorning a `timedelta`?
`timedelta` cannot represent certain durations, such as those involving years or months.
Since it cannot represent all possible durations without dangerous arithmetic, then it
must not be the right type.

### Why don't you use regular expressions to parse duration strings?
[Regular expressions should only be used to parse regular languages.](https://stackoverflow.com/a/1732454)

### Why is parsing the inverse of formatting, but the converse is not true?
Because this wonderful representation is not unique.

### Why do you support `<insert here a weird case>`?
Probably because the standard made me to.

### Why do you not support `<insert here a weird case>`?
Probably because the standard doesn't allow me to.

### Why is it not possible to subtract a datetime from a duration?
I'm confused.

### Why should I use this over some other thing?
You shouldn't do what people on the Internet tell you to do.

### Why are ISO standards so strange?
Yes.

## References.

- [XML Schema Part 2: Datatypes, Appendix D](https://www.w3.org/TR/xmlschema-2/#isoformats):
  This excitingly named document contains more details about ISO 8601 than any human
  should be allowed to understand.
- [`isodate`](https://pypi.org/project/isodate/): The original implementation of ISO
  durations in Python. Worth a look. But ours is cooler.
