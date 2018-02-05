# Guidelines for Correctness Through Types (Accompanying Slides)

This repository contains a markdown file (`slides.md`) that contains all the
slides for my presentation on "Guidelines for Correctness Through Types".

## Building the slides

A pre-built version of this markdown file is available in the releases
directory.

In order to generate the HTML yourself, you'll need to download
[https://revealjs.com/](https://revealjs.com/) and have
[pandoc](https://pandoc.org/) and
[inliner](https://www.npmjs.com/package/inliner) installed as well (the latter
is for inlining all JS and CSS into a single HTML file).

Assuming you have pandoc and inliner, you can run the following from the
top-level of this repository.

    # Download reveal.js
    wget https://github.com/hakimel/reveal.js/archive/master.tar.gz
    tar -xzvf master.tar.gz
    mv reveal.js-master reveal.js

    pandoc -s --mathjax -i -t revealjs slides.md -o slides_noninlined.html -V revealjs-url=./reveal.js
    inliner slides_noninlined.html > slides.html

This should result in a `slides.html` that you can view in a web browser.
