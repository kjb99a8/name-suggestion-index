## Contributing

### tl;dr

##### :raising_hand: &nbsp; How to help:

* `npm run build` will reprocess the files and output warnings
* Resolve warnings - [show me](#thinking--resolve-warnings)
* Remove generic names - [show me](#hocho--remove-generic-names)
* Add `brand:wikidata` and `brand:wikipedia` tags - [show me](#female_detective--add-wiki-tags)
* Add missing brands - [show me](#convenience_store--add-missing-brands)

##### :no_entry_sign: &nbsp; Don't edit the files in `dist/` - they are generated:

* `dist/allNames.json` - all the frequent names and tags collected from OpenStreetMap
* `dist/discardNames.json` - discarded subset of allNames
* `dist/keepNames.json` - kept subset of allNames

##### :white_check_mark: &nbsp; Do edit the files in `config/`:

* `config/filters.json`- Regular expressions used to filter `allNames` into `keepNames` / `discardNames`
* `config/canonical.json` - Main config file containing all the most correct names and tags to assign to them

&nbsp;

## Background

### :world_map: &nbsp; About OpenStreetMap

[OpenStreetMap](https://openstreetmap.org) is a free, editable map of the whole world that
is being built by volunteers.
Features on the map are defined using _tags_.  Each tag is a `key=value` pair of text strings.

For example, a McDonald's restaurant might have these tags:
```js
  "amenity": "fast_food"
  "cuisine": "burger"
  "name": "McDonald's"
  ... and more tags to record its address, opening hours, and so on.
```

&nbsp;

### :bulb: &nbsp; About the name-suggestion-index

The goal of this project is to define the _most correct tags_ to assign to each common brand name.
This helps people contribute to OpenStreetMap, because they can pick "McDonald's" from a list
and not need to worry about the tags being added.

&nbsp;

### :card_file_box: &nbsp; About `config/canonical.json`

__`config/canonical.json` is our main list of the most correct names and tags.__

This file is created by:
- Processing the OpenStreetMap "planet" data to extract common names -> `dist/allNames.json`
- Filtering all the names into -> `dist/keepNames.json` and `dist/discardNames.json`
- Merging the names we are keeping into -> `config/canonical.json` for us to decide what to do with them

Each entry looks like this _(comments added for clarity)_:


```js
  "amenity/fast_food|McDonald's": {         // Identifier like "key/value|name"
    "count": 19040,                         // `count` - generated # of these that we found in OpenStreetMap
    "match": [
      "amenity/fast_food|Mc Donald's",      // Optional `match` array
      "amenity/fast_food|McDonalds",        //   Contains less-desirable variations to ignore.
      "amenity/restaurant|Mc Donald's",     //   (we want to keep only "amenity/fast_food|McDonald's")
      "amenity/restaurant|Mc Donalds",      //
      "amenity/restaurant|McDonald's",      //
      "amenity/restaurant|McDonalds",       //
    ],
    "tags": {                               // "tags" - OpenStreetMap tags that every McDonald's should have
      "amenity": "fast_food",               //   The OpenStreetMap tag for a "fast food" restaurant
      "brand": "McDonald's",                //   `brand` - Brand name in the local language (English)
      "brand:wikidata": "Q38076",           //   `brand:wikidata` - Universal Wikidata identifier
      "brand:wikipedia": "en:McDonald's",   //   `brand:wikipedia` - Reference to English Wikipedia
      "cuisine": "burger",                  //   `cuisine` - What kind of fast food is served here
      "name": "McDonald's"                  //   `name` - Display name, also in the local language (English)
    }
  },
```

There may also be entries for McDonald's in other languages!

```js
  "amenity/fast_food|??????????????????": {         // Identifier like "key/value|name"
    "count": 1445,
    "countryCodes": ["jp"],                 // Optional `countryCodes` - array of countries where this entry is used
    "tags": {
      "amenity": "fast_food",
      "brand": "??????????????????",                // `brand` - Brand name in the local language (Japanese)
      "brand:en": "McDonald's",             // `brand:en` - For non-English brands, tag the English version too
      "brand:wikidata": "Q38076",           // `brand:wikidata` - Same Universal wikidata identifier
      "brand:wikipedia": "ja:??????????????????",   // `brand:wikipedia` - Reference to Japanese Wikipedia
      "cuisine": "burger",
      "name": "??????????????????",                 // `name` - Display name, also in the local language (Japanese)
      "name:en": "McDonald's"               // `name:en` - For non-English names, tag the English version too
    }
  },
```

&nbsp;

#### Optional properties to suppress warnings

These properties always start with `"no"`.

##### `nomatch`

Sometimes there are multiple different entries that use the same name.

For example, "Sonic" can be either a fast food restaurant or a fuel station.

We want to allow both kinds of "Sonic" to exist in the index, and we don't want
to be warned that they are potentially duplicate, so we can add a `nomatch`
property to each entry to suppress the "duplicate name" warning.

```js
  "amenity/fuel|Sonic": {
    "count": 127,
    "nomatch": ["amenity/fast_food|Sonic"],
    ...
  }
  "amenity/fast_food|Sonic": {
    "count": 1110,
    "nomatch": ["amenity/fuel|Sonic"],
    ...
  }
```

##### `nocount`

Sometimes a new, uncommon OpenStreetMap tag-name combination should replace an
existing tag-name combination.

For example, many Costcos are tagged as `shop=department_store` or `shop=supermarket`,
but the new preferred tag is `shop=wholesale`.

We don't want to be warned that the new correct tag is uncommon, so we add a
`nocount` property to the entry to suppress the "uncommon name" warning.

```js
  "shop/wholesale|Costco": {
    "nocount": true,
    "match": [
      "shop/department_store|Costco",
      "shop/department_store|Costco Wholesale",
      "shop/supermarket|Costco",
      "shop/supermarket|Costco Wholesale"
    ],
    ...
  },
```

&nbsp;


#### Identical names, multiple brands

Sometimes multiple brands with the same name will operate in geographically
distinct locations.  You can modify the key to include a tilde `~` after the name
to tell the difference between two otherwise identical brands.
The text after the tilde can contain anything.

When using a tilde `~` name:
* You should add `"nocount": true`, as the script will not be able to determine the true count.
* You should add `"nomatch":` properties to each name so they do not generate duplicate name warnings.


```js
  "shop/supermarket|Price Chopper~(Kansas City)": {
    "countryCodes": ["us"],
    "nocount": true,
    "nomatch": [
      "shop/supermarket|Price Chopper~(New York)"
    ],
    "tags": {
      "brand": "Price Chopper",
      "brand:wikidata": "Q7242572",
      "brand:wikipedia": "en:Price Chopper (supermarket)",
      "name": "Price Chopper",
      "shop": "supermarket"
    }
  },
  "shop/supermarket|Price Chopper~(New York)": {
    "countryCodes": ["us"],
    "nocount": true,
    "nomatch": [
      "shop/supermarket|Price Chopper~(Kansas City)"
    ],
    "tags": {
      "brand": "Price Chopper",
      "brand:wikidata": "Q7242574",
      "brand:wikipedia": "en:Price Chopper Supermarkets",
      "name": "Price Chopper",
      "shop": "supermarket"
    }
  },
```


&nbsp;

## What you can do

### :building_construction: &nbsp; Building the project

To rebuild the index, run:
* `npm run build`

This will output a lot of warnings, which you can help fix!

&nbsp;

### :thinking: &nbsp; Resolve warnings

Warnings mean that you need to edit `config/canonical.json`.
The warning output gives a clue about how to fix or suppress the warning.
If you aren't sure, just ask on GitHub!

&nbsp;

#### Duplicate names

```
Warning - Potential duplicate names in `canonical.json`:
To resolve these, remove the worse entry and add "match" property on the better entry.
To suppress this warning for entries that really are different, add a "nomatch" property on both entries.
  "tourism/motel|Motel 6" -> duplicates? -> "tourism/hotel|Motel 6"
```

_What it means:_  "Motel 6" exists in the index twice - as both a `tourism=hotel` (wrong)
and a `tourism=motel` (correct). In this situation we want to:
* Delete the entry for `"tourism/hotel|Motel 6"` and
* Add `"match": ["tourism/hotel|Motel 6"]` to the `"tourism/motel|Motel 6"` entry

Usually the entry which is used more frequently (indicated by "count" property)
is the one to keep.  If you are not sure, you can also search on the
[OpenStreetMap Wiki](https://wiki.openstreetmap.org/wiki/Map_Features) for tag best practices.

&nbsp;

#### Uncommon names

```
Warning - Uncommon entries in `canonical.json` not found in `keepNames.json`:
These might be okay. It just means that the entry is not commonly found in OpenStreetMap.
To suppress this warning, add a "nocount" property to the entry.
  "shop/wholesale|Costco"
```

_What it means:_  This warning can occur if the index contains an entry that is not common in OpenStreetMap.
Either replace it with a more common tag, or add `"nocount": true` to suppress the warning.

(In this situation, `shop=wholesale` is a new preferred tag, but most existing Costcos were
still tagged as `shop=department_store`. Suppressing the warning is the correct thing to do).

&nbsp;

### :hocho: &nbsp; Remove generic names

Some of the common names in the index might not actually be brand names. We want to remove these
generic words from the index, so they are not suggested to mappers.

For example, "??????????????????" is just a Russian word for "Department store":

```js
  "shop/department_store|??????????????????": {
    "count": 210,
    "tags": {
      "brand": "??????????????????",
      "name": "??????????????????",
      "shop": "department_store"
    }
  },
```

To remove this generic name:
1. Delete the entries from `config/canonical.json`
2. Edit `config/filters.json`. Add a regular expression matching the generic name in either the `discardKeys` or `discardNames` list.
3. Run `npm run build` - if the filter is working, the name will not be put back into `config/canonical.json`
4. `git diff` - to make sure that the entries you wanted to discard are gone (and no others are affected)
5. If all looks ok, submit a pull request with your changes.

&nbsp;

### :female_detective: &nbsp; Add wiki tags

Adding `brand:wikipedia` and `brand:wikidata` tags is a very useful task that anybody
can help with.

#### Example #1 - Worldwide / English brands...

1. Find an entry in `config/canonical.json` that is missing these tags:

```js
  "amenity/fast_food|Chipotle": {
    "count": 708,
    "match": [
      "amenity/fast_food|Chipotle Mexican Grill",
      "amenity/restaurant|Chipotle",
      "amenity/restaurant|Chipotle Mexican Grill"
    ],
    "tags": {
      "amenity": "fast_food",
      "brand": "Chipotle",
      "cuisine": "mexican",
      "name": "Chipotle"
    }
  },
```

2. Google for that brand - if you are lucky, you might find the Wikipedia page right away.

<img width="600px" alt="Google for Chipotle" src="https://raw.githubusercontent.com/osmlab/name-suggestion-index/master/docs/chipotle_1.png"/>

3. From the Wikipedia page URL, you can identify the `brand:wikipedia` value.

OpenStreetMap expects this tag to be formatted like `"en:Chipotle Mexican Grill"`.
* Copy the page name from the URL.
* Add the language prefix "en:".
* Replace the underscores '_' with spaces.

On the brand's Wikipedia page, you can also find its "Wikidata item" link.  This appears
under the "tools" menu in the sidebar.

:point_right: protip: [@maxerickson] has created a user script to make copying these values even easier - see [#1881]

[#1881]: https://github.com/osmlab/name-suggestion-index/issues/1881
[@maxerickson]: https://github.com/maxerickson

<img width="600px" alt="Chipotle Wikipedia" src="https://raw.githubusercontent.com/osmlab/name-suggestion-index/master/docs/chipotle_2.png"/>

4. On the brand's Wikidata page, you can identify the `brand:wikidata` value.  It is a code starting with 'Q' and several numbers.

<img width="600px" alt="Chipotle Wikidata" src="https://raw.githubusercontent.com/osmlab/name-suggestion-index/master/docs/chipotle_3.png"/>

5. Update `config/canonical.json`:

We can add the `"brand:wikipedia"` and `"brand:wikidata"` tags.

```js
  "amenity/fast_food|Chipotle": {
    "count": 708,
    "match": [
      "amenity/fast_food|Chipotle Mexican Grill",
      "amenity/restaurant|Chipotle",
      "amenity/restaurant|Chipotle Mexican Grill"
    ],
    "tags": {
      "amenity": "fast_food",
      "brand:wikidata": "Q465751",                            // added
      "brand:wikipedia": "en:Chipotle Mexican Grill",         // added
      "brand": "Chipotle",
      "cuisine": "mexican",
      "name": "Chipotle"
    }
  },
```

_(comments added for clarity)_

6. Rebuild and submit a pull request.

* Run `npm run build`
* If it does not fail with an error, you can submit a pull request with your changes.

&nbsp;

#### Example #2 - Regional / non-English brands...

This example uses a brand "???????????????".  I don't know what that is, so I will do some research.

1. Find an entry in `config/canonical.json`:

```js
  "amenity/fast_food|???????????????": {
    "count": 91,
    "tags": {
      "amenity": "fast_food",
      "brand": "???????????????",
      "name": "???????????????"
    }
  },
```

2. Google for that brand - if you are lucky, you might find the Wikipedia page right away.

Tip: You might want to narrow you search by Googling with a `site:` filter:  `"??????????????? site:ja.wikipedia.org"`

From these results, we can know that the brand is "Kappazushi", owned by a Japanese company
called "Kappa Create".  We can also find the Wikipedia page.

<img width="600px" alt="Google for ???????????????" src="https://raw.githubusercontent.com/osmlab/name-suggestion-index/master/docs/kappa_1.png"/>

3. As with English brands, you can identify the `brand:wikipedia` value from the URL.
Because this is a Japanese brand, we will link to the Japanese Wikipedia page.

OpenStreetMap expects this tag to be formatted like `"ja:???????????????"`.
* Copy the page name from the URL.
* Add the language prefix "ja:".
* Replace the underscores '_' with spaces.

Although I can not read Japanese, I can identify the "Wikidata item" link because
it always appears in the sidebar and mouseover will show the Wikidata 'Q' code in the URL.

<img width="600px" alt="Kappa Sushi Wikipedia" src="https://raw.githubusercontent.com/osmlab/name-suggestion-index/master/docs/kappa_3.png"/>

4. On the brand's Wikidata page, you can identify the `brand:wikidata` value.  It is a code starting with 'Q' and several numbers.

Note: The Wikidata page looks a bit sparse - you can edit this too if you want to help!

<img width="600px" alt="Kappa Sushi Wikidata" src="https://raw.githubusercontent.com/osmlab/name-suggestion-index/master/docs/kappa_4.png"/>

5. Update `config/canonical.json`:

We can add:
* `"brand:en"` and `"name:en"` tags to contain the English name "Kappazushi"
* `"brand:wikipedia"` and `"brand:wikidata"` tags.
* `"cuisine": "sushi"` OpenStreetMap tag
* `"countryCodes"` property, to indicate that this brand should only be used in Japan.

```js
  "amenity/fast_food|???????????????": {
    "count": 91,
    "countryCodes": ["jp"],                // added
    "tags": {
      "amenity": "fast_food",
      "brand": "???????????????",
      "brand:en": "Kappazushi",            // added
      "brand:wikipedia": "ja:???????????????",     // added
      "brand:wikidata": "Q11263916",        // added
      "cuisine": "sushi",                   // added
      "name": "???????????????",
      "name:en": "Kappazushi"              // added
    }
  },
```

_(comments added for clarity)_

6. Rebuild and submit a pull request.

* Run `npm run build`
* If it does not fail with an error, you can submit a pull request with your changes.

&nbsp;

### :convenience_store: &nbsp; Add missing brands

If it exists, we want to know about it!

Some brands aren't mapped enough (50+ times) to automatically be added to the index so this
is a valuable way to get ahead of incorrect tagging.

1. Before adding a new brand, the minimum information you should know is the correct tagging required for instances of the brand (`name`, `brand` and what it is - e.g. `shop=food`). Ideally you also have `brand:wikidata` and `brand:wikipedia` tags for the brand and any other appropriate tags - e.g. `cuisine`.

2. Add your new entry anywhere in `config/canonical.json` (it will be sorted alphabetically later) and using the `"tags"` key add all appropriate OSM tags. Refer to [here](#card_file_box--about-configcanonicaljson) if you're not familiar with the syntax.

3. Add the `"nocount": true` key to your new entry as this will suppress the warnings that it doesn't appear often in OSM.

4. If the brand only has locations in a known set of countries add the `"countryCodes": []` key to your new entry. This takes an array of [ISO 3166-1 alpha-2 country codes](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) in lowercase (e.g. `["de", "at", "nl"]`).

5. If instances of this brand are commonly mistagged add the `"match": []` key to list these. Again, refer to [here](#card_file_box--about-configcanonicaljson) for syntax.

6. Run `npm run build` and resolve any [duplicate name warnings](#thinking--resolve-warnings).

&nbsp;

#### Using Overpass Turbo

Sometimes you might want to know the locations where a brand name exists in OpenStreetMap.
Overpass Turbo can show them on a map:

1. Go to https://overpass-turbo.eu/

2. Enter your query like this, replacing the `name` and other OpenStreetMap tags.
Because we don't specify a bounding box, this will perform a global query.

```
nwr["name"="???????????????"]["amenity"="fast_food"];
out center;
```

3. Click run to view the results.

As expected, the "???????????????" (Kappazushi) locations are all concentrated in Japan.

<img width="600px" alt="Overpass search for ???????????????" src="https://raw.githubusercontent.com/osmlab/name-suggestion-index/master/docs/overpass.png"/>

