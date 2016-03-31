# blog

Before You Start: Create Author Page
------------------------------------
Before writing a post, first create an author page for yourself and place it in
the `_authors` folder. This will put your display name on all your posts on the
home page and on the post itself. It will also create a page for you at
`/authors/authorname/`

Add the following content:

~~~
---
author: authorname
display_name: Your Name
gravatar: gravatar-hash
---

A short paragraph or two about yourself.
~~~

To generate your gravatar hash, first register on gravatar.com, then run
`echo -n your.email@example.com | md5sum`

  1. Trim leading and trailing whitespace from an email address
  2. Force all characters to lower-case
  3. md5 hash the final string

More details are available on Gravatar's
[page](http://en.gravatar.com/site/implement/hash/).

Writing Posts
-------------
Ensure that you first create an author page.

Posts go in the `_posts` folder and need to be named `YYYY-MM-DD-<whatever>.(html|md)`

The files need to have a FrontMatter section setting the title and author of the post (and maybe other stuff as well, later on).
E.g.

~~~
---
title: The title of the post
author: authorname
---
~~~

Adding `<!--more-->` into the text will treat the preceding text as the excerpt to be used on the overview page.

Please use h4 (`####` in markdown) by default for section headers.

Installing Jekyll Locally
-------------------------
The following assumes you are using Windows

 1. Download and install ruby from [here](http://rubyinstaller.org/downloads/) (~17mb)
 2. Download and extract the DevKit from the same site (~42mb). In that folder run `ruby dk.rb init` and then `ruby dk.rb install` to include DevKit in the PATH. You need this for redcarpet.
 3.  Install Jekyll `gem install jekyll`. If you need to go through a proxy you will need to add the -p switch after the install command. make sure your proxy url starts with http://. If you additionally need to ignore ssl errors, create the file %USERPROFILE%\\.gemrc and add the following line: `:ssl_verify_mode: 0`
~~ 4.  Install redcarpet `gem install redcarpet`using the proxy switch again if you need to.~~
 4.  Install rouge `gem install rouge`using the proxy switch again if you need to.
 5.  Now you can preview the site by navigating to the directory of the blog and running `jekyll serve`

 Github forces us to switch to kramdown/rouge for markdown parsing and syntax-highlighting. Not sure what that means for windows users yet. Might make all the redcarpet/devkit stuff obsolete.
