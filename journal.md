## Day Date

Question...

Suggested Solution:

...

More Info: [link]()

*Tags: ...*

## Wednesday 19 April 2023

How do I create optional arguments using Python's built-in `argparse` library?

Suggested Solution:

```python
parser.add_argument(
    "-o",
    "--optional",
    default="default_value",
    type=str,
    help="Change the value of optional.",
)
```

More Info: [Meduim post with lots of examples](https://medium.com/swlh/python-argparse-by-example-a530eb55ced9)

*Tags: #argparse, #optional, #argument

## Monday 27 February 2023

How do I launch a flatpak application from the command line?

Suggested Solution:

Use this `fuzzpak` script:

```bash
#!/usr/bin/env bash
#GPLv3

DIR=${DIR:-$HOME/.var/app}
CMD=${CMD:-flatpak run}

launch_app() {
    find "${DIR}" -mindepth 1 -maxdepth 1 \
     -type d -iname "*$1*" -printf '%f\n' \
    | xargs $CMD
}

# parse opts
while [ True ]; do
if [ "$1" = "--help" -o "$1" = "-h" ]; then
    echo " "
    echo "$0 [OPTIONS]"
    echo "--directory, -d   Location of flatpaks (default: $HOME/.var/app"
    echo " "
    exit
elif [ "$1" = "--directory" -o "$1" = "-d" ]; then
    DIR=$DIR
    shift 2
else
    break
fi
done

# main
launch_app "${1}"
```

More Info: [Seth Kenlon's blog post](https://www.redhat.com/sysadmin/launch-flatpaks-terminal-fuzzpak), see also [Seth's `pakrat` script for creating flatpak aliases](https://gitlab.com/slackermedia/pakrat) 

*Tags: #CLI, #flatpak*

## Wednesday 7 September 2022

What is a simple way to convert a...

* snake case string into camel case?
* camel case string into snake case?

Suggested Solution:

### Snake to Camel

As a one-liner:

```python
In [14]: snake_string = "steven_c_howell"
    ...: camel_string = snake_string.title().replace("_", "")

In [15]: print(camel_string)
StevenCHowell
```

As a function:

```python
def snake2camel(snake_string):
    return snake_string.title().replace("_", "")
```

### Camel to Snake

Use regular expressions (for simple cases):

```python
In [16]: import re

In [17]: camel_string = "StevenCHowell"
    ...: snake_string = re.sub(r'(?<!^)(?=[A-Z])', '_', name).lower()

In [18]: print(snake_string) 
steven_c_howell
```

More Info: [StackOverflow Snake to Camel example](https://stackoverflow.com/a/34655712/3585557), [StackOverflow Camel to Snake example (includes code for more complicated cases, such as `"getHTTPResponseCode"`](https://stackoverflow.com/a/1176023/3585557)

*Tags: #snake_case, #CamelCase*

## Friday 2 Septemebr 2022

I am combining `.gitignore` files from two projects.  How do I deduplicate the resulting combined file?

Suggested Solution:

* Generate a new `.gitignore` file: https://www.toptal.com/developers/gitignore/
* Combine the two, then use the following Python code to remove duplicates (not perfect)

```python
import pathlib
fname_in = pathlib.Path('.gitignore')
fname_out = pathlib.Path('.gitignore_dedup')

lines_seen = set()  
enter = '\n'
with open(fname_out, 'w') as outfile:
    for line in open(fname_in, 'r'):
        if line[:1] == '#' or line == '\n':
            outfile.write(line)
        elif line not in lines_seen:
            outfile.write(line)
            lines_seen.add(line)
```

More info: [Stackoverflow post on removing duplicate lines from a text file](https://stackoverflow.com/a/1215244/3585557)

*Tags: #gitignore, #deduplicate*

## Monday 8 August 2022

Why do my Python logging messages do not print when the logging level is set to `info`?

Suggested Solution:

In more recent versions of Python 3 (tested with Python 3.8), console logging requires creating a console handler to correctly show info messages.

The following example is modified from the [Configuring Logging example](https://docs.python.org/3.8/howto/logging.html#configuring-logging) in the Python documentation:

```python
import logging

# create logger
logger = logging.getLogger('__name__')
level = logging.INFO
logger.setLevel(level)

# ----> console info messages require these lines <----
# create console handler and set level to debug
ch = logging.StreamHandler()
ch.setLevel(level)

# add ch to logger
logger.addHandler(ch)
# -----------------------------------------------------

# 'application' code
logger.debug('debug message')
logger.info('info message')
logger.warning('warn message')
logger.error('error message')
logger.critical('critical message')
```
Running the above code generates the following output:

```bash
info message
warn message
error message
critical message
```

Here is the same code without the console handler:

```python
import logging

# create logger
logger = logging.getLogger('__name__')
level = logging.INFO
logger.setLevel(level)

# 'application' code
logger.debug('debug message')
logger.info('info message')
logger.warning('warn message')
logger.error('error message')
logger.critical('critical message')
```

Without the console handler, the output does not include the info message.

```bash
warn message
error message
critical message
```

I do not understand why it is necessary to include the console handler; it seems unnecessary.

More Info: [Python Logging Documentation](https://docs.python.org/3.8/howto/logging.html#configuring-logging)

*Tags: #Logging*

## Wednesday 6 July 2022

How do I convert a Python object that contains nested objects into a json serializable dictionary?

Suggested Solution:

Provide a function to for the `default` input of the `json.dumps()` or `json.dump()` methods.  This input value is [described as follows](https://docs.python.org/3/library/json.html#json.dump):
    
> If specified, default should be a function that gets called for objects that can’t otherwise be serialized. It should return a JSON encodable version of the object or raise a TypeError. If not specified, TypeError is raised.

For class objects, a function that returns the `__dict__` attribute works nicely.  This requires that the nested class attributes be serializable themselves, which may not be the case for some custom data structures, e.g., NumPy arrays or Pandas DataFrames.

Here is a function for converting a class object into a dictionary:

```python
import json

def to_dict(obj):
    return json.loads(json.dumps(obj, default=lambda o: o.__dict__))
```

Here is an example that directly saves a class object to a file:

```python
import json

def to_json(obj, fname):
    with open(fname, 'w') as f:
        json.dump(obj, f, default=lambda o: o.__dict__)
```

More info: [StackOverflow post]([https://towardsdatascience.com/4-pre-commit-plugins-to-automate-code-reviewing-and-formatting-in-python-c80c6d2e9f5](https://stackoverflow.com/a/48413290/3585557)

*Tags: #Object-Oriented, #class, #dict, #json*

## Wednesday 25 May 2022

How do I setup pre-commit hooks to automate Python style rules?

Suggested Solution:

Install pre-commit, black, flake8, and isort using `pip`:
    
    pip install pre-commit black flake8 isort

or using [Poetry](https://python-poetry.org/docs/master/managing-dependencies/#adding-a-dependency-to-a-group):

    poetry add pre-commit black flake8 isort --group dev

In the repo root directory, create (or modify) the file `.pre-commit-config.yaml` with the following:

```yaml
repos:
-   repo: https://github.com/psf/black
    rev: 22.3.0
    hooks:
    - id: black
-   repo: https://github.com/pycqa/flake8
    rev: 4.0.1
    hooks:
    - id: flake8
-   repo: https://github.com/timothycrosley/isort
    rev: 5.10.1
    hooks:
    -   id: isort
```

Add the following configurations to `tox.ini`:

```ini
[flake8]
max-line-length = 88
select = C,E,F,W,B,B950
extend-ignore = E203, E501

[isort]
profile = black
line_length = 88
```

The last step is to install the pre-commit hooks by running:

```bash
pre-commit install
```

More info: [This blog post](https://towardsdatascience.com/4-pre-commit-plugins-to-automate-code-reviewing-and-formatting-in-python-c80c6d2e9f5) provided much of the information for this solution.  The docstring hook did not apply for my use case but would be useful in many situations.  [This second blog post](https://python.plainenglish.io/how-to-improve-your-python-code-style-with-pre-commit-hooks-e7fe3fd43bfa) had additional information about built-in pre-commit hooks ([full list here](https://pre-commit.com/hooks.html))

*Tags: #Git, #code-style, #PEP8, #black, #flake8, #isort*

## Monday 18 October 2021

How do I create a MS COCO dataset from a selection of images from Google Open Images Dataset?

Suggested Solution:

The [FiftyOne application](https://voxel51.com/fiftyone/), from [VOXEL51](https://voxel51.com/), provides a simple SDK for interfacing with various datasets and data formats.  FiftyOne can easily be used to download a selection of data from the Google Open Images Dataset then stored in MS COCO format as outlined below.

```python
import pathlib
import fiftyone as fo
import fiftyone.zoo as foz

# select the classes of interest
classes = ["Handgun", "Dagger"]

# load in data from Google Open Images Dataset
dataset = foz.load_zoo_dataset(
    "open-images-v6",
    split="train",
    label_types=["segmentations"],
    classes=classes,
    max_samples=100,
)

# export the data in COCO MS format
export_dir = 'coco_version/'
dataset.export(
    export_dir=export_dir,
    dataset_type=fo.types.COCODetectionDataset,
    label_field="segmentations",
    classes=classes,  # only output information about the classes of interest
)    
```

More info: This solution combines information from the [FiftyOne Integration guide for Open Images Integration](https://voxel51.com/docs/fiftyone/integrations/open_images.html) and a section of the [FiftyOne User Guide on Exporting FiftyOne Datasets](https://voxel51.com/docs/fiftyone/user_guide/export_datasets.html#cocodetectiondataset).  Huge thanks to [Eric Hofesmann](https://twitter.com/ehofesmann) for helping me connect the dots.

*Tags: #object-detection, #COCO, #OID, #FiftyOne, #image*

## Wednesday 21 April 2021

How do I convert an 3-channel RGB image to a 1-channel B&W image?

Suggested Solution:

```python
def color_to_bw(data_c, rec='709-6'):
    weights = {
        '709-6': np.array([0.2125, 0.7174, 0.0721]),  # Rec. 709-6 luminance weights
        '609-7': np.array([0.299, 0.587, 0.114]),  # Rec. 609-7 luminance weights
    }
    data_bw = np.dot(data_c, weights[rec])
    return np.expand_dims(data_bw, -1)
```

More info: My solution is a functionalized version of Example 3 from [this Holy Python post](https://holypython.com/python-pil-tutorial/how-to-convert-an-image-to-black-white-in-python-pil/#:~:text=Color()%20method%20to%20get,turn%20in%20a%20grayscale%20image.).

*Tags: #image, #color, #b&w*

## Monday 25 January 2021

**Problem: How do I remove duplicate columns from a pandas DataFrame?**

In working with a data file I indentified several columns had identical information. Sometimes the columns had similar names, `heading` and `heading:1`, but not always.

Suggested Solution:

```python
def find_duplicate_cols(data):
    duplicate_columns = set()
    columns = list(data.columns)
    n = len(columns)
    for i, col in enumerate(columns):
        col_i = data[col]
        for j in range(i+1, n):
            col_j = data[columns[j]]
            if col_i.equals(col_j):
                duplicate_columns.add(columns[j])
    return list(duplicate_columns)
```

More info: My solution is a modified version of code from [thispointer.com article](https://thispointer.com/how-to-find-drop-duplicate-columns-in-a-dataframe-python-pandas/#:~:text=To%20find%20these%20duplicate%20columns,stored%20in%20duplicate%20column%20list.)

*Tags: #pandas, #duplicate, #columns*

## Monday 26 October 2020

**Problem: How do I make a `venv` Virtual Environment visible to Jupyter installed in another environment?**

In addition to installing the `ipykernel` module (common to conda environments),

    pip install ipykernel
    
the environment path must be added to the appropriate config file using,

    python -m ipykernel install --user --name=<helpful-kernel-name>
    
This should print something like the following:

    Installed kernelspec tabular_env in C:\Users\user\AppData\Roaming\jupyter\kernels\tabular_env
    
or 

    Installed kernelspec myenv in /home/user/.local/share/jupyter/kernels/myenv

More info: [Using Virtual Environments in Jupyter Notebook and Python](https://janakiev.com/blog/jupyter-virtual-envs/)

*Tags: #jupyter, #virtual-environments, #envs, #venv*

## Thursday 27 August 2020

**Problem: How can I pass inputs into the [callback function for a Bokeh widget](https://docs.bokeh.org/en/latest/docs/user_guide/interaction/widgets.html#callbacks)?**

The Bokeh widgets are designed to take only specific inputs.  Widgets `.on_change` method require the function signature `(attr, old, new)`.  Similarly, widgets `.on_click` method require the function signature `(new)`.  I want to create widgets groups that interact with each other. To avoid using on a host of global variables I need to modify the input signature of the callbacks to accept additional inputs.

Suggested Solution:

Wrap the functions in `functools.partial` to provide the additional inputs.  Here is an example Bokeh app that create a button group using a class that contains the callback methods and does not require global variables.

```python
from functools import partial
from bokeh import models, plotting
from bokeh.layouts import row, column


class OptionFilter():
    def __init__(self, options):
        button_select = models.Button(label='Select All', width=100,
                                      button_type='primary')
        button_clear = models.Button(label='Clear All', width=100, height=30,
                                     button_type='primary')
        cb_group = models.CheckboxGroup(labels=options,
                                        active=list(range(len(options))))
        info = models.Paragraph(text="""Push the button.""",
                                width=200, height=100)

        button_select.on_click(partial(self.select_all, cb_group, info))
        button_clear.on_click(partial(self.clear_all, cb_group, info))
        cb_group.on_change('active', partial(self.cb_changed,
                                             cb_group=cb_group, info=info))

        self.layout = column(row(button_select, button_clear),
                             cb_group, info)

    @staticmethod
    def select_all(cb_group, info):
        info.text = 'Selecting all.'
        n = len(cb_group.labels)
        cb_group.active = list(range(n))

    @staticmethod
    def clear_all(cb_group, info):
        info.text = 'Clearing all.'
        cb_group.active = []

    @staticmethod
    def cb_changed(attr, old, new, cb_group=None, info=None):
        if info is not None and cb_group is not None:
            info.text = f'Active: {cb_group.active}'


options1 = ['A', 'B', 'C', 'D']
group1 = OptionFilter(options1)

options2 = [str(i) for i in range(5)]
group2 = OptionFilter(options2)

layout = row(group1.layout, group2.layout)
plotting.curdoc().add_root(layout)
plotting.curdoc().title = 'Testing'
```

More info: [Python Bokeh send additional parameters to widget event handler](https://stackoverflow.com/q/41926478/3585557)

*Tags: #bokeh, #dashboard, #class*

## Friday 19 June 2020

**Problem: How can I create a dictionary/json map of an HDF5 file?**

I want to explore a completely unfamiliar HDF5 file and create a representation of its contents that can help in making use of the file.

Suggested Solution:

The following recursive function takes as input an HDF5 file pointer, or a HDF5 You can pass a file pointer to an HDF5 file the following recursive function to get a dictionary.

```python
def hdf_to_dict(group):
    d_group = {}
    for key, value in zip(group.keys(), group.values()):
        if 'group' in str(type(value)):
            d_group[key] = hdf_to_dict(value)
        else:
            d_group[key] = str(type(value)).split('.')[-2]
    return d_group
```

Here is an example using the above function.

```python
import pathlib
import json

fname_h5 = pathlib.Path('some_file.hdf5')
with f5py.File(fname_h5, 'r') as f_in:
    with open(fname1.with_suffix('.json'), 'w') as f_out:
        f_out.write(json.dumps(hdf_to_dict(f_in), indent=4))
```

As a note, you can simply print out all the element names using the following.

```python
with f5py.File(fname_h5, 'r') as f_in:
    f_in.visit(lambda name: print(name))
```

More info: [h5py docs](http://docs.h5py.org/en/stable/quick.html), [Blog Post: How to use HDF5 files in Python](https://www.pythonforthelab.com/blog/how-to-use-hdf5-files-in-python/)

*Tags: #hdf5, #h5, #json, #data-exploration*

## Thursday 9 April 2020

**Problem: How do I run a PostgreSQL database without requiring `sudo`?**

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

*Tags: #database, #postgresql*

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

*Tags: #parallel, #threading, #multiprocessing, #API*

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

*Tags: #LOC, #tree, #directory, #code*

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

*Tags: #multithreading*

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

*Tags: #multiple-inheritance, #object-oriented*

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

*Tags: #regular-expression, #string*

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

*Tags: #int, #float, #string*
