## Friday 3 Jan 2020
**Problem: using Flask, return JSON output together with a status code**

You could return results and the status code as `(result, code)` tuple.  In previous versions of Flask (maybe 1.0 and earlier), the `result` had to be JSON, so many references use `jsonify(result)` to convert the result dictionary to JSON.As of v1.1, you can now simply return the `result` as a dictionary.

Suggested Solution:
```python
@app.route('/path/<string:some_value>)
def perform_task(some_value):
    res = append_value_to_task(some_value)
    return {'result': res}, 201
```

More info: [SO q1](https://stackoverflow.com/a/59580439/3585557)

## Monday 30 Dec 2019
**Problem: call `__init__` methods of multiple super classes with different arguments**

It creates a problem when a child class inherits from multiple parent classes that have different argument expectations.  This problem arose from a method inheriting from both `QThread` and another class I had defined.

Suggested Solution:
```python
# class GuiConverter(QThread, Converter):  # fail
class GuiConverter(Converter, QThread):
    def __init___(self, arg1, arg2, arg3):
        # super.__init__(arg1, arg2, arg3)  # fail
        Converter.__init__(self, arg1, arg2, arg3)  # requires extra args
        QThread.__init__(self)  # requires no extra args
```

More info: [SO q1](https://stackoverflow.com/questions/34884567/python-multiple-inheritance-passing-arguments-to-constructors-using-super), [SO q2](https://stackoverflow.com/questions/9575409/calling-parent-class-init-with-multiple-inheritance-whats-the-right-way), [SO q3](https://stackoverflow.com/questions/3277367/how-does-pythons-super-work-with-multiple-inheritance), [Real Python article](https://realpython.com/python-super/#a-super-deep-dive)

*Tags: multiple inheritance, object-oriented*

## Wednesday 6 Nov 2019
**Problem: search strings using regular expressions**

Suggested Solutions:
```python
import re

# define regular expression prior to search
INTEGER = re.compile(r'\d+')
CHAR = re.compile('[a-z]+', re.I)  # re.I makes this case independent
FLOAT = re.compile(r'\d+.\d+')
SNAKE_FLOAT = re.compile(r'\d+_\d+')

some_str = 'I code at least 3 hours 5 days a week.'

first_match = INTEGER.search(some_str)
if first_match:
    print(first_match.group())
    
all_matches = INTEGER.findall(some_str)
for match in all_matches:
    print(match) 
    
# define regular expression during search
all_words = re.findall('[a-z]+', some_str, re.I)
for word in all_words:
    print(word)
```

More info: [Python Regular Expressions Complete Tutorial](https://www.guru99.com/python-regular-expressions-complete-tutorial.html)

*Tags: regular expression, string*

## Wednesday 6 Nov 2019
**Problem: attempt to convert string to int or at least float**

Suggested Solution:
```python
def int_else_float(rep_string):
    try:
        rep_float = float(rep_string)
        rep_int = int(rep_float)
        return rep_int if rep_int == rep_float else rep_float
    except ValueError:
        u_minus = u'\u2212'
        if u_minus in rep_string:
            # replace unicode minus
            return int_else_float(rep_string.replace(u'\u2212', '-'))
        else:
            return rep_string
            
vals = [
    '234no',
    '87.023',
    '1.2987e-7',
    u'1.2987e\u22127',
    '666',
]
for val in vals:
    res = int_else_float(val)
    print(f'{type(res)}: {val}')
    
# <class 'str'>: 234no, 234no
# <class 'float'>: 87.023, 87.023
# <class 'float'>: 1.2987e-7, 1.2987e-07
# <class 'float'>: 1.2987e−7, 1.2987e-07
# <class 'int'>: 666, 666 
```
I forget where I found this, maybe SO...

*Tags: int, float, string*
