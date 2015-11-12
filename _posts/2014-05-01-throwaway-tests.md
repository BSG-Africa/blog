---
title: Throwaway Tests
---

A while back I had to write a PowerShell script. The script was not particularly complicated, it needed
to watch a directory and perform an action when specific set of files existed. As I started adding
functionality to the script, it became increasingly harder to test. Each time I wanted to run my script,
I had to delete and recreate the whole folder structure, and then monitor the output.
<!--more-->

My aversion to doing anything manually led me to search for a testing framework for PowerShell. I
very quickly dismissed all the search results as way too much effort for a few lines of script. I instead
decided to write quick set-up and tear-down scripts to create and delete my directory structure.
Things were automated, and I was happy. For a while.

My next irritation was that I needed to constantly scan the output of the script to check if it had run
correctly. I decided to write a very primitive `assertEquals` function at the bottom of the file and in 5
minutes I had a string of tests covering my script.

What I had discovered was an often overlooked advantage of using tests to drive out code: A small
and quick feedback loop for testing and running individual parts of the code. These tests will not be
run as part of any build, and are not likely to be used to refactor my 20-something lines of script, but
they definitely helped me write this code in a much more pain-free way.
