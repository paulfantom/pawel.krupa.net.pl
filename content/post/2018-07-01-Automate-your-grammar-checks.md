---
title: "Automate your grammar checks"
description: "Continuous Integration for a spoken language"
thumbnailImagePosition: left
thumbnailImage: https://www.nichemarket.co.za/wp-content/uploads/2018/03/spell-check-805x452.jpg
date: 2018-07-01
tags:
  - documentation
  - ci
---

# Continuous Integration for a spoken language

I wanted to start a blog, so like a lot of people nowadays I've created a [GitHub repository](https://github.com/paulfantom/pawel.krupa.net.pl),
added automatic deployment via [netlify](https://www.netlify.com/blog/2016/09/29/a-step-by-step-guide-deploying-on-netlify/), and voila!
But I still needed to write something... And while doing so it would be nice to use proper spelling,
so why not adapt some kind of automatic spell checker?

## Aspell? Hunspell? *spell?

To begin with it would be nice to find a CLI tool which could spell check plain-text files. A minute with google reveals that most frequently
used spell checker tools are **Aspell** and **Hunspell**. They are similar in use and there are lots of articles on how to use them
([1](https://opensource.com/article/18/2/how-check-spelling-linux-command-line-aspell), [2](https://alexwlchan.net/2016/09/please-use-aspell/)).
One problem is that they are primarily used in interactive mode and I would like to do spell checking in non-interactive mode though and that 
produces [not human-friendly output](https://github.com/hunspell/hunspell/blob/master/docs/hunspell.1.md#pipe-mode). Nonetheless I decided to
give Hunspell a go and tune it up a bit with bash to improve readability of the output.

### Friendly Hunspell?

To run Hunspell in non-interactive mode just pass text through a pipe to Hunspell program executed with `-a` flag, so simple
`cat some_file | hunspell -a` will do the trick. However there is one downside to this. By default Hunspell will report its status on every
line of passed text with a variety of single characters like `*`, `&`, `@`, `+`, `-`, `#`, or 
[others](https://github.com/hunspell/hunspell/blob/master/docs/hunspell.1.md#pipe-mode) and I'm interested only in lines starting with `&` or `#`.
I also like to have some colors there. So let's parse and colorize Hunspell! To do this I came up with script below. It iterates over every
file with `.md` extension and runs Hunspell on that file. Next it sorts output, removes duplicate lines and ones starting with `*` or `@(#)`.
After that it parses Hunspell output to create human readable output with some colors. Unfortunately I didn't found how to get Hunspell to return
different exit code than 0, so the script looks for phrase "Misspelled word" in output and returns 1 if such phrase was detected.

```bash
#!/bin/bash

RET=0

exec 5>&1
for file in $(find . -name "*.md"); do
    echo -e "\n\e[33mChecking $file\e[0m"
    OUT=$(cat $file | \
        hunspell -a -d en_US 2>/dev/null | \
        sort --ignore-case | uniq | \
        grep -v '^\*$\|@(#).*' | \
        sed -r "s/^[&#]\s([[:alpha:]]+)\s([0-9]+)\s([0-9]+):/$(printf '\033[0m')Misspelled word $(printf '\033[31m\033[1m')\1$(printf '\033[0m') in line \2 position \3, possible alternatives:$(printf '\033[36m')/g" | \
        tee >(cat - >&5))
    echo -en "\e[0m"
    if [ "$(echo $OUT | grep 'Misspelled word' | wc -l)" -gt 0 ]; then
        RET=1
    fi
done

exit $RET
```

### Not so markdown friendly Hunspell

After running above script on some markdown files with embedded code (like this blog post) it was clear that I need some other tool. Script
produced many more lines which were part of code section and weren't important, for example:
```
& ' 15 29: e, s, i, a, n, r, t, o, l, c, d, u, g, m, p
& ' 15 70: e, s, i, a, n, r, t, o, l, c, d, u, g, m, p
& 'Misspelled 3 29: misspelled, dispelled, misspell
```
Time to get back to the drawing board.

## Developers lingo

Maybe I started it wrong? I essentially need a tool written by developers for developers not to "spell check" my blog
posts, but to "lint" my "code". And since my "code" looks more like a GitHub read-me or a documentation, I should
probably look for a "documentation linter". Quick google search and here I found a Codeship post about
[improving documentation by automating spelling and grammar checks](https://blog.codeship.com/improve-documentation-by-automating-spelling-and-grammar-checks/).
Hell yeah! That is exactly what I want to do! That blog post describes how to use an awesome tool called
["markdown-spellcheck"](https://www.npmjs.com/package/markdown-spellcheck) which was created to run spell checks on
markdown files (duh!) using Hunspell and it even supports non-interactive mode! After running simple
`npm install markdown-spellcheck -g` and `mdspell --ignore-acronyms --en-us --report "content/*/*.md"` I've got a
nicely formatted output:

![mdspell example output](/images/20180701-mdspell-example.png)

Since not every word can be in dictionary, `mdspell` can use `.spelling` file to store exceptions. Write a word in this
file and `mdspell` won't report it as an error.

Codeship also describes how to use a tool called ["write good"](https://www.npmjs.com/package/write-good) which promotes
better writing style by being a 
["naive linter for English prose for developers who can't write good and wanna learn to do other stuff good too"](https://github.com/btford/write-good#write-good-). 
And since I can't write good yet, it looks like an ideal tool for me. Couple of commands later and I have additional
advices on how I shouldn't write:

![write good example output](/images/20180701-write-good-example.png)

## [Automate!](https://memegenerator.net/img/instances/65228817/automate.jpg)

Since I want to use GitHub PR system for new articles and other changes, I can use tools which are available in
[GitHub marketplace](https://github.com/marketplace). This time the choice fell on Travis CI, partially because I've
been using it for a long time. Quick look into
[Travis documentation about using JavaScript](https://docs.travis-ci.com/user/languages/javascript-with-nodejs/) and 
here is a `.travis.yml` file with everything needed.

```yaml
---
language: node_js
node_js:
  - "lts/*"
cache:
  directories:
    - "node_modules"
before_install:
  - npm i -g markdown-spellcheck write-good
jobs:
  include:
    - script: 'mdspell --ignore-acronyms --en-us --report content/*/*.md'
    - script: 'write-good --yes-eprime --parse content/*/*.md || true'
braches:
  only:
    - master
```

This also uses new Travis feature of build stages, runs checks in parallel, and caches NPM installation directory, so
everything should be fast. I have also changed `write-good` execution a bit, so it will do more checks, report it in
more compact format, and will always return true (`write-good` doesn't have a mechanism for exceptions).

## Sum up

No one likes to go through blog posts or even worse - documentations when there are plenty of grammar mistakes. We got 
used to the red zigzag line under misspelled words. But who writes markdown in LibreOffice Writer or Microsoft Word?
We tend to use same editors for markdown and for code, so why not adapt same techniques? Why not lint documentation?
As you can see - it's really simple!
