# Workshop Series: Intermediate Python

_[UC Davis DataLab](https://datalab.ucdavis.edu/)_  
_Author & Maintainer: Nick Ulle <<naulle@ucdavis.edu>>_  

* [Reader](https://ucdavisdatalab.github.io/workshop_intermediate_python/)

These five standalone workshops aim to help Python users understand language
features, packages, and programming strategies that will enable them to write
more efficient code, be more productive when writing code, and debug code more
effectively. This is not an introduction to Python and is appropriate for
motivated intermediate to advanced users who want a better understanding of
working with Python for their research. Researchers from all domains at UC
Davis who meet the workshop prerequisites are welcome to enroll.

<!--
In order to satisfy the requirements for the GradPathways microcredential,
students must attend all four Intermediate Python workshops scheduled for April
18, May 2, May 16, and June 6, 2022.
-->

### Learning Objectives

See the reader for learning objectives.

### Prerequisites

Participants are expected to have taken DataLab’s [Python Basics workshop
series][python-basics] and/or have prior experience using Python, be
comfortable with basic Python syntax, and have it pre-installed and running on
their laptops.

[python-basics]: https://ucdavisdatalab.github.io/workshop_python_basics/

## Contributing

The course reader is a live webpage, hosted through GitHub, where you can enter
curriculum content and post it to a public-facing site for learners.

To make alterations to the reader:

1.  Run `git pull`, or if it's your first time contributing, see the
    [Setup](#setup) section of this document.

2.  Edit an existing chapter file or create a new one. Chapter files are
    Markdown files (`.md`) in the `chapters/` directory. Enter your text, code,
    and other information directly into the file. Make sure your file:

    - Follows the naming scheme `##_topic-of-chapter.md` (the only exception is
      `index.md`, which contains the reader's front page).
    - Begins with a first-level header (like `# This`). This will be the title
      of your chapter. Subsequent section headers should be second-level
      headers (like `## This`) or below.

    Put any supporting resources in `data/` or `img/`. For large files, see the
    [Large Files](#large-files) section of this document. You do not need to
    add resources generated by your code (such as plots). The next step saves
    these in `docs/` automatically.

3.  Run the command `jupyter-book build .` in a shell at the top level of the
    repo to regenerate the HTML files in the `_build/`.

4.  When you're finished, `git add`:
    - Any files you edited directly
    - Any supporting media you added to `img/`
    - The `.gitattributes` file (if you added a large file)

    Then `git commit` and `git push`. This updates the `main` branch of the
    repo, which contains source materials for the web page (but not the web
    page itself).

5.  Run the command `ghp-import -n -p -f _build/html` in a shell at the top
    level of the repo to update the `gh-pages` branch of the repo. This uses
    the [`ghp-import` Python package][ghp-import], which you will need to
    install first (`pip install ghp-import`). The live web page will update
    automatically after 1-10 minutes.

[ghp-import]: https://github.com/c-w/ghp-import


## Setup

### Python Packages

We recommend using [conda][] to manage Python dependencies. The `env.yaml` file
in this repo contains a list of packages necessary to build the reader. You can
create a new conda environment with all of the packages listed in that file
with this shell command:

```sh
conda env create --file env.yaml
```

[conda]: https://docs.conda.io/en/latest/
