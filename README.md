# Multilingual Jekyll Websites (the easy way, no plug-ins)

This is a straight-to-the-point tutorial (or rather my personal notes) on how create a multilingual website using Jekyll **without any plug-ins** (and therefore no dependencies, which means no chance of things suddenly breaking in the future!)

Version 3 / Updated: Dec 2022

## Folder Structure
Suppose we want a website with content in both English (en) and German (de). We want every English page to have a German counterpart and vice versa.

**Structure your folder as follows** (note: the below structure does not include all the necessary files and folders that Jekyll needs to work, but **only the ones relating to making the site multilingual**. [Read about Jekyll’s structure in the official docs](https://jekyllrb.com/docs/structure/)):

```
- _data
  - index.yml
  - page1.yml
  - page2.yml
- _layouts
  - index.html
  - page1.html
  - page2.html
- de
  - index.html
  - page1.html
  - page2.html
- en
  - index.html
  - page1.html
  - page2.html
- index.html
```

### _data
Contains one YAML file for each page. The yml file has the text in all languages. It looks like this:

```yml
hero:
  title:
    en: This is the English title
    sk: Das ist der deutsche Title
```

### _layouts
Contains one HTML file for each page. (And any other layouts we want, see the Jekyll docs for that.)
Inside the file, wherever we want to put text, we reference a line from the corresponding YAML file. For example, to reference “This is the English title” from above, we would write `{{site.data.index.hero.title[page.lang]}}` (I’ll explain `[page.lang]` below)
To avoid repeating the `site.data.index` part for every line of text we want to pull, we’ll add the following to the top of the HTML file, before any content:

```liquid
{% assign data = site.data.index %}
```

Then, instead of `{{site.data.index.hero.title[page.lang]}}` we can write `{{data.hero.title[page.lang]}}`

### Language folders (e.g. en, de)
Contain one HTML file for each page. Each file’s front-matter looks like this:

```liquid
---
layout: index
title: My Homepage
lang: en
ref: index
---
```

Notice the `lang` attribute. That’s what `[page.lang]` from above references. All files in the "en" folder will have "lang: en" and all files in the "de" folder, "lang: de".

Then we can create if statements that insert content based on the page’s `lang` (from the front-matter).

For example, in our navigation include we’ll use this:

```liquid
{% if navlink in site.data.navlinks[page.lang] %}
  {% for navlink in site.data.navlinks[page.lang] %}
    <li {% if page.url contains '/{{navlink.title}}' %}class="active"{% endif %}>
      <a href="/{{navlink.url}}/">{{navlink.title}}</a>
    </li>
  {% endfor %}
{% endif %}
```

Assuming the existence of:
```
- _data
  - navlinks.yml
```

#### What does "ref" do?
Following this method, you’ll have the same page in multiple language folders. When the user clicks on a “switch language” link, we want them to be directed to the *same* page in a *different* folder.

Say you are in an English page. The button thn changes the language to German must be structured like `/de/{{ page.ref }}` where `ref` is the unique front-matter ref of each page AND the filename of the page. That ensures that, upon clicking the “change language” button, the user is taken to the same page in their language of choice, instead of being taken back to the homepage no matter where they currently are on the site.

## End Result
By having separate folders for each language (in this case de and en) we automatically get URLs that look like this: `https://example.com/en/page1` for all our English pages, and like this: `https://example.com/de/page1` for all our German pages.

* **Note**: we could have done that using other methods (e.g. permalink attribute in the front-matter of every single page) but I think it’s easier and way more organized and easy to visualize the structure of the website when everything is in folders.
* **Note 2**: to remove the ugly `.html` from the website’s URLs, just include the line `permalink: pretty` anywhere in the `_config.yml`!

## Default Language
You probably noticed that there’s one more instance of `index.html` than there are instances of the rest of the pages. **The** `index.html` **in the root directory will dictate your website’s default language.** Meaning it will be a duplicate of the `index.html` in the en folder if you want the default language to be English.

## Links
To make links work right, assuming that pages are grouped into folders by language, the links have to look like this to work:
`/{{ page.lang }}/page-name` instead of just `/page-name/`
What this does, is tell the browser to direct the user to `page-name.html` WITHIN the same language folder (since every page has a `lang` attribute in its front-matter.)