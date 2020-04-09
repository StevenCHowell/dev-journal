## Thursday 9 April 2020
** Problem: How do I run a PostgreSQL database without requiring `sudo`?**

I want to run PostgreSQL for a local project with requiring `sudo`.

Suggested Solution:

```bash
export PATH=/usr/lib/postgresql/9.5/bin/:"$PATH"  #9.5 should be the version installed

export PGDATA="$PWD/pg_data"
export PGHOST="$PGDATA/sockets"
export PGDATABASE="postgres"
export PGUSER="$USER"

pg_ctl init

mkdir -p "$PGDATA/sockets"
echo "unix_socket_directories = 'sockets'" >> "$PGDATA/postgresql.conf"
echo "listen_addresses = ''" >> "$PGDATA/postgresql.conf"

pg_ctl -D pg_data -l pg.log estart
createdb <postgres_db_name> 
```

Then when done, shutdown postgres using 

```bash
pg_ctl  stop
```

More info: [PostgreSQL without sudo](https://datagrok.org/unix/a_little_postgres/)

*Tags: database, postgresql*

## Thursday 2 April 2020
**Problem: How do I interrupt a `multiprocessing.Pool` and clean-up?**

I am creating a Flask application that responds to certain requests by starting a new thread to perform the work. This new thread then creates a multiprocessing.Pool of workers to process multiple unrelated inputs in parallel (the output of one process has no impact on other processes).  I was tring to terminate the threadusing using `pool.terminate()` and then `pool.close()` without success. 

Suggested Solution:
I ended up solving this by using a `multiprocessing.Manager().Event()` to signal if each task should keep going or terminate.

Here is a minimal working example to demonstrate the issue.

### Worker file
```python
# sub.py
import logging
import multiprocessing
import psutil
import threading
import time

LOGGER = logging.getLogger(__name__)
N_CORES = psutil.cpu_count(logical=False)


class WorkerClass():
    def __init__(self, input_values):
        self.input_values = input_values
        self.thread = threading.Thread(target=self.run, args=())

    def run(self):
        LOGGER.debug(f'Starting pool of {N_CORES} processes')
        manager = multiprocessing.Manager()
        self.proceed = manager.Event()  # create an event flag
        self.proceed.set()
        with multiprocessing.Pool(processes=N_CORES) as pool:
            jobs = {}
            for i in self.input_values:
                jobs[i] = pool.apply_async(self.worker, [i, self.proceed])

            LOGGER.debug(f'{len(jobs)} workers started')

            done = []
            while len(done) < len(jobs):
                for key in jobs:
                    if key not in done:
                        try:
                            res = jobs[key].get(1)
                            LOGGER.debug(f'{key} result: {res}')
                            done.append(key)
                        except multiprocessing.TimeoutError:
                            LOGGER.debug(f'Job still running: {key}')
            LOGGER.debug('Collected all worker jobs.')
        LOGGER.debug('Multiprocessing pool closed.')

    @staticmethod
    def worker(i, proceed):
        result = 0
        while result < i**2 and proceed.is_set():  # check that the event is still set
            time.sleep(2)  # some long process
            result += i
        LOGGER.info(f'{i} * {i} = {result}')
        return result
```

```python
# main.py
import logging
import time

from sub import WorkerClass

logging.basicConfig(level=logging.DEBUG)
LOGGER = logging.getLogger()


if __name__ == '__main__':
    my_worker = WorkerClass(range(5))
    LOGGER.debug('starting thread')
    my_worker.thread.start()

    # simulate the request to terminate the job
    time.sleep(7)
    my_worker.proceed.clear()  # clear the event flag
 ```
 
More info: [My question](https://stackoverflow.com/questions/60996940/how-do-i-cleanly-interrupt-and-terminate-a-multiprocessing-pool-of-jobs), [GeeksforGeeks article](https://geeksforgeeks.org/python-different-ways-to-kill-a-thread/)

*Tags: parallel, threading, multiprocessing, API*

## Tuesday 7 Jan 2020
**Problem: print a directory tree with the number of lines of code in any text files**

Suggested Solution:
```python
import pathlib
import itertools

space = '    '
branch = '│   '
tee = '├── '
last = '└── '


def tree(dir_path: pathlib.Path, level: int=-1, n_wspace: int=50,
         limit_to_directories: bool=False, length_limit: int=1000):
    """Given a directory Path object print a visual tree structure"""
    dir_path = pathlib.Path(dir_path)  # accept string coerceable to Path
    files = 0
    directories = 0

    def inner(dir_path: pathlib.Path, prefix: str = '', level=-1):
        nonlocal files, directories
        if not level:
            return  # 0, stop iterating
        contents = [path for path in dir_path.iterdir() if path.name != '.git']
        pointers = [tee] * (len(contents) - 1) + [last]
        for pointer, path in zip(pointers, contents):
            if path.is_dir():
                yield prefix + pointer + path.name
                directories += 1
                extension = branch if pointer == tee else space
                yield from inner(path, prefix=prefix+extension, level=level-1)
            elif not limit_to_directories:
                info = prefix + pointer + path.name
                try:
                    with path.open('r') as f:
                        n_lines = len(f.readlines())
                    loc = f' LOC: {n_lines}'
                    info += ' ' * (n_wspace - len(info)) + loc
                except UnicodeDecodeError:
                    pass
                yield info
                files += 1

    print(dir_path.name)
    iterator = inner(dir_path, level=level)
    for line in itertools.islice(iterator, length_limit):
        print(line)
    if next(iterator, None):
        print(f'... length_limit, {length_limit}, reached, counted:')
    print(
        f'\n{directories} directories' +
        (f', {files} files' if files else '')
    )
```

Example output:
```
LaTeX_template
├── .gitignore       LOC: 276
├── appendix_a.tex   LOC: 8
├── clean.sh         LOC: 1
├── demo.bib         LOC: 247
├── demo.tex         LOC: 121
├── fancyhdr.sty     LOC: 236
├── figures
│   ├── redtriangle.png
│   └── yellowcircle.png
├── front_matter.tex LOC: 121
├── Makefile         LOC: 20
├── README.md        LOC: 28
└── section_1.tex    LOC: 34

1 directories, 12 files
```

More info: [List directory tree structure in Python](https://stackoverflow.com/a/59109706/3585557), [Count lines of code in directory using Python](https://stackoverflow.com/questions/38543709/count-lines-of-code-in-directory-using-python)

*Tags: LOC, tree, directory, code*

## Friday 3 Jan 2020
**Problem: create and interact with a separate thread for performing long tasks without locking up the controller**

Use the `threading` module.

Suggested Solution:
```python
>>> import threading
>>> import time

>>> def loop_from_i_to_j(i, j):
        for k in range(i, j):
            time.sleep(.5)
            print(f'~~{k}~~')
>>> def loop_over_alphabet():
        for letter in 'abcdefghijklmnopqrstuvwxyz':
                time.sleep(.1)
                print(letter)
>>> thread = threading.Thread(target=loop_from_i_to_j, args=(5, 10))
>>> thread.start(); loop_over_alphabet()
a
b
c
d
~~5~~
e
f
g
h
i
~~6~~
j
k
l
m
n
~~7~~
o
p
q
r
s
~~8~~
t
u
v
w
x
~~9~~
y
z
```

More info: [Python Programming WikiBook](https://en.wikibooks.org/wiki/Python_Programming/Threading)

*Tags: multithreading*

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
