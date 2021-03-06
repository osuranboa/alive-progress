![alive!](https://raw.githubusercontent.com/rsalmei/alive-progress/master/img/alive.gif)

# alive-progress :)
## An animated and smart Progress Bar for python!

[![PyPI version](https://img.shields.io/pypi/v/alive-progress.svg)](https://pypi.python.org/pypi/alive-progress/)
[![PyPI pyversions](https://img.shields.io/pypi/pyversions/alive-progress.svg)](https://pypi.python.org/pypi/alive-progress/)
[![PyPI status](https://img.shields.io/pypi/status/alive-progress.svg)](https://pypi.python.org/pypi/alive-progress/)

Ever found yourself in a remote ssh session, doing some lengthy operations, and every now and then you feel the need to hit [enter] just to ensure you didn't lose the connection? Ever wondered where your processing is in, and when will it finish? Ever needed to *pause* the progress bar for a while, return to the python REPL for a manual inspection or fixing an item, and then *resume* the process like it never happened? I did...

I've made this cool progress bar thinking about all that, the Alive-Progress bar! :)

![alive-progress](https://raw.githubusercontent.com/rsalmei/alive-progress/master/img/main.gif)


I like to think of it as a new kind of progress bar for python, as it has among other things:
  - a cool live spinner, which makes it clear the process did not hang and your terminal/connection is healthy;
  - a visual feedback of the current speed/throughput, as the spinner runs faster or slower according to the actual processing speed;
  - an efficient multi-threaded bar, which updates itself at a fraction of the actual speed (1,000,000 iterations per second equates to roughly 60fps refresh rate) to keep CPU usage low and avoid terminal spamming; (📌 new: you can now calibrate this!)
  - an expected time of arrival (eta), that shows the remaining processing time in a friendly way, not anything like `eta: 1584s`, it will nicely show `eta: 0:26:24` as you would expect (but anything less than a minute is indeed `eta: 42s`);
  - a `print()` hook, which allows print statements in the midst of an alive-bar context, nicely cleaning the output of any garbage, and even enriching with the current count when it occurred;
  - after your processing has finished, a nice receipt is printed with the statistics of that run, including the elapsed time and observed throughput;
  - it tracks the actual count in regard of the expected count, so it will look different if you send in more or less than expected;
  - it automatically detects if there's really an allocated tty, and if there isn't, only the final receipt is printed, so you can safely include the alive-bar in all and any code and rest assure your log file won't get 60fps garbage;
  - you can pause the alive-bar! I think that's an unprecedented feature for a progress-bar, it's incredible to orchestrate operations that require manual interaction on some items;
  - it is customizable, with a growing smorgasbord of different bar and spinner styles, as well as several factories to easily generate yours!


## Get it

Just install with pip:

```bash
$ pip install alive-progress
```


## Awake it

Open a context manager like this:

```python
from alive_progress import alive_bar
items = range(1000)                  # retrieve your set of items
with alive_bar(len(items)) as bar:   # declare your expected total
    for item in items:               # iterate as usual
        # process each item
        bar()                        # call after consuming one item
```

And it's alive! 👏

In general lines, just retrieve the items, enter the `alive_bar()` context manager, and iterate/process normally, calling `bar()` once per item.


### Understand it

- the `items` can be any iterable, and usually will be some queryset;
- the first argument of the `alive_bar` is the expected total, so it can be anything that returns an integer, like `qs.count()` for querysets, `len(items)` for iterables that support it, or even a static integer;
- the `bar()` call is what makes the bar go forward -- you usually call it in every iteration after consuming an item, but you can get creative! For example you could call it only when you find something you want, or call it more than once in the same iteration, depending on what you want to monitor. Just adjust the total accordingly to get a useful eta;
- the `bar()` call also returns the current count/percentage if needed, and enables to pass situational messages to the bar.

So, you could even use it without any loops, like:

```python
with alive_bar(3) as bar:
    corpus = read_big_file()
    bar('file read, tokenizing')
    tokens = tokenize(corpus)
    bar('tokens ok, processing')
    process(tokens)
    bar()
```


## Alive-Bar modes

Actually, the `total` argument is optional. Providing it makes the bar enter the **definite mode**, the one used for well-bounded tasks. This mode has all statistics widgets the alive-bar has to offer: counter, throughput and eta.
This is the only mode that tracks and displays overflow and underflow of items.

If you do not provide a `total`, the bar enters the **unknown mode**. In this mode, the whole progress-bar is animated like the cool spinners, as it's not possible to determine the percentage of completion. Therefore it's also not possible to compute an eta, but you still get the counter and throughput widgets (note the cool spinners are still present in this mode, and each animation runs independently of each other, rendering a unique show in your terminal 😜).

Then you have the (📌 new) **manual modes**, where you get to manually control the bar!
Just pass a `manual=True` argument to `alive_bar()` or `config_handler.set_global()`, and you get to send a percentage (a float between 0 and 1) to the `bar()` handler, and put the alive-bar in wherever position you want! Call it as frequently as you need, the refresh rate will be asynchronously computed according to the sent progress and the actual elapsed time.

In these modes, you can also provide the `total` to enter the **manual definite mode**, and get all the same counter, throughput and eta statistics widgets as the _auto definite mode_. The counter is inferred from the supplied percentage.
Or you can omit the `total` to enter the **manual unknown mode**, where it's not possible to infer neither the counter nor the throughput widgets, and you get a simpler "percent/second" (%/s) and a rough eta, calculated to get to 100%.

So, to summarize it all:

| mode | automatic | total | percentage | counter | throughput | eta | outstanding features |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| definite        | ✅ | ✅ | ✅ | ✅            | ✅           | ✅ | overflow and underflow |
| unknown         | ✅ | ❌ | ❌ | ✅            | ✅           | ❌ | super cool animations |
| manual definite | ❌ | ✅ | ✅ | ✅ (inferred) | ✅           | ✅ | choose any completion |
| manual unknown  | ❌ | ❌ | ✅ | ❌            | ⚠️ (simpler) | ✅ | choose any completion |

> Just remember: You do not have to think about which mode you should be using, just pass a `total` if you have one, and use `manual` if your processing needs it! 👏


### Signatures of the `bar()` handler

- in **automatic** modes: `bar(text=None, incr=1)` ➔ increases the current count (by any positive increment), optionally setting the situational text message, and returns the new count;
- in **manual** modes: `bar(perc=None, text=None)` ➔ sets the new progress percentage, optionally setting the situational text message, and returns the new percentage.


## Styles

Wondering what styles does it have bundled? It's `showtime`! ;)

![alive-progress spinner styles](https://raw.githubusercontent.com/rsalmei/alive-progress/master/img/showtime-spinners.gif)

I've made these styles to test all combinations of parameters of the factories, but I think some of them ended up very very cool! Use them, or create your own!

There's also a bars `showtime`, check it out! ;)

![alive-progress bar styles](https://raw.githubusercontent.com/rsalmei/alive-progress/master/img/showtime-bars.gif)


## Printing messages

While in an alive progress bar context, you have two ways to output messages:
  - calling `bar(text='message')`, which sets/overwrites a situational message within the bar line, usually to display something about the items being processed, or the phase the processing is in;
  - calling the usual Python `print('message')`, which will print an enriched message that includes the current position of the alive bar, thus leaving behind a log and continuing the bar below it.

Both methods work the same in **definite**, **unknown** and **manual** modes, and always clear the line appropriately to remove any garbage on screen.

![alive-progress messages](https://raw.githubusercontent.com/rsalmei/alive-progress/master/img/printing.gif)


## Configuration

There are several options to customize behavior, both globally and per use! Just use the `config_handler` object!

```python
from alive_progress import config_handler
config_handler.set_global(...)
```

The options are:
- `length`: number of characters to render the animated progress bar
- `spinner`: spinner name in alive_progress.SPINNERS or custom
- `bar`: bar name in alive_progress.BARS or custom
- `unknown`: spinner name in alive_progress.SPINNERS or custom
- `theme`: theme name in alive_progress.THEMES
- `force_tty`: runs animations even without a tty (pycharm terminal for example)
- `manual`: set to manually control percentage

And you can mix and match them, global and local! (_Click to see it in motion_)

[![alive-progress customization](https://asciinema.org/a/j392MaLz1w0zDw6EVHw4QbLAO.svg)](https://asciinema.org/a/260882)


## Advanced

### Calibration (📌 new)

The Alive-Bar has a cool visual feedback of the current throughput, so you can instantly see how fast your processing is, as the spinner runs faster or slower with it.
For this to happen, I've put together and implemented a few fps curves to empirically find which one gave the best feel of speed:

![alive-bar fps curves](https://raw.githubusercontent.com/rsalmei/alive-progress/master/img/alive-bar_fps.png)
(interactive version [here](https://www.desmos.com/calculator/ema05elsux))

The graph shows the logarithmic (red), parabolic (blue) and linear (green) curves, as well as an adjusted logarithmic curve (dotted orange), with a few twists for small numbers. I've settled with the adjusted logarithmic curve, as it seemed to provide the best all around perceived speed changes.

The default Alive-Bar calibrations are _1,000,000_ in auto (and manual definite) modes and 1 (100%) in manual unknown mode. Both enable a vast operating range and generally work well.

But let's say your processing hardly gets to 20 items per second, and you think the Alive-Bar is rendering sluggish, you can:

```python
    with alive_bar(total, calibrate=20) as bar:
        ...
``` 

And it will be running waaaay faster... :)
Adjust the calibration to your liking!


### Create your own animations

Make your own spinners and bars! All of the major components are individually customizable!

There's builtin support for a plethora of special effects, like frames, scrolling, bouncing, delayed and compound spinners! Get creative!

These animations are made by very advanced generators, defined by factories of factory methods: the first level receives and process the styling parameters to create the actual factory; this factory then receives operating parameters like screen length, to build the infinite animation generators.

These generators are stateful, they can render the next frame and yield, until a full animation cycle is complete. But one generator is capable of several animation cycles, when one are exhausted, this allows the next cycle to start. This has all kinds of cool implications! The cycles can have different animation sizes, different screen lengths, they do not need to be synchronized, they can create long different sequences by themselves, they can cooperate with each other to play cycles in sequence or simultaneously, and I can display several at once on screen without any interferences!

The types I've made are:
- `frames`: draw any sequence of characters, that will be played frame by frame in sequence;
- `scrolling`: pick a frame or a sequence of characters and make them flow smoothly from side to side, hiding behind or wrapping upon the invisible borders; supports several cycles of distinct characters;
- `bouncing`: aggregates two `scrolling` in opposing directions, to make two frames or two sequences of characters flow interleaved from/to each side, hiding or immediately bouncing upon the invisible borders; supports several interleaved cycles;
- `delayed`: get any other animation generator, and copy it multiple times, skipping some frames in the process! very cool effects are made here;
- `compound` get a handful of generators and play them side by side simultaneously! why choose if you can have them all?

A small example (_Click to see it in motion_)

[![alive-progress creative](https://asciinema.org/a/mK9rbzLC1xkMRfRDk5QJMy8xc.svg)](https://asciinema.org/a/260884)


### The Pause mechanism

To use the pause mechanism, you must wrap the alive-bar inside a function generator to yield the objects you want to interact with. The `bar` handler includes another context manager to yield anything you want, just do `with bar.pause(): yield obj`.

Suppose you need to reconcile transactions. You need to iterate over thousands of them, detect somehow the faulty ones, and fix them. They could be broken or not synced or invalid or anything else, several different problems. Typically you would have to let the process run, appending to a list each inconsistency found, and waiting, potentially a long time, until the end to be able to do anything. You could mitigate this by processing in chunks, but that has its own shortcomings.

With the Alive-Progress bar and the Pause mechanism, you can inspect these transactions in **real-time**! You wait only until the next one is found! To use it you would do something like this:

```python
def reconcile_transactions():
    qs = Transaction.objects.filter()  # django example, or in sqlalchemy: session.query(Transaction).filter()
    with alive_bar(qs.count()) as bar:
        for transaction in qs:
            if not validate(transaction):
                with bar.pause():
                    yield transaction
            bar()
```

That's it! Then you could use it in ipython (or your preferred _REPL_)! Just call the function to instantiate the generator and `next()` with it. The progress bar will run as usual, but as soon as an inconsistency is found, the bar pauses itself and you get the prompt back! 😃

```text
In [11]: gen = reconcile_transactions()

In [12]: next(gen, None)
|█████████████████████                   | 105/200 [52%] in 5s (18.8/s, eta: 4s)
Out[12]: Transaction<#123>
```

How cool is that?! You have the transaction to debug and fix any way you want, and when you're done, continue the process with the same `next` as before... The bar returns like nothing happened!! :)

```text
In [21]: next(gen, None)
|█████████████████████                   | ▁▃▅ 105/200 [52%] in 5s (18.8/s, eta: 4s)
```


### Forcing animations on non-interactive consoles (like Pycharm's)

Pycharm's python console for instance do not report itself as "interactive", so I've included a `force_tty` argument to be able to use the alive-progress bar in it.

So, just start it as:

```python
with alive_bar(1000, force_tty=True) as bar:
    for i in range(1000):
        time.sleep(.01)
        bar()
```

You can also set it system-wide in `config_handler`.

Do note that this console is heavily instrumented and has more overhead, so the outcome may not be as fluid as you would expect.


## To do

- ~~create an unknown mode for bars (without a known total and eta)~~
- ~~implement a pausing mechanism~~
- ~~change spinner styles~~
- ~~change bar styles~~
- ~~include a global configuration system~~
- ~~create generators for scrolling, bouncing, delayed and compound spinners~~
- ~~create an exhibition of spinners and bars, to see them all in motion~~
- ~~include theme support in configuration~~
- include colors in spinners and bars
- try some adaptive algorithm for ETA, like moving average, exponential smoothing or Kalman Filter
- any other ideas welcome!


## Interesting facts

- This whole project was implemented in functional style;
- It does not declare even a single class;
- It uses extensively (and very creatively) python _Closures_ and _Generators_, they're in almost all modules (look for instance the [spinners generators and spinner_player](https://github.com/rsalmei/alive-progress/blob/master/alive_progress/spinners.py) 😜);
- It does not have any dependencies.


## Changelog highlights:
- 1.3.0: new fps calibration system, support force_tty and manual options in global configuration, multiple increment support in bar handler
- 1.2.0: filled blanks bar styles, clean underflow representation of filled blanks
- 1.1.1: optional percentage in manual mode
- 1.1.0: new manual mode
- 1.0.1: pycharm console support with force_tty, improve compatibility with python stdio streams
- 1.0.0: first public release, already very complete and mature


## License
This software is licensed under the MIT License. See the LICENSE file in the top distribution directory for the full license text.


## Did you like it?

Thank you for your interest!

I've put much ❤️ and effort into this.

Hope you like it.
