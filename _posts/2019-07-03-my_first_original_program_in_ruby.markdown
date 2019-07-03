---
layout: post
title:      "My First Original Program in Ruby"
date:       2019-07-03 16:04:59 +0000
permalink:  my_first_original_program_in_ruby
---

**WELCOME**

Welcome to my typing program!  It's a simple practice program built to solidify my ruby skills developed so far in the Flatiron School.

**MOTIVATION:**

The Flatiron school pace is unyeilding, and I was beginning to feel like I was forgetting some of the earlier content in the Ruby Module.

My learning style is very much experiential - I gain the most from experimenting with concepts and techniques, mostly through the process of making something that's broken at first and then figuring out how to fix it.

To that end, I wanted to synthesize what I have so far learned through the Flatiron School's Ruby curriculum in the form of an original custom program that takes advantage of (most of) the programming skills taught so far (in this case, up to the OBJECT RELATIONSHIPS in the Object Oriented Ruby module).

WHAT IS IT?

It's a CLI typing game! Type out the provided strings and the program will let you know if you were successful or got an error.  (NOTE: The actual contents of the typing strings aren't too important, so I kept them simple for functionality's sake).

**HOW IT IS BUILT **

  (this will likely read as higgledy piggledy... I suggest you have a look at file.rb to see how the program is put  together)

The program is housed within a class 'Game', with each game being instantiated when the game file is run.

Options for the easy/medium/hard/custom game levels are held within a class hash (the custom hash can have arrays of strings added to it through the .save method, although this currently isn't configured in the game)

gets.strip prompts are used throughout the program to receive user input on how the game is to be configured.

The gets.strip inputs are mapped to a Key in the Key: value pairs of the class hash.  Options for the different keys are held within case operators.

The selected key is passed to an iterator that iterates over the corresponding value array.  

The iteration has a yield in it that performs the gets.strip for the user's answer and conducts an accuracy check against the array element that has yielded.

If the user wants to scramble the array order, an option is given that allows for a scrambler method to randomly reorder the selected array sequence.

**HOW TO RUN IT**

Simply clone the git repository at [https://github.com/jgrantps/Blog-Post-1](http://github.com/jgrantps/Blog-Post-1) to bring it into your local IDE.
Run the bin/game executable file to start the game.  Follow the prompts and have fun!

**LESSONS LEARNED:**

Building this program was REALLY refreshing.  It forced me to think creatively on my own, and it held me responsible for mapping out the entire build process.  Mapping out the process - breaking down the different steps to get a functioning game - was more rewarding than I thought.  I really enjoyed the sense of ownership over how the game would function, what the code would look like, and how the bugs were addressed.

Be careful of scope creep!  The program would have been 3 times bigger if I didn't force myself to say "it's fine the way it is".  Some things I would have liked to include would have been a parsing ability for more complicated strings (ie multiline strings), and the ability to upload your own array into the game through the gameplay prompts.

This build took me approx 1.75 days to complete.  It was time away from the curriculum, but I think it was exactly what I needed in order to start really feeling comfortable with the course material.  I'll definitely be doing this again.

