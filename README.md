# File ML  v 1.1 - RFC

This document suggests a standard of convenient reading and manipulating data in a file of any extension.

<!-- todo TOC (when QurDocs ready) -->

## TL;DR

HTML inside comments of *any* files. Enables:

* semantically structuring file content
* finding pieces of file content
* manipulating (editing, deleting, moving, swapping, etc.) pieces of file content

```js
// file.js
// <comment-out when="in_production">
console.log(healthStatus, loadTime)
// </comment-out>
```

```bash
#!/bin/bash
# <section author="John Doe" description="does foo">
...
# </section>

# <section author="Nurbol Alpysbayev" description="checks quux">
...
# </section>
```

## Keywords

Keywords MUST, MUST NOT, SHOULD, SHOULD NOT, MAY have the meaning described [here](https://www.ietf.org/rfc/rfc2119.txt).

## Target Audience

Software developers writing in any programming languages.

## Application Areas

Writing to and reading from files, especially shared ones.

## Scope of this Document

The document only describes possible syntax and use cases. Following subjects are not covered:

* implementing a parser/interpreter (but links to existing ones are [provided](#implementations)).

## Possible Benefits

* easier writing of data;
* easy extracting of data without using regular expressions;
* easy removing of data (e.g. when uninstalling software from an OS, when uninstalling dependencies from a package etc.);
* possibility of as easy navigating in a file contents as in directories;
* easier analyzing of the shared files (multiple parties can write to them), i.e. it is easy to know which parties have already written their data to the file.
  
## The Problem

```shell
# .git/hooks/pre-commit (or any other shared file, e.g. .bashrc, .profile, etc.)

# the code added by library A
some_command_from_library_A

# the code added by library B
some_command_from_library_B
```

In the code above we see that there are two pieces added by different parties: libraries A and B.

While it is trivial to write to the file, it is not so when editing or removing the data is required. Usually developers go with wrapping data with identifying blocks and using regex solution:

```shell
# .git/hooks/pre-commit

# [A lib's code starts]
some_command_from_library_A
# [A lib's code ends]

#B_start
some_command_from_library_B
#B_end
```

The libraries A and B can, with the help of regular expressions, find their data, and remove it. You can see, that there is no consistency in style. However, it is only matter of style/aesthetics, right? Doesn't seem so. The above example makes it hard to do the following:

* Removing data. Other than having to use regular expressions, there is a theoretical possibility of removing excessive parts of the file if one of two things happens: either the developer used non-unique wrapping identifiers, or his regex was not made correctly. The developer has to spend his time to think about these cases to prevent them, which means, not only more precious time is spent, but also, the human factor is still there.
* Get all parties, who inserted their code, find out if a competing/collaborating library's code already presented in the file.
* Sort code blocks of different parties (sometimes, in rare cases, may appear to be a useful feature).
* Probably most important, navigating the file. It is absolutely possible to navigate directories. This is probably one of the earliest feature available in programming history. There are plenty of ways to do that, in any language. However, files can still become very big, containing many discrete concerns. Yet there is no way to get a structured representation of a file. Files and directories are only data structure concepts, they both contain bits of data, and having the last and the main data container (a file) unstructured seems not right.

## The Solution

There should be a consistent style for placing a data from different parties/vendors to a shared file.

```html
# <section id="A">
some_command_from_library_A
# </section>

# <section id="B">
some_command_from_library_B
# </section>
```

The above code allows to implement a tool for enabling features listed previously.

### More Examples

#### Multiple File Sections of one vendor

```html
# <section id="A">
some_command_from_library_A
# </section>

# <section id="B">
some_command_from_library_B
# </section>

++ # <section id="B: other_command">
++ other_command_from_library_A
++ # </section>

```

#### Nested File Sections

```html
# <section id="A">
some_command_from_library_A
# </section>

# <section id="B">
some_command_from_library_B

++    # <section id="B:other_command">
++    other_command_from_library_B
++    # </section>

# </section>

```

_Note:_ `B:other_command` is not some special syntax, it is written so only to avoid collisions with other `id`s.

## Specification

1. **NAMING**

    1.1. `<section>` and `</section>` constructs are each called a _**tag**_ (the opening and the closing one, respectively).

    1.2. The whole system of File ML tags of a file MUST be called _**FDOM**_ (File DOM) to differentiate from _**DOM**_ of web browsers. Continuing the logic, a file can also MAY be called a _**document**_ in the context of analyzing and manipulating file contents.

2. **SYNTAX**

    2.1. **TAGS**

    2.1.1. Any tags can be used, but default one SHOULD be `<section>`. This tag MUST be used when interoperability of different parties is required, unless when all the parties know name of another tag to be used.

    _Default and preferred for shared files_
    ```html
    # <section id="Library_X">
    ...
    # </section>
    ```

    _Ok for files used by one-party:_
    ```html
    # <my-component id="Library_X">
    ...
    # </my-component>
    ```

    2.1.2. The closing tag MUST always be present and MUST have exactly one slash before its name.

    2.1.3. The tags MUST be written as inline comments of the file type of the file in which they are located.

    _Example for a bash script (# used):_

    ```bash

    # <section id="bash_example">
    echo "i am written in bash"
    # </section>
    ```

    _Example for a javascript file  (// used):_
    ```js
    // <section id="javascript_example">
    console.log("i am written in javascript")
    // </section>
    ```

    2.2. **ATTRIBUTE NAMES**

    2.2.1. Names of attributes MUST follow this regex: `[a-z][a-z\-]*[a-z]`.

    _valid_:

    ```html
    # <section valid-name=abc_def>
    ...
    # </section>
    ```
    _invalid_:

    ```html
    # <section invalid_name=abc-def>
    ...
    # </section>

    # <section name123=abc-def>
    ...
    # </section>

    # <section 123name=abc-def>
    ...
    # </section>

    # <section $name=abc-def>
    ...
    # </section>
    ```

    2.2.2. Any attribute names can be used unless they do not comply the specified regex.

    2.2.3. If a tag has an `id` attribute, the said attribute's value MUST be unique in the file.

    2.3. **ATTRIBUTE VALUES**

    2.3.1. Values of attributes MUST be wrapped by double quotes:
    ```html
    # <section info="the quotes are mandatory">
    ...
    # </section>
    ```

    2.4. **NESTING**

    2.4.1. Tags MAY be nested in each other:

    ```html
    # <section vendor="abc">
        # <section id="abc:def">
        echo "i am written in bash"
        # </section>
    # </section>
    ```

    2.5. **COMMENTING**

    2.5.1. Commenting starts with `<!--` and ends with `-->`

    ```js

    // <!-- <section info="this MUST not be parsed by FML interpreter"> -->

    // <!-- </section> -->

    ```

    2.6. **ACCESSING FDOM**

    2.6.1. Tools that expose APIs to access/manipulate FDOM MUST use the same naming that is used in the web browsers' DOM standard.

    ```js
    document.getElementById('id-of-Files-ML-document') // correct

    file.elemById('id-of-Files-ML-document') // wrong
    ```

## General guideline

Syntax, implementations (parsers, lexers, interpreters), style should follow those of HTML and DOM as much as possible. If the Specification did not describe a subject, the description of the subject MUST be looked for in the HTML or DOM standard.

## HTML: differences and similarities

### Different: Visual vs Semantic

**HTML** is used heavily for layouting and styling information, hence it has many tags for layouting (`<header>`, `<ul>`, `<p>`) and visually decorating it (`<strong>`, `<small>`).

**File ML** hardly can be imagined to be used for the same purpose. Instead, it can be used to semantically (not visually) structure information (the `<section>` tag is good for it). If you are doubting what does it mean to "structure" information, you can think of it as splitting monolithic, multi-concern text (code) to smaller parts that have their own, focused concern.

### Similar: used to manipulate elements

Both **HTML** and **File ML** can be used to manipulate (edit, move, sort, delete, etc.) elements of DOM/FDOM. However, the next section should be read.

### Different: browser vs disk

**HTML** lives in browser, and every manipulation we make with it, is managed by browser.

**File ML** "lives" on disk. Every manipulation we make is written to the disk. This may require care and responsibility from **File ML** coders, especially when the file is shared among multiple parties.

## Other possible usages

File ML surely can be imagined to be used not only inside comments of programming languages, but also:

* inside strings
* in plain text files
* et. al.

## Implementations

Javascript: TODO

## Status of this Document

This document follows the [Semantic Versioning](https://semver.org/) model. Please look at the version (in the main title) of this document to find out status information.
