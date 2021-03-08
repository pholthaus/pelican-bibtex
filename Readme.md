Pelican BibTeX
==============

Organize your scientific publications with BibTeX in Pelican. Original author is Vlad Niculae (http://vene.ro).
This project is forked from https://github.com/vene and provides support for multiple BibTeX files and slightly
more advanced sorting and displaying options.

*Note*: This code is unlicensed. It was not submitted to the `pelican-plugins`
official repository because of the license constraint imposed there.


Requirements
============

`pelican_bibtex` requires `pybtex`.

```bash
pip install pybtex
```

The example template (which is optional) requires `Markdown` at runtime.

```bash
pip install Markdown
```

How to Configure the Plugin
===========================

This plugin reads a user-specified BibTeX file and populates the context with
a list of publications, ready to be used in your Jinja2 template.

As with all pelican plugins, you need to specify the path in which pelican searches for plugins.
Let's assume you have cloned or downloaded this plugin to the folder `plugins/pelican-bibtex` inside your pelican folder.
You then need to configure pelican like this by inserting the following into in `pelicanconf.py`:

```python
PLUGIN_PATHS = [ 'plugins' ]
PLUGINS = [ 'other-plugin-a', 'other-plugin-b', 'pelican-bibtex' ]
```

The plugin itself is configured as follows (also in `pelicanconf.py`):

```python
PUBLICATIONS = {
  'mypub': { 'file': 'content/publications.bib' }
}
```

What the Plugin provides
========================

You will be able to find a `publications` variable in all templates. If the given
`file` is present and readable, *BibTeX* entries will also be accessible in the template.
As a result, the `publications` variable (`dict`) will contain a field with the identifier `mypub` (also a `dict`) containing entries of the following tuple:

```
(key, year, text, bibtex, pdf, slides, poster)
```

1. `key` is the *BibTeX* key (identifier) of the entry.
2. `year` is the year when the entry was published.  Useful for grouping by year in templates using Jinja's `groupby`
3. `text` is the HTML formatted entry, generated by `pybtex`.
4. `bibtex` is a string containing *BibTeX* code for the entry, useful to make it available to people who want to cite your work.
5. `pdf`, `slides`, `poster`: in your *BibTeX* file, you can add these special fields, for example:
```
@article{
   foo13
   ...
   pdf = {/papers/foo13.pdf},
   slides = {/slides/foo13.html}
}
```

If a field is not defined, the tuple field will be `None`.  Furthermore, the fields are stripped from the generated *BibTeX* (found in the `bibtex` field).

Advanced Configuration
======================

The configuration allows for multiple files and control over several variables **that are also passed on to the template phase**.
The following optional fields can be specified for each bibliography in the `PUBLICATIONS` variable:

* `title`: Title for this bibliography (h2), if empty, the bibliographies key is used instead.
* `header`: Bool denoting whether a header (h2) should be produced for this bibliography.
* `split`: Bool denoting whether bibliographies should be split by year (h3).
* `split_link`: Bool denoting whether to generate a "link to top" after each year's section.
* `bottom_link`: Bool denoting whether to generate a "link to top" after this bibliography.
* `highlight`: String, e.g., a name, that will be entailed in a \<strong\> tag to highlight.
* `all_bibtex`: Provide a link to the original .bib file

A `PUBLICATIONS_NAVBAR` variable can be used in `pelicanconf.py` to specify whether or not to produce a line that contains links to each bibliography section.

The following example contains a default entry `simple-example` plus a secondary bibliography `modified-defaults` in which all variables that can be configured through the plugin are set to non-standard.

```python
PUBLICATIONS = {
  'simple-example': { 'file': 'all.bib' },
  'modified-defaults': {
    'file': 'all.bib',
    'title': 'Different Appearance',
    'header': False,
    'split': False,
    'split_link': False,
    'bottom_link': False,
    'highlight': ['Patrick Holthaus'],
    'all_bibtex': True }
}

PUBLICATIONS_NAVBAR = True
```

Template Example
================

You probably want to define a `publications.html` direct template that makes use of the variables this plugin defines.
Create an appropriate template file in the folder ```templates``` and add it to pelican's search path by editing ```pelicanconf.py```:

```python
EXTRA_TEMPLATES_PATHS = [ 'templates' ]
```

The template also needs to be included in your page by adding the following to the header section.
The entry must match the template's file name.

```rst
:template: publications
```

---
**NOTE**

If you don't define a template, this plugin won't achieve you any visible result.

---

Consider the following template for inclusion. It respects all configuration methods described above.
Additionally, the following header entries can be used per page to restrict what bibliographies are displayed on that page and on which (header) level.

```rst
:bibliographies: modified-defaults,simple-example
:bibheader: 2
```

By default, all bibliographies are considered and included with an `<h2>` tag.

<details><summary><strong>Click to reveal example template</strong></summary>

```jinja2
{% extends "page.html" %}
{% block content %}

<!-- header part: original bootstrap pages content block -->
<section id="content" class="body">
    {% if page.title %}
        <h1 class="entry-title">{{ page.title }}</h1>
    {% endif %}
    {% import 'includes/translations.html' as translations with context %}
    {{ translations.translations_for(page) }}
    {% if PDF_PROCESSOR %}
        <a href="{{ SITEURL }}/pdf/{{ page.slug }}.pdf">
            get the pdf
        </a>
    {% endif %}
    <div class="entry-content">
        {{ page.content }}

<!-- add header navbar -->
{% if PUBLICATIONS_NAVBAR %}
<p>
  {% for bib in publications|sort %}
    <a class="reference external" href="#{{ bib }}">{{ publications[bib]['title'] }}</a>
    {% if not loop.last %}&middot;{% endif %}
  {% endfor %}
</p>
{% endif %}

<!-- check page header -->
{% if page.bibliographies %}
  {% set bibliographies = page.bibliographies.split(',') %}
{% else %}
  {% set bibliographies = publications.keys() %}
{% endif %}
{% if page.bibheader %}
  {% set mainheader = page.bibheader %}
  {% set splitheader = mainheader|int() + 1 %}
{% else %}
  {% set mainheader = 2 %}
  {% set splitheader = 3 %}
{% endif %}

<!-- add publication list -->
{% for bib in publications|sort %}
  {% if bib in bibliographies %}
    <div class="publications" id="{{ bib }}">
      {% if publications[bib]['header'] %}
        <h{{ mainheader }}>{{ publications[bib]['title'] }}</h{{ mainheader }}>
      {% endif %}
      {% if publications[bib]['all_bibtex'] %}
      {% set ns = namespace(fbt='') %}
      {% for key, year, text, bibtex, pdf, slides, poster in publications[bib]['data'] %}
        {% set ns.fbt = ns.fbt + bibtex %}
      {% endfor %}
      You can <a href="{{ publications[bib]['path'] }}" download>download</a> or <a data-toggle="collapse" data-target="#{{ bib }}-bib">display</a> all {{ publications[bib]['title']|lower }} in BibTeX format.
      <div style="clear:both" id="{{ bib }}-bib" class="collapse">
        {% set fbt = '```tex\n' + ns.fbt + '```' %}
        {{ fbt|md }}
        <a data-toggle="collapse" data-target="#{{ bib }}-bib">Hide BibTeX for all {{ publications[bib]['title']|lower }}</a>.
      </div>
      {% endif %}
      {% if publications[bib]['split'] %}
        {% set remember = namespace(year="0") %}
        {% for key, year, text, bibtex, pdf, slides, poster in publications[bib]['data'] %}
          {% if remember.year != year %}
            {% if remember.year !="0" %}
              </ul>
              {% if publications[bib]['split_link'] %}
                <div style="text-align:right"><i class="fa fa-arrow-up"></i> <a href="#">Back to top</a></div>
              {% endif %}
            {% endif %}
            <h{{ splitheader }}>{{ year }}</h{{ splitheader }}>
            <ul>
            {% set remember.year=year %}
          {% endif %}
          <li style="margin: 5px 0;" id="{{ key }}">
            {{ text }}
            [&nbsp;<a data-toggle="collapse" data-target="#{{ key}}-bib">BibTeX</a>&nbsp;]
            {% for label, target in [('PDF', pdf), ('Slides', slides), ('Poster', poster)] %}
              {{ "[&nbsp;<a href=\"%s\">%s</a>&nbsp;]" % (target, label) if target }}
            {% endfor %}
            <div style="clear:both" id="{{ key }}-bib" class="collapse">
              {% set bibtex = '```tex\n' + bibtex + '```' %}
              {{ bibtex|md }}
            </div>
          </li>
        {% endfor %}
        </ul>
      {% else %}
        <ul>
          {% for key, year, text, bibtex, pdf, slides, poster in publications[bib]['data'] %}
            <li style="margin: 5px 0;" id="{{ key }}">
              {{ text }}
              [&nbsp;<a data-toggle="collapse" data-target="#{{ key }}-bib">BibTeX</a>&nbsp;]
              {% for label, target in [('PDF', pdf), ('Slides', slides), ('Poster', poster)] %}
                {{ "[&nbsp;<a href=\"%s\">%s</a>&nbsp;]" % (target, label) if target }}
              {% endfor %}
              <div style="clear:both" id="{{ key }}-bib" class="collapse">
                {% set bibtex = '```tex\n' + bibtex + '```' %}
                {{ bibtex|md }}
              </div>
            </li>
          {% endfor %}
        </ul>
      {% endif %}
      {% if not loop.last and publications[bib]['bottom_link'] %}
        <div style="text-align:right"><i class="fa fa-arrow-up"></i> <a href="#">Back to top</a></div>
      {% endif %}
    </div>
  {% endif %}
{% endfor %}

<!-- footer part: original bootstrap pages content block -->
        {% if page.comments == 'enabled' %}
            {% include 'includes/comments.html' %}
        {% endif %}
    </div>
</section>

{% endblock %}
```

</details>

---
**NOTE**

This template uses Jinja filter to parse the resulting text with Markdown.

---

To make it work, you will have to include `Markdown` in the `pelicanconf.py` with:

```python
from markdown import Markdown
```

And define the function `md` in `pelicanconf.py` as such:

```python
def md(content, *args):
    return markdown.convert(content)
JINJA_FILTERS = {
    'md': md,
}
```

Extending this plugin
=====================

A relatively simple but possibly useful extension is to make it possible to
write internal links in Pelican pages and blog posts that would point to the
corresponding paper in the Publications page.

A slightly more complicated idea is to support general referencing in articles
and pages, by having some BibTeX entries local to the page, and rendering the
bibliography at the end of the article, with anchor links pointing to the right
place.
