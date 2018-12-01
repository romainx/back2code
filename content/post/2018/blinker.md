---
title: Blinker
date: '2018-04-27'
tags: [dev]
---

I’m using [Pelican](https://blog.getpelican.com/) for [another blog](http://www.aubonroman.com) dedicated to books--no one is perfect. For several needs--an mainly because I’m a nerd--I have developed several plugins. And I have discovered that the Pelican plugin mechanism is based on a small framework called [Blinker](https://pythonhosted.org/blinker/).

> Blinker provides fast & simple object-to-object and broadcast signaling for Python objects.

It provides a way to **communicate between objects through signals** (a kind of event). I really found this way of working handy and elegant so I decided to have a closer look and to talk a bit about it.

It can be installed using pip: `$ pip install blinker`
To demonstrate its usage I’ve made a completely dumb example:

*A number generator sending signals and methods written to listen to these signals in order to print some information.*

```python
# The only import needed
from blinker import signal

# Signals definition
number_generator_number = signal('number_generator_number', doc='Return a generated number')
number_generator_start = signal('number_generator_start', doc='The number generator has started')
number_generator_end = signal('number_generator_end', doc='The number generator has ended')

class NumberGenerator():
    """A dumb number generator"""
    
    def __init__(self, name, bound):
      self.name = name
      self.bound = bound
      self.counter = 0

    def generate(self):
        # Sending a simple signal
        number_generator_start.send(self)
        while self.counter < self.bound:
            # Sending a signal with an associated data (the number generated)
            number_generator_number.send(self, number=self.counter)
            yield self.counter
            self.counter += 1
        # Informing that the generation has ended
        number_generator_end.send(self)
    
    def __repr__(self):
        return self.name

# Connecting with a decorator a different method for the each signal
@number_generator_start.connect
def start_generation(sender):
    print('{sender} started'.format(sender=sender))

@number_generator_number.connect
def print_number(sender, number):
    print('Got [{number}] from {sender}'.format(number=number, sender=sender))

@number_generator_end.connect
def end_generation(sender):
    print('{sender} ended'.format(sender=sender))

# Creating a generator
number_generator = NumberGenerator('Simple Generator', 5)

# Summing values
print('Sum is {result}'.format(result=sum(number_generator.generate())))
```

And the result is.

```bash
Simple Generator started
Got [0] from Simple Generator
Got [1] from Simple Generator
Got [2] from Simple Generator
Got [3] from Simple Generator
Got [4] from Simple Generator
Simple Generator ended
Sum is 10
```

The main advantage is its **simplicity**. It permits to build features with a **loose coupling between objects** and to act as a **kind of interface to permit further developments**. For that reason it’s particularly useful if you want to offer **plugin mechanism or if you build something like a parser**--Pelican is the perfect example.