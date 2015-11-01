---
layout: post
title: Prime Number Calculations
---

I started a little java project to play around with some of the ideas I'd learned while doing
my degree in Computing and Statistics at the Open University.     The project is at 
[Github](https://github.com/thomasbridge74/PrimeNumbers) - no github pages
for the project though.

It's been interesting - I started off with a basic class using 32 bit int types and calculating
the factors of those.   I then added a (very crude) GUI app on top of it.    (One of the things I still want
to do is to publish the GUI app so it can be used on other hosts.   While I'm sure this is easy, 
I just haven't done it yet).

Then I started playing around with some of the [Project Euler](https://projecteuler.net/) stuff - one
of which was to calculate a prime number of a number too large to be held in an int.    I then recoded the
whole project to use the long type - and also to include some of the optimisations suggested in the file.
But then I started to run into the issue that just by keyboard
bashing numbers, it could take a very long time to calculate the prime number even with the optimisations. This seemed a
good opportunity for me to try and mess around with threading options - using one thread to do the calculations and the \
other to monitor the calculation process.

In the time I took to do all that - the number I had keyboard bashed - 1784216523597212 - was still 
going through the calculation process.   The next thing I need to do is to work out how to run a 
thread in the GUI which automatically updates the window current number being tested as a factor.   This is relatively
easy to do from the command line - doing it in the GUI will be a little more impressive in my view.


