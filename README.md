Note: This repo is largely a snapshop record of bring Wikidata
information in line with Wikipedia, rather than code specifically
deisgned to be reused.

The code and queries etc here are unlikely to be updated as my process
evolves. Later repos will likely have progressively different approaches
and more elaborate tooling, as my habit is to try to improve at least
one part of the process each time around.

---------

Step 1: Check the Position Item
===============================

The Wikidata item for the [Minister for Social
Development](https://www.wikidata.org/wiki/Q55623217) was a bit of a
mess — as was English Wikipedia. 

Enwiki _does_ have a page for it, as "Minister for Social
Development (New Zealand)" but the links from the list of ministers etc
were to "Minister of Social Development (New Zealand)", which in turn
was a redirect to the Ministry page instead.

Possibly because most bots probably weren't finding it properly, the
Wikidata item was fairly bare, and still had the disambiguation-style
label.

So I fixed those up, added country+jurisdiction, and connected it in
both directions to the
[Ministry of Social Development](https://www.wikidata.org/wiki/Q6867545).

Step 2: Tracking page
=====================

Initial PositionHolderHistory list set up at https://www.wikidata.org/w/index.php?title=Talk:Q55623217&oldid=1232928285

Current status: knows of no officeholders (probably because of notes at
#1 above)

Step 3: Set up the metadata
===========================

The first step now in a new repo is always to edit [add_P39.js script](add_P39.js)
to set up the Item ID and source URL.

Step 4: Scrape
==============

Comparison/source = [Minister for Māori Development](https://en.wikipedia.org/wiki/Minister_for_M%C4%81ori_Development)

    wb ee --dry add_P39.js  | jq -r '.claims.P39.references.P4656' |
      xargs bundle exec ruby scraper.rb | tee wikipedia.csv

Scraped cleanly on first pass.

Step 5: Get local copy of Wikidata information
==============================================

    wd ee --dry add_P39.js | jq -r '.claims.P39.value' |
      xargs wd sparql office-holders.js | tee wikidata.json

Wikidata has no information at all about officeholders though, so we'll
need to create a blank file:

    echo '[]' > wikidata.json

Step 6: Create missing P39s
===========================

    bundle exec ruby new-P39s.rb wikipedia.csv wikidata.json |
      wd ee --batch --summary "Add missing P39s, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

27 new additions as officeholders -> https://tools.wmflabs.org/editgroups/b/wikibase-cli/278399c2ae02a

Step 7: Add missing qualifiers
==============================

As there were no members initially, we've nothing to do at this step!

Step 8: Refresh the Tracking Page
=================================
Final version at https://www.wikidata.org/w/index.php?title=Talk:Q55623217&oldid=1232961820

