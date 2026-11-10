# Beautiful Soup 4 (BS4): Comprehensive Python Web Scraping Tutorial

> A practical, reference-style guide to parsing HTML/XML, navigating the
> parse tree, finding elements, extracting data, modifying markup, and
> building robust scraping workflows.
>
> **Library:** Beautiful Soup 4 (`bs4`)\
> **Language:** Python 3\
> **Documentation checked:** 11 November 2026\
> **Official documentation:**
> https://www.crummy.com/software/BeautifulSoup/bs4/doc/

------------------------------------------------------------------------

## Table of contents

1.  [What is Beautiful Soup?](#1-what-is-beautiful-soup)
2.  [Installation and dependencies](#2-installation-and-dependencies)
3.  [A mental model: HTML becomes a
    tree](#3-a-mental-model-html-becomes-a-tree)
4.  [The four important object
    types](#4-the-four-important-object-types)
5.  [Parsing HTML and XML](#5-parsing-html-and-xml)
6.  [Inspecting a parsed document](#6-inspecting-a-parsed-document)
7.  [Extracting text: `text`, `string`, and
    `get_text()`](#7-extracting-text-text-string-and-get_text)
8.  [Attributes and tag properties](#8-attributes-and-tag-properties)
9.  [Navigating the parse tree](#9-navigating-the-parse-tree)
10. [Searching with `find()` and
    `find_all()`](#10-searching-with-find-and-find_all)
11. [Searching with CSS selectors: `select()` and
    `select_one()`](#11-searching-with-css-selectors-select-and-select_one)
12. [The complete family of directional search
    methods](#12-the-complete-family-of-directional-search-methods)
13. [Filtering: strings, regex, lists, functions, and
    attributes](#13-filtering-strings-regex-lists-functions-and-attributes)
14. [Working with links, images, tables, and
    lists](#14-working-with-links-images-tables-and-lists)
15. [Modifying the parse tree](#15-modifying-the-parse-tree)
16. [Output formatting and encoding](#16-output-formatting-and-encoding)
17. [A practical Requests + Beautiful Soup
    workflow](#17-a-practical-requests--beautiful-soup-workflow)
18. [Reusable scraping patterns](#18-reusable-scraping-patterns)
19. [Common errors and
    troubleshooting](#19-common-errors-and-troubleshooting)
20. [Performance, limitations, and responsible
    scraping](#20-performance-limitations-and-responsible-scraping)
21. [Quick-reference cheat sheet](#21-quick-reference-cheat-sheet)
22. [Practice exercises](#22-practice-exercises)
23. [Official references](#23-official-references)

------------------------------------------------------------------------

## 1. What is Beautiful Soup?

**Beautiful Soup** is a Python library for extracting information from
HTML and XML documents. It parses markup and builds a navigable tree of
Python objects.

It helps you:

-   Find HTML elements by tag, class, ID, attributes, text, or CSS
    selector.
-   Move through parent, child, sibling, and descendant relationships.
-   Extract text, links, image URLs, attributes, and tabular data.
-   Modify parsed markup and serialize it back to HTML/XML.

### What Beautiful Soup does not do

Beautiful Soup is a **parser and extraction library**, not a browser:

-   It does not download web pages by itself. Use `requests`, `httpx`,
    or another HTTP client.
-   It does not execute JavaScript.
-   It does not automatically interact with buttons, log in, or scroll
    pages.
-   It does not bypass authentication, access controls, CAPTCHAs, or
    anti-bot protections.

A typical workflow is:

``` text
Website → HTTP client (requests) → HTML response
        → BeautifulSoup parser → parsed tree
        → search / navigate → extract and clean data
        → CSV / JSON / database
```

------------------------------------------------------------------------

## 2. Installation and dependencies

Install Beautiful Soup:

``` bash
python -m pip install beautifulsoup4
```

Useful optional parsers:

``` bash
python -m pip install lxml html5lib
```

For downloading pages:

``` bash
python -m pip install requests
```

Check the installed version:

``` python
import bs4
print(bs4.__version__)
```

### Which parser should you choose?

  ------------------------------------------------------------------------
  Parser                  Install                  Characteristics
  ----------------------- ------------------------ -----------------------
  `html.parser`           Built into Python        Convenient default; no
                                                   extra dependency

  `lxml`                  `pip install lxml`       Usually fast; good HTML
                                                   and XML support

  `html5lib`              `pip install html5lib`   Browser-like HTML5
                                                   parsing; often slower
  ------------------------------------------------------------------------

Example:

``` python
from bs4 import BeautifulSoup

soup = BeautifulSoup(html, "html.parser")
# or
soup = BeautifulSoup(html, "lxml")
# or
soup = BeautifulSoup(html, "html5lib")
```

**Important:** Different parsers can build different trees from
malformed HTML. If extraction depends on a particular structure, choose
a parser deliberately and test it. Beautiful Soup's default parser may
vary depending on what is installed, so explicitly naming the parser
makes your code more predictable.

------------------------------------------------------------------------

## 3. A mental model: HTML becomes a tree

Consider:

``` html
<div class="article">
    <h1>Beautiful Soup</h1>
    <p>Learn <strong>Python</strong> scraping.</p>
    <a href="/guide">Read guide</a>
</div>
```

The parsed structure is conceptually:

``` text
div.article
├── h1
│   └── "Beautiful Soup"
├── p
│   ├── "Learn "
│   ├── strong
│   │   └── "Python"
│   └── " scraping."
└── a
    └── "Read guide"
```

The tree has relationships:

-   **Parent:** the immediate container of a node.
-   **Child:** an immediate node inside a parent.
-   **Descendant:** any node nested below an element, at any depth.
-   **Sibling:** another node with the same immediate parent.
-   **Next/previous element:** adjacent nodes in document traversal
    order.

Whitespace and line breaks in the source HTML can become text nodes.
This is why `.children`, `.contents`, and sibling navigation can
encounter strings containing spaces or newlines---not only HTML tags.

------------------------------------------------------------------------

## 4. The four important object types

Beautiful Soup mainly exposes four object types.

### 4.1 `BeautifulSoup`

Represents the parsed document as a whole.

``` python
soup = BeautifulSoup(html, "html.parser")
print(type(soup))
```

You can search and navigate from `soup`.

### 4.2 `Tag`

Represents an HTML/XML element such as `<div>`, `<p>`, or `<a>`.

``` python
tag = soup.find("p")
print(type(tag))
print(tag.name)
```

A `Tag` has a name, attributes, children, text, and navigation/search
methods.

### 4.3 `NavigableString`

Represents text inside markup.

``` python
tag = soup.find("p")
text_node = tag.string
print(type(text_node))
```

It behaves much like a Python string, but it is connected to the parse
tree. Convert it to a normal string when you need to keep it
independently:

``` python
plain_text = str(text_node)
```

A `NavigableString` does not have children, so it does not support
properties such as `.contents` or `.string`.

### 4.4 `Comment`

Represents an HTML comment.

``` python
from bs4 import Comment

soup = BeautifulSoup("<!-- internal note --><p>Hello</p>", "html.parser")
comment = soup.find(string=lambda s: isinstance(s, Comment))
print(comment)
print(type(comment))
```

Other special string subclasses include declarations and processing
instructions, particularly when parsing XML.

------------------------------------------------------------------------

## 5. Parsing HTML and XML

### 5.1 Parse an HTML string

``` python
from bs4 import BeautifulSoup

html = """
<html>
  <head><title>Example page</title></head>
  <body>
    <h1>Hello</h1>
    <p class="intro">Welcome to scraping.</p>
  </body>
</html>
"""

soup = BeautifulSoup(html, "html.parser")
```

### 5.2 Parse a local file

``` python
from bs4 import BeautifulSoup

with open("page.html", "r", encoding="utf-8") as file:
    soup = BeautifulSoup(file, "html.parser")
```

### 5.3 Parse XML

``` python
from bs4 import BeautifulSoup

xml = """
<catalog>
    <book id="b1">
        <title>Python Basics</title>
    </book>
</catalog>
"""

soup = BeautifulSoup(xml, "xml")
print(soup.find("title").get_text(strip=True))
```

Use an XML-capable parser such as `xml` (which relies on `lxml`) when
you need XML parsing semantics.

------------------------------------------------------------------------

## 6. Inspecting a parsed document

Useful inspection techniques:

``` python
print(soup)                 # Serialize the parsed document
print(soup.prettify())      # Indented, readable markup
print(soup.title)            # First title tag
print(soup.title.name)       # Tag name: "title"
print(soup.title.parent)     # Parent tag
print(soup.title.get_text()) # Text inside title
```

`prettify()` is excellent for debugging. It is not necessarily the
original source formatting: the parser may normalize or repair the
markup.

### Tag-name shortcuts

If the name does not conflict with a built-in attribute or method, you
can use:

``` python
soup.title
soup.body
soup.body.h1
```

These are convenient shortcuts for finding the first matching nested
tag. For clarity and reliability, explicit calls are often preferable:

``` python
soup.find("title")
soup.find("body").find("h1")
```

------------------------------------------------------------------------

## 7. Extracting text: `text`, `string`, and `get_text()`

These are related, but they are not interchangeable.

### 7.1 `.text`

Returns the combined text of all descendants as a normal Python string.

``` python
html = "<p>Learn <strong>Python</strong> scraping.</p>"
soup = BeautifulSoup(html, "html.parser")

p = soup.find("p")
print(p.text)
```

Output:

``` text
Learn Python scraping.
```

`.text` is a convenient property equivalent in purpose to `get_text()`
with default options.

### 7.2 `.string`

Returns the tag's single string content when the structure is
unambiguous. It returns `None` when the tag contains multiple pieces of
content.

``` python
soup = BeautifulSoup("<p>Hello</p>", "html.parser")
print(soup.p.string)  # Hello
```

With a nested tag containing one string, `.string` can still resolve to
that string:

``` python
soup = BeautifulSoup("<p>Hello <b>there</b></p>", "html.parser")
print(soup.p.string)  # None: the p contains multiple pieces
print(soup.b.string)  # there
```

Think of `.string` as asking: **Does this tag resolve to one unambiguous
string?**

### 7.3 `.get_text()`

Offers more control over text extraction.

Signature:

``` python
tag.get_text(separator="", strip=False, types=...)
```

Examples:

``` python
html = "<div><p>First</p><p>Second</p></div>"
soup = BeautifulSoup(html, "html.parser")
div = soup.div

print(div.get_text())
# FirstSecond

print(div.get_text(" "))
# First Second

print(div.get_text(" | "))
# First | Second

print(div.get_text(" ", strip=True))
# First Second
```

-   `separator`: text inserted between text fragments.
-   `strip=True`: strips leading/trailing whitespace from each extracted
    text piece.
-   `types`: controls which string-node types are included; most users
    can leave it at its default.

You can also use:

``` python
text = div.get_text(separator=" ", strip=True)
```

### Comparison

  -----------------------------------------------------------------------
  Expression                          Result / behaviour
  ----------------------------------- -----------------------------------
  `tag.text`                          All descendant text as a string

  `tag.get_text()`                    Text extraction with configurable
                                      options

  `tag.string`                        One unambiguous string, otherwise
                                      usually `None`

  `tag.strings`                       Generator of descendant text
                                      strings

  `tag.stripped_strings`              Generator of descendant strings
                                      with surrounding whitespace removed
  -----------------------------------------------------------------------

Examples:

``` python
for item in div.strings:
    print(repr(item))

for item in div.stripped_strings:
    print(item)
```

**Practical recommendation:** Use `get_text(" ", strip=True)` for many
scraping tasks. Use `.string` only when you expect a simple,
single-string tag.

------------------------------------------------------------------------

## 8. Attributes and tag properties

Given:

``` html
<a id="docs" class="external featured" href="https://example.com" target="_blank">
    Documentation
</a>
```

``` python
link = soup.find("a")
```

### 8.1 Common properties

  Property         Meaning                          Example
  ---------------- -------------------------------- ---------------------
  `.name`          Tag name                         `link.name` → `"a"`
  `.attrs`         Dictionary of attributes         `link.attrs`
  `.text`          Combined descendant text         `link.text`
  `.string`        Single string, if unambiguous    `link.string`
  `.contents`      List of immediate children       `link.contents`
  `.children`      Iterator of immediate children   `link.children`
  `.parent`        Immediate parent                 `link.parent`
  `.parents`       Iterator over ancestors          `link.parents`
  `.descendants`   Iterator over all descendants    `link.descendants`

### 8.2 Read attributes

Dictionary-style access:

``` python
href = link["href"]
classes = link["class"]
```

Safer access when an attribute may be missing:

``` python
href = link.get("href")
target = link.get("target")
missing = link.get("data-id")  # None if absent
```

Inspect all attributes:

``` python
print(link.attrs)
# {'id': 'docs', 'class': ['external', 'featured'],
#  'href': 'https://example.com', 'target': '_blank'}
```

### 8.3 HTML classes

A class attribute containing multiple values is generally represented as
a list:

``` python
print(link.get("class"))
# ['external', 'featured']

if "featured" in link.get("class", []):
    print("Featured link")
```

### 8.4 Data attributes

``` html
<div data-product-id="P100" data-stock="available"></div>
```

``` python
product = soup.find("div")
product_id = product.get("data-product-id")
stock = product.get("data-stock")
```

### 8.5 Attribute presence

``` python
soup.find_all("a", href=True)
soup.select("a[href]")
```

------------------------------------------------------------------------

## 9. Navigating the parse tree

Use the following example throughout this section:

``` python
html = """
<div>
    <h1>Title</h1>
    <p>First paragraph</p>
    <section>
        <p>Second paragraph</p>
    </section>
    <p>Third paragraph</p>
</div>
"""

soup = BeautifulSoup(html, "html.parser")
```

### 9.1 `.contents` --- immediate children as a list

``` python
div = soup.div
print(div.contents)
```

-   Returns a Python list.
-   Contains only direct children.
-   Includes text nodes for whitespace and line breaks.
-   Supports indexing and list operations, but do not mutate this list
    directly to modify the tree.

### 9.2 `.children` --- iterator over immediate children

``` python
for child in div.children:
    print(repr(child))
```

-   Returns an iterator/generator-like object.
-   Only immediate children.
-   Includes text nodes such as indentation and newline whitespace.

### 9.3 `.descendants` --- all nested descendants

``` python
for item in div.descendants:
    print(repr(item))
```

-   Visits children, grandchildren, and deeper descendants.
-   Includes both `Tag` and string nodes.
-   Useful when you need to inspect every node beneath a tag.

### `.contents` vs `.children` vs `.descendants`

  Feature                          `.contents`   `.children`   `.descendants`
  -------------------------------- ------------- ------------- ----------------
  Return form                      List          Iterator      Iterator
  Immediate children only          Yes           Yes           No
  Includes deeper nested nodes     No            No            Yes
  Includes whitespace text nodes   Yes           Yes           Yes
  Indexable directly               Yes           No            No

### 9.4 `.parent` --- immediate parent

``` python
p = soup.find("p")
print(p.parent)
```

For the first paragraph, the parent is the outer `<div>`.

### 9.5 `.parents` --- all ancestors

``` python
p = soup.find("p")

for ancestor in p.parents:
    print(ancestor.name)
```

This walks upward through the hierarchy, eventually reaching the
document object.

### 9.6 `.next_sibling` and `.previous_sibling`

These move to the adjacent node at the same tree level.

``` python
first_p = soup.find("p")

print(first_p.next_sibling)
print(first_p.previous_sibling)
```

**Important:** The next sibling may be a whitespace `NavigableString`,
not the next HTML tag. Pretty-printed HTML commonly has
newline/indentation nodes between tags.

To find the next matching sibling tag, use:

``` python
first_p.find_next_sibling("p")
```

### 9.7 `.next_siblings` and `.previous_siblings`

Iterate through later or earlier siblings:

``` python
for sibling in first_p.next_siblings:
    print(repr(sibling))

for sibling in first_p.previous_siblings:
    print(repr(sibling))
```

These also may include whitespace strings.

### 9.8 `.next_element` and `.previous_element`

These follow **document traversal order**, not just sibling order.

``` python
first_p = soup.find("p")

print(first_p.next_element)
print(first_p.previous_element)
```

A subtle but important example:

``` python
heading = soup.find("h1")
print(heading.next_element)
```

This can return the text inside the `<h1>`---`"Title"`---because the
first item encountered after entering the heading is its text node. It
does not necessarily return the next sibling tag.

### 9.9 `.next_elements` and `.previous_elements`

Iterators over all later/earlier elements in document traversal order:

``` python
for element in first_p.next_elements:
    print(repr(element))
```

### Navigation summary

  Property               Direction / scope               What it returns
  ---------------------- ------------------------------- -----------------------------
  `.parent`              One level up                    One parent
  `.parents`             All levels up                   Iterator
  `.contents`            One level down                  List of direct children
  `.children`            One level down                  Iterator of direct children
  `.descendants`         All levels down                 Iterator
  `.next_sibling`        Next node at same level         One node
  `.previous_sibling`    Previous node at same level     One node
  `.next_siblings`       Later nodes at same level       Iterator
  `.previous_siblings`   Earlier nodes at same level     Iterator
  `.next_element`        Next in document order          One node
  `.previous_element`    Previous in document order      One node
  `.next_elements`       All later in document order     Iterator
  `.previous_elements`   All earlier in document order   Iterator

------------------------------------------------------------------------

## 10. Searching with `find()` and `find_all()`

These are the core search methods.

### 10.1 `find()` --- first matching result

``` python
first_paragraph = soup.find("p")
```

Returns one `Tag`, or `None` if no match exists.

### 10.2 `find_all()` --- all matching results

``` python
paragraphs = soup.find_all("p")

for paragraph in paragraphs:
    print(paragraph.get_text(strip=True))
```

Returns a list-like `ResultSet`, including an empty result if no matches
exist.

### Comparison

  Feature           `find()`                 `find_all()`
  ----------------- ------------------------ -----------------------------
  Number returned   First match              All matches by default
  Return type       `Tag` or `None`          `ResultSet`
  No match          `None`                   Empty result
  Common use        One title, one element   Repeated cards, links, rows

### 10.3 Search by tag name

``` python
soup.find("h1")
soup.find_all("p")
```

### 10.4 Search by ID

``` python
soup.find(id="main")
soup.find_all(id="main")
```

### 10.5 Search by class

Use `class_` because `class` is a reserved Python keyword:

``` python
soup.find("div", class_="product")
soup.find_all("div", class_="product")
```

Multiple class values can be searched in several ways:

``` python
soup.find_all("div", class_="product featured")
soup.find_all("div", class_=["product", "featured"])
```

For more precise combinations of classes, CSS selectors are often
clearer:

``` python
soup.select("div.product.featured")
```

### 10.6 Search by arbitrary attributes

``` python
soup.find("a", attrs={"href": "/about"})
soup.find_all("div", attrs={"data-type": "product"})
```

Keyword-argument form:

``` python
soup.find_all("a", href=True)
soup.find_all("input", type="text")
```

### 10.7 Limit the number of results

``` python
soup.find_all("p", limit=2)
```

Stops after the first two matches.

### 10.8 Search only direct children with `recursive=False`

By default, `find_all()` searches descendants at all depths.

``` python
container = soup.find("div")

container.find_all("p")                    # Nested p tags too
container.find_all("p", recursive=False)   # Direct child p tags only
```

`recursive` is supported by `find()` and `find_all()`, not by every
search method.

### 10.9 The call shortcut

Calling a `Tag` or `BeautifulSoup` object is equivalent to `find_all()`:

``` python
soup("a")
# Equivalent to:
soup.find_all("a")
```

------------------------------------------------------------------------

## 11. Searching with CSS selectors: `select()` and `select_one()`

Beautiful Soup integrates CSS selector support through **Soup Sieve**.

### 11.1 `select()` --- all matching elements

``` python
products = soup.select(".product")
```

Returns a list-like `ResultSet` of every matching tag.

### 11.2 `select_one()` --- first matching element

``` python
product = soup.select_one(".product")
```

Returns one `Tag`, or `None` if no match exists.

### Comparison

  Feature    `select()`              `select_one()`
  ---------- ----------------------- ------------------------
  Returns    List-like `ResultSet`   Single `Tag` or `None`
  Matches    All                     First
  No match   Empty result            `None`
  Use        Repeated records        One field/element

**Common mistake:**

``` python
prices = soup.select(".price")
print(prices.text)  # Wrong: prices is a ResultSet
```

Correct:

``` python
prices = soup.select(".price")
for price in prices:
    print(price.get_text(strip=True))
```

Or:

``` python
price = soup.select_one(".price")
if price:
    print(price.get_text(strip=True))
```

### 11.3 Common CSS selectors

  Selector              Meaning
  --------------------- -----------------------------------------------
  `p`                   All `<p>` elements
  `*`                   All elements
  `.product`            Any element with class `product`
  `#main`               Element with ID `main`
  `div.product`         A `<div>` with class `product`
  `.product.featured`   Element having both classes
  `div p`               Any `<p>` nested anywhere inside a `<div>`
  `div > p`             `<p>` that is a direct child of `<div>`
  `h1, h2`              `<h1>` or `<h2>`
  `a[href]`             `<a>` with an `href` attribute
  `a[href="x"]`         Exact attribute value
  `a[href^="https"]`    Attribute starts with `https`
  `a[href$=".pdf"]`     Attribute ends with `.pdf`
  `a[href*="docs"]`     Attribute contains `docs`
  `li:first-child`      First child `<li>` when it is the first child
  `li:last-child`       Last child `<li>`
  `p:nth-of-type(2)`    Second `<p>` among siblings of that tag type
  `input:checked`       Checked input
  `div:not(.hidden)`    `<div>` not having class `hidden`

Examples:

``` python
soup.select(".product .price")
soup.select("div#main > p")
soup.select("a[href^='https://']")
soup.select("h1, h2, h3")
soup.select("ul.items > li:nth-of-type(2)")
```

### 11.4 Select relative to a specific tag

``` python
card = soup.select_one(".product-card")

if card:
    name = card.select_one(".name")
    price = card.select_one(".price")
```

This searches inside `card`, not across the entire document.

### 11.5 `select()` versus `find_all()`

  ----------------------------------------------------------------------------------
  Task                    CSS selector            Search API
  ----------------------- ----------------------- ----------------------------------
  All product cards       `select(".product")`    `find_all(class_="product")`

  First page heading      `select_one("h1")`      `find("h1")`

  Links with href         `select("a[href]")`     `find_all("a", href=True)`

  Direct child paragraphs `select("div > p")`     `find_all("p", recursive=False)`
                                                  on the div

  Complex combinations    Often concise           Sometimes requires multiple
                                                  arguments/functions
  ----------------------------------------------------------------------------------

Choose the method that makes the intent clearest. CSS selectors are
especially convenient when the site's HTML structure is already
familiar.

------------------------------------------------------------------------

## 12. The complete family of directional search methods

Beautiful Soup offers search methods that move in different directions
through the tree. They use similar filtering rules to `find()` and
`find_all()`.

### 12.1 Downward search

  ------------------------------------------------------------------------
  Method                   Purpose                 Returns
  ------------------------ ----------------------- -----------------------
  `find_next()`            First matching element  One match or `None`
                           after current element   
                           in document order       

  `find_all_next()`        All matching elements   ResultSet
                           after current element   

  `find_next_sibling()`    First matching later    One match or `None`
                           sibling                 

  `find_next_siblings()`   All matching later      ResultSet
                           siblings                
  ------------------------------------------------------------------------

### 12.2 Upward search

  Method             Purpose                   Returns
  ------------------ ------------------------- ---------------------
  `find_parent()`    First matching ancestor   One match or `None`
  `find_parents()`   All matching ancestors    ResultSet

### 12.3 Backward search

  ----------------------------------------------------------------------------
  Method                       Purpose                 Returns
  ---------------------------- ----------------------- -----------------------
  `find_previous()`            First matching element  One match or `None`
                               before current element  
                               in document order       

  `find_all_previous()`        All matching earlier    ResultSet
                               elements                

  `find_previous_sibling()`    First matching earlier  One match or `None`
                               sibling                 

  `find_previous_siblings()`   All matching earlier    ResultSet
                               siblings                
  ----------------------------------------------------------------------------

### 12.4 Important: `find_next()` is not the same as `find_next_sibling()`

Given:

``` html
<div>
    <h1>Title</h1>
    <p>First paragraph</p>
    <section>
        <p>Second paragraph</p>
    </section>
    <p>Third paragraph</p>
</div>
```

Example A:

``` python
heading = soup.find("h1")
result = heading.find_next()
```

With no filter, `find_next()` returns the next `Tag` encountered in
document order. That is the first `<p>`.

Example B:

``` python
heading = soup.find("h1")
result = heading.find_next("section")
```

Returns the `<section>` element.

Example C:

``` python
first_p = soup.find("p")
result = first_p.find_next()
```

The next `Tag` is `<section>`, because the section tag is encountered
before its nested second paragraph.

Example D:

``` python
first_p = soup.find("p")
result = first_p.find_next("p")
```

Returns the second paragraph inside `<section>`.

Example E:

``` python
first_p = soup.find("p")
result = first_p.find_next_sibling("p")
```

Returns the third paragraph. It searches later siblings of the first
paragraph, not arbitrary descendants elsewhere in the document.

### 12.5 Useful related methods

``` python
tag.find_next(string="Exact text")
tag.find_all_next("a")
tag.find_parent("section")
tag.find_parents("div")
tag.find_previous("h1")
tag.find_next_siblings("p")
```

A method such as `find_next()` searches for a matching **Tag** by
default. To search for strings, use the `string` argument:

``` python
tag.find_next(string=True)
```

------------------------------------------------------------------------

## 13. Filtering: strings, regex, lists, functions, and attributes

The search methods accept several kinds of filters.

### 13.1 Exact tag-name string

``` python
soup.find_all("a")
```

### 13.2 Regular expressions

``` python
import re

soup.find_all(re.compile("^h[1-6]$"))
soup.find_all("a", href=re.compile(r"^https://"))
```

### 13.3 A list of tag names

``` python
soup.find_all(["h1", "h2", "h3"])
```

### 13.4 `True` --- match all tags (or all strings when used with `string`)

``` python
soup.find_all(True)               # All tags
soup.find_all(string=True)        # All string nodes
```

### 13.5 A custom function

A function can decide whether an element matches.

``` python
def has_data_id(tag):
    return tag.has_attr("data-id")

matches = soup.find_all(has_data_id)
```

For attribute filtering, the function receives the attribute value:

``` python
def external_link(href):
    return href is not None and href.startswith("https://")

links = soup.find_all("a", href=external_link)
```

### 13.6 Search by text with `string`

``` python
soup.find(string="Welcome")
```

Use regex for partial/pattern matching:

``` python
soup.find_all(string=re.compile("price", re.I))
```

To find tags based on their text, you may inspect the returned string's
parent:

``` python
text_node = soup.find(string=re.compile("Welcome"))
if text_node:
    print(text_node.parent)
```

**Note:** In current Beautiful Soup 4 documentation, `string` is the
preferred argument name. Older tutorials may use the deprecated `text`
argument.

### 13.7 Search using multiple conditions

``` python
soup.find_all(
    "a",
    class_="external",
    href=re.compile(r"^https://")
)
```

Or use `attrs`:

``` python
soup.find_all(
    "div",
    attrs={
        "data-category": "books",
        "data-active": "true"
    }
)
```

------------------------------------------------------------------------

## 14. Working with links, images, tables, and lists

### 14.1 Extract all links

``` python
links = []

for a in soup.find_all("a", href=True):
    links.append({
        "text": a.get_text(" ", strip=True),
        "href": a["href"]
    })
```

### 14.2 Extract image URLs and alt text

``` python
images = []

for img in soup.find_all("img"):
    images.append({
        "src": img.get("src"),
        "alt": img.get("alt")
    })
```

Some sites use lazy-loading attributes such as `data-src`,
`data-lazy-src`, or `srcset`. Inspect the actual markup and extract the
appropriate attribute:

``` python
src = img.get("src") or img.get("data-src")
```

### 14.3 Extract a table

``` html
<table>
  <tr><th>Name</th><th>Role</th></tr>
  <tr><td>Asha</td><td>Engineer</td></tr>
  <tr><td>Ravi</td><td>Analyst</td></tr>
</table>
```

``` python
table = soup.find("table")

rows_data = []

for row in table.find_all("tr"):
    cells = row.find_all(["th", "td"])
    values = [cell.get_text(" ", strip=True) for cell in cells]
    rows_data.append(values)

print(rows_data)
```

For complex tables, account for `rowspan`, `colspan`, nested tables, and
header rows; a simple row-by-row loop does not expand merged cells.

### 14.4 Extract list items

``` python
items = [
    li.get_text(" ", strip=True)
    for li in soup.select("ul.menu > li")
]
```

### 14.5 Convert relative URLs to absolute URLs

A page may contain `/products/123` instead of a complete URL.

``` python
from urllib.parse import urljoin

base_url = "https://example.com/catalog/"
absolute_url = urljoin(base_url, "/products/123")
print(absolute_url)
# https://example.com/products/123
```

Use the page's final response URL as the base if redirects occurred:

``` python
base_url = response.url
```

------------------------------------------------------------------------

## 15. Modifying the parse tree

Beautiful Soup can change the in-memory parse tree. This is useful for
cleaning markup or generating transformed HTML.

### 15.1 Change tag name

``` python
tag = soup.find("b")
tag.name = "strong"
```

### 15.2 Change or add attributes

``` python
tag["class"] = ["highlight"]
tag["id"] = "important"
tag["data-status"] = "active"
```

### 15.3 Remove an attribute

``` python
del tag["id"]
```

### 15.4 Replace a tag's contents

``` python
tag.string = "Replacement text"
```

Assigning `.string` replaces the tag's contents, including nested
markup.

### 15.5 Append content

``` python
from bs4 import NavigableString

tag.append(NavigableString(" Additional text"))
```

### 15.6 Append a new tag

``` python
from bs4 import BeautifulSoup

new_tag = soup.new_tag("p")
new_tag.string = "New paragraph"
soup.body.append(new_tag)
```

### 15.7 Insert before or after

``` python
new_tag = soup.new_tag("hr")
existing_tag.insert_before(new_tag)
# or:
existing_tag.insert_after(new_tag)
```

### 15.8 Remove a tag but keep its contents: `unwrap()`

``` python
tag = soup.find("strong")
tag.unwrap()
```

The tag is removed while its contents remain in the tree.

### 15.9 Remove a tag and its contents: `decompose()`

``` python
tag = soup.find("script")
tag.decompose()
```

The tag and its contents are destroyed from the tree.

### 15.10 Extract a tag: `extract()`

``` python
tag = soup.find("aside")
removed_tag = tag.extract()
```

Removes the tag from the original tree and returns it.

### 15.11 Replace a node

``` python
old_tag = soup.find("b")
new_tag = soup.new_tag("strong")
new_tag.string = "Important"
old_tag.replace_with(new_tag)
```

### 15.12 Clear contents

``` python
tag.clear()
```

Removes a tag's children but leaves the tag itself.

**Caution:** These operations modify the in-memory tree. They do not
change the live website or the original downloaded file unless you
explicitly write the modified output somewhere.

------------------------------------------------------------------------

## 16. Output formatting and encoding

### 16.1 Print readable HTML

``` python
print(soup.prettify())
```

### 16.2 Convert to string

``` python
html_output = str(soup)
```

### 16.3 Encode as bytes

``` python
html_bytes = soup.encode("utf-8")
```

### 16.4 Write to a file

``` python
with open("cleaned.html", "w", encoding="utf-8") as file:
    file.write(soup.prettify())
```

### 16.5 Inspect detected encoding

``` python
print(soup.original_encoding)
```

Beautiful Soup converts input markup to Unicode while parsing. When
writing files, specify the encoding explicitly.

------------------------------------------------------------------------

## 17. A practical Requests + Beautiful Soup workflow

Beautiful Soup parses markup; Requests fetches it.

``` python
import requests
from bs4 import BeautifulSoup

url = "https://example.com/"

headers = {
    "User-Agent": "Mozilla/5.0 (compatible; LearningScraper/1.0)"
}

response = requests.get(url, headers=headers, timeout=20)

# Raise an exception for HTTP errors such as 404 or 500.
response.raise_for_status()

# Parse the response body.
soup = BeautifulSoup(response.text, "html.parser")

# Extract the page title safely.
title_tag = soup.select_one("title")
title = title_tag.get_text(strip=True) if title_tag else None

print("Title:", title)

# Extract links.
for link in soup.select("a[href]"):
    text = link.get_text(" ", strip=True)
    href = link.get("href")
    print(text, href)
```

### Recommended workflow

1.  Inspect the page in a browser's developer tools.
2.  Identify a stable container for each record.
3.  Fetch the page and check the HTTP status.
4.  Parse the response using a deliberate parser.
5.  Select each record container.
6.  Extract individual fields relative to that container.
7.  Handle missing elements and missing attributes.
8.  Normalize whitespace and URLs.
9.  Save the extracted records.
10. Log failures and validate the output.

### Why parse each record container first?

Suppose a page has many product cards. Avoid searching the entire
document separately for every product name and price. Instead:

``` python
for card in soup.select(".product-card"):
    name_tag = card.select_one(".product-name")
    price_tag = card.select_one(".price")

    name = name_tag.get_text(" ", strip=True) if name_tag else None
    price = price_tag.get_text(" ", strip=True) if price_tag else None
```

This keeps fields associated with the correct product.

------------------------------------------------------------------------

## 18. Reusable scraping patterns

### 18.1 A safe text helper

``` python
def get_text_or_none(parent, selector):
    tag = parent.select_one(selector)
    return tag.get_text(" ", strip=True) if tag else None
```

Usage:

``` python
name = get_text_or_none(card, ".product-name")
price = get_text_or_none(card, ".price")
```

### 18.2 A safe attribute helper

``` python
def get_attr_or_none(parent, selector, attribute):
    tag = parent.select_one(selector)
    return tag.get(attribute) if tag else None
```

Usage:

``` python
image_url = get_attr_or_none(card, "img", "src")
```

### 18.3 Extract records into dictionaries

``` python
records = []

for card in soup.select(".product-card"):
    records.append({
        "name": get_text_or_none(card, ".product-name"),
        "price": get_text_or_none(card, ".price"),
        "url": get_attr_or_none(card, "a", "href"),
        "image": get_attr_or_none(card, "img", "src"),
    })
```

### 18.4 Save as JSON

``` python
import json

with open("products.json", "w", encoding="utf-8") as file:
    json.dump(records, file, ensure_ascii=False, indent=2)
```

### 18.5 Save as CSV

``` python
import csv

if records:
    with open("products.csv", "w", newline="", encoding="utf-8") as file:
        writer = csv.DictWriter(file, fieldnames=records[0].keys())
        writer.writeheader()
        writer.writerows(records)
```

### 18.6 Add a delay between requests

For a small, permitted crawl:

``` python
import time

time.sleep(2)
```

For larger crawls, use a deliberate rate limit, caching, retries with
backoff, and the site's published access rules. Do not send concurrent
requests merely because the code allows it.

------------------------------------------------------------------------

## 19. Common errors and troubleshooting

### 19.1 `AttributeError: 'NoneType' object has no attribute ...`

Cause: a search returned `None`, and the code tried to access a property
on it.

Fragile:

``` python
title = soup.find("h1").text
```

Safer:

``` python
title_tag = soup.find("h1")
title = title_tag.get_text(strip=True) if title_tag else None
```

### 19.2 `AttributeError: 'ResultSet' object has no attribute 'text'`

Cause: `select()` or `find_all()` returned multiple results.

Wrong:

``` python
items = soup.select(".item")
print(items.text)
```

Correct:

``` python
for item in items:
    print(item.get_text(strip=True))
```

Or use `select_one()` if you only need the first match.

### 19.3 `.string` returns `None`

Cause: the tag contains multiple text fragments or nested structures.

Use:

``` python
tag.get_text(" ", strip=True)
```

### 19.4 `.next_sibling` returns `'\n'` or spaces

Cause: whitespace is a text node and counts as a sibling.

Use:

``` python
tag.find_next_sibling("p")
```

when you want the next matching paragraph tag.

### 19.5 `find_next()` returns a nested container

Cause: it searches in document traversal order. A container tag may
occur before a nested target tag.

Use a filter:

``` python
tag.find_next("p")
```

or sibling-specific search:

``` python
tag.find_next_sibling("p")
```

depending on the intended relationship.

### 19.6 The page looks empty or missing data

Possible causes:

-   The data is inserted by JavaScript after the initial HTML loads.
-   The response is a login page, consent page, or error page.
-   The request was blocked or redirected.
-   The data is embedded in JSON or another script rather than visible
    markup.
-   The selector no longer matches because the website changed.

Inspect:

``` python
print(response.status_code)
print(response.url)
print(response.text[:2000])
```

If the content is generated client-side, inspect the site's permitted
API/data source or use a browser automation tool where appropriate.
Beautiful Soup itself does not execute JavaScript.

### 19.7 Parser differences

Malformed markup can be repaired differently by different parsers.
Compare `html.parser`, `lxml`, and `html5lib` if the tree looks
unexpected.

### 19.8 Selenium and Beautiful Soup

They solve different problems:

-   Selenium/browser automation can load pages and interact with
    browser-rendered content.
-   Beautiful Soup parses the HTML you give it.
-   You can pass rendered page source from Selenium to Beautiful Soup:

``` python
html = driver.page_source
soup = BeautifulSoup(html, "html.parser")
```

Use browser automation only when needed; a normal HTTP request is
simpler and lighter when the required data is present in the response.

------------------------------------------------------------------------

## 20. Performance, limitations, and responsible scraping

### Performance tips

-   Choose `lxml` when speed matters and it is suitable for the markup.
-   Search within a record container rather than repeatedly scanning the
    whole page.
-   Use `limit` when you only need a few matches.
-   Avoid repeatedly reparsing the same HTML.
-   Extract only the fields you need.
-   Use CSS selectors or the search API based on clarity, not habit.

### Limitations

-   Beautiful Soup is not a browser and does not run JavaScript.
-   It cannot guarantee that a website's markup is stable.
-   It does not automatically understand the meaning of a page's data.
-   It does not download files or manage browser sessions by itself.

### Responsible scraping checklist

-   Read the website's terms and published crawling rules.
-   Respect applicable laws, privacy, copyright, and access
    restrictions.
-   Avoid collecting personal or sensitive data without a valid basis.
-   Use reasonable request rates and identify your client honestly.
-   Do not bypass login walls, CAPTCHAs, paywalls, or technical access
    controls.
-   Prefer an official API or downloadable dataset when available.
-   Stop if the site signals that automated access is not permitted.

------------------------------------------------------------------------

## 21. Quick-reference cheat sheet

### Parse

``` python
soup = BeautifulSoup(html, "html.parser")
```

### Inspect

``` python
soup.prettify()
soup.title
soup.title.name
```

### Extract text

``` python
tag.text
tag.string
tag.get_text(" ", strip=True)
list(tag.strings)
list(tag.stripped_strings)
```

### Read attributes

``` python
tag["href"]            # Raises KeyError if absent
tag.get("href")        # Returns None if absent
tag.attrs              # All attributes
tag.get("class", [])
```

### Search

``` python
soup.find("p")
soup.find_all("p")
soup.find(id="main")
soup.find_all(class_="item")
soup.select(".item")
soup.select_one(".item")
```

### Navigate

``` python
tag.parent
tag.parents
tag.contents
tag.children
tag.descendants
tag.next_sibling
tag.previous_sibling
tag.next_element
tag.previous_element
```

### Directional search

``` python
tag.find_parent("div")
tag.find_parents("div")
tag.find_next("p")
tag.find_all_next("a")
tag.find_next_sibling("p")
tag.find_next_siblings("p")
tag.find_previous("h1")
tag.find_all_previous("a")
tag.find_previous_sibling("p")
tag.find_previous_siblings("p")
```

### Modify

``` python
tag.name = "section"
tag["class"] = ["new-class"]
tag.append("text")
tag.insert(0, new_tag)
tag.unwrap()
tag.decompose()
tag.extract()
tag.clear()
tag.replace_with(new_tag)
```

### Save

``` python
html_output = soup.prettify()

with open("output.html", "w", encoding="utf-8") as file:
    file.write(html_output)
```

### Essential distinctions to remember

  -----------------------------------------------------------------------
  Do not confuse                      Correct distinction
  ----------------------------------- -----------------------------------
  `.text` vs `.string`                All descendant text vs one
                                      unambiguous string

  `.contents` vs `.children`          List vs iterator; both direct
                                      children

  `.children` vs `.descendants`       Direct children vs all nested
                                      descendants

  `.next_sibling` vs `.next_element`  Same-level next node vs next node
                                      in document order

  `find_next()` vs                    Anywhere later in document order vs
  `find_next_sibling()`               later sibling only

  `find()` vs `find_all()`            First match vs all matches

  `select_one()` vs `select()`        First CSS match vs all CSS matches

  `select()` vs `find_all()`          CSS selector language vs Beautiful
                                      Soup's filter arguments
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 22. Practice exercises

Use this HTML:

``` python
html = """
<div class="catalog">
    <h1>Catalog</h1>

    <div class="product" data-id="101">
        <h2 class="name">Laptop</h2>
        <p class="price">₹60,000</p>
        <a href="/laptop">Details</a>
    </div>

    <div class="product" data-id="102">
        <h2 class="name">Mobile</h2>
        <p class="price">₹25,000</p>
        <a href="/mobile">Details</a>
    </div>
</div>
"""

soup = BeautifulSoup(html, "html.parser")
```

Try these tasks:

1.  Extract the page's `<h1>` text.
2.  Find all product containers.
3.  Extract the name and price of each product.
4.  Fetch the first product only.
5.  Find all links that have an `href`.
6.  Extract the `data-id` attribute of every product.
7.  Find the `<p>` that is a direct child of each product.
8.  From the first product, find its next sibling product.
9.  Compare `product.contents` with `product.descendants`.
10. Rewrite the extraction using CSS selectors only.

### Suggested solution

``` python
# 1
print(soup.select_one("h1").get_text(strip=True))

# 2
products = soup.select(".product")

# 3
for product in products:
    name = product.select_one(".name")
    price = product.select_one(".price")

    print(
        name.get_text(strip=True) if name else None,
        price.get_text(strip=True) if price else None
    )

# 4
first_product = soup.select_one(".product")

# 5
for link in soup.select("a[href]"):
    print(link.get_text(strip=True), link["href"])

# 6
for product in products:
    print(product.get("data-id"))

# 7
for product in products:
    print(product.find_all("p", recursive=False))

# 8
first_product = soup.select_one(".product")
next_product = first_product.find_next_sibling("div", class_="product")
```

------------------------------------------------------------------------

## 23. Official references

The tutorial focuses on the most useful Beautiful Soup 4 API for
practical scraping. The library has additional advanced features and
version-specific details; consult the official documentation for the
complete API and current behaviour.

1.  **Beautiful Soup 4 official documentation**\
    https://www.crummy.com/software/BeautifulSoup/bs4/doc/

2.  **Soup Sieve documentation** --- CSS selector implementation used by
    Beautiful Soup\
    https://facelessuser.github.io/soupsieve/

3.  **Requests documentation** --- HTTP requests in Python\
    https://requests.readthedocs.io/

4.  **Python `re` documentation** --- regular expressions\
    https://docs.python.org/3/library/re.html

5.  **Python `urllib.parse` documentation** --- URL joining and parsing\
    https://docs.python.org/3/library/urllib.parse.html

------------------------------------------------------------------------

## Final learning recommendation

Learn Beautiful Soup in this order:

1.  Parse HTML and understand `soup`, `Tag`, and `NavigableString`.
2.  Master text extraction and attributes.
3.  Understand tree navigation: parent, children, descendants, siblings,
    and document order.
4.  Master `find()` / `find_all()` and their filters.
5.  Master CSS selectors with `select()` / `select_one()`.
6.  Practice extracting repeated records into dictionaries.
7.  Combine Requests and Beautiful Soup, then handle errors, pagination,
    and output files.

The most important practical pattern is:

``` python
for record in soup.select(".record"):
    title = record.select_one(".title")
    link = record.select_one("a[href]")

    data = {
        "title": title.get_text(" ", strip=True) if title else None,
        "url": link.get("href") if link else None,
    }

    print(data)
```

This pattern---**select all record containers, then extract each field
within its own container**---is the foundation of many maintainable
Beautiful Soup scrapers.
