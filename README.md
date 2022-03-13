# MediaWiki Tools

[![Coverage Status](https://coveralls.io/repos/github/nick-robo/MediaWiki-Tools/badge.svg?branch=main)](https://coveralls.io/github/nick-robo/MediaWiki-Tools?branch=main)

A high level library containing set of tools for for filtering pages using the rich data available in MediaWikis such as categories and info boxes. Uses both web-scraping and API methods (where available and feasible) to gather information.

# Goals

- Generate useful data (and datasets) from a wiki.
- To work on any MediaWiki (including `fandom.com`) with or without api.
- Get arbitrary subsets of pages based on categories and template parameters (todo).
- Be very robust to variations and inconsistencies in user input.
- Be efficient.


# Installation

Install it using pip.

```
pip install mediawiki-tools
```

Requires python `>3.8` because I like the walrus operator.

# Basic Usage

Create `MediaWikiTools` object:

```python
from mediawiki-tools import MediaWikiTools

hp_wiki = MediaWikiTools('harrypotter.fandom.com')
hp_wiki.has_api     # True
```

## Getting pages

Get page names from a category.

```python
hp_wiki.get_pages('1980_births')

#  ['Dudley Dursley',
#  'Neville Longbottom',
#  'Ernest Macmillan',
#  'Draco Malfoy',
#  'Harry Potter',
#  'Ronald Weasley']
```

Get pages from subcategories too.

```python
wiki = MediaWikiTools('en.wikipedia.org')

# get pages from category that contains only subcategories
wiki.get_pages("Art_collectors_by_nationality")
# []

# get pages from first level subcategories
wiki.get_pages("Art_collectors_by_nationality", get_subcats=True)
# ['William Hayes Ackland',
#  'William Acquavella',
#  'Frederick Baldwin Adams Jr.',
#  'Marella Agnelli',
#  'Robert Agostinelli',
#  ...]
```

Recursively get pages from subcategories. This can be quite slow for deep subcategory trees and may break if loops are present.

```python
wiki.get_pages("Art_collectors_by_nationality", 
               get_subcats=True,
               recursive=True)
# ['Very',
#  'Long,
#  'List']
```

Get pages from subcategories as a dictionary containing subcategories as keys. `'self'` contains the pages in the root category. Using `recursive` in combination with the `with_subcats` results in a nested dictionary.

```python
# get first level subcats in dict
wiki.get_pages("Art_collectors_by_nationality", 
               get_subcats=True, with_subcats=True)
# {'self': [],
#  'American art collectors': ['William Hayes Ackland',
#                              'William Acquavella',
#                              'Frederick Baldwin Adams Jr.',
#                              ...],
#   ...
#  'Venezuelan art collectors': ['Gustavo Cisneros',
#                                'Patricia Phelps de Cisneros',
#                                'Nina Fuentes'],
#  'Yugoslav art collectors': ['Antun Bauer (museologist)', 'Erich Šlomović']}
```

## Getting sets

Get an intersection of 2 or more categories.

```python
hp_wiki.get_set(['1980_births', 'Hogwarts_dropouts'], operation='&')
# ['Harry Potter', 'Ronald Weasley']

hp_wiki.get_set(['1980_births', 
                'Hogwarts_dropouts', 
                'Green-eyed_individuals'], 
                operation='intersection')
# ['Harry Potter']
```

Get a union of 2 or more categories.

```python
wiki = MediaWikiTools('en.wikipedia.org')

wiki.get_set(['Countries in Asia', 'Countries_in_Europe'],
	           operation='or')
# ['Cyprus',
#  'Pakistan',
#  'Croatia',
#   ...
#  'Belarus',
#  'Bangladesh',
#  'Lithuania']
```

Chaining operations.

```python
# get the union of countries in Europe and Asia and save it
countries = wiki.get_set(['Countries in Asia', 'Countries_in_Europe'],
	          operation='union')
# intersect the saved list with a different category
wiki.get_set('Russian-speaking_countries_and_territories', 
            operation='&', 
            pages_list=countries)
# ['Kyrgyzstan',
#  'Moldova',
#  'Russia',
#  'Armenia',
#  'Tajikistan',
#  'Belarus',
#  'Azerbaijan',
#  'Uzbekistan',
#  'Mongolia',
#  'Kazakhstan']
```