# Moving Beyond Defensive Programming (Accompanying Slides)

This repository contains a markdown file (`slides.md`) that contains all the
slides for my presentation on "Moving Beyond Defensive Programming".

A version of this talk was given at the [2018 NE Scala
conference](http://www.nescala.org/) (in particular v0.2.0).

## Building the slides

A pre-built version of this markdown file is available in the releases
directory (e.g.
[this](https://github.com/changlinli/types_presentation_slides/releases/download/v0.2.0/slides.html)).

In order to generate the HTML yourself, you'll need to download
[https://revealjs.com/](https://revealjs.com/) and have
[pandoc](https://pandoc.org/) and
[inliner](https://www.npmjs.com/package/inliner) installed as well. The latter
is for inlining all JS and CSS into a single HTML file so that you don't need
any web connection to view the slides. If you don't require that, you can leave
out the `inliner` depedency and command.

Assuming you have pandoc and inliner, you can run the following from the
top-level of this repository.

    # Download reveal.js
    wget https://github.com/hakimel/reveal.js/archive/master.tar.gz
    tar -xzvf master.tar.gz
    mv reveal.js-master reveal.js

    pandoc -s --mathjax -i -t revealjs slides.md -o slides_noninlined.html -V revealjs-url=./reveal.js
    inliner -m slides_noninlined.html > slides.html

This should result in a `slides.html` that you can view in a web browser.
