# api-doc

## A REST API

This design is thought to be used as an implementation of a REpresentational State Transfer ([REST](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)) API. The present definition is intended to serve as both an example and the documentation for a particular case of this API document structure.

## Documents format

Every document will be expressed using the JavaScript Object Notation (*JSON*) for the sake of readability and expressiveness, but it is fairly trivial to translate all the examples to another format like *XML* or even *HTML*.

## Hypermedia and common attributes

All documents (hence, all resources) must use hypermedia links to reference external relations or complimentary resources that do not strictly belong to the set of own attributes. Aside from that, all documents must include a link to themselves under the `self` relation key.

Links for any document or resource must be defined inside the `links` attribute, with a key identifying the kind of relation that is being defined with that particular link (the **relation key**) and a horizontal reference to the linked document on the `href` attribute. In extension, any link must respect the following structure:

```json
{
    "links": {
        "relationKey": {
            "href": "/url/to/linked/resource"
        }
    }
}
```

As this is a minimal definition of a link, any other attributes might be added if needed as long as this basic structure is still maintained.

The following relation keys are reserved:

- `self`: indicates the link which can be followed in order to get the same resource again. As stated before, this relation key will be present in every document and should contain any parameters passed to obtain the current resource.
- `previous`: this relation indicates the link that should be followed in order to get the previous page on a paginated collective response. This will only be present when the current collective document has a previous page.
- `next`: this relation indicates the link that should be followed in order to get the next page on a paginated collective response. This will only be present when the current collective document has a next page.
- `full`: this relation represents the link that needs to be used in order to retrieve the full version of a partial response document. Please refer to the dedicated section for further details.

## Collective documents

Any collection (as in "a group of items") must respond with a document like the one below. Key elements will be explained after the example:

```json
{
    "entries": [
        {
            "links": {
                "self": {
                    "href": "/some/item.json"
                }
            }
        },
        {
            "links": {
                "self": {
                    "href": "/some/other-item.json"
                }
            }
        }
    ],

    "offset": 0,
    "limit": 10,
    "total": 2,

    "links": {
        "self": {
            "href": "/link/to/self.json"
        }
    }
}
```

* `entries`: An `array` of items, where each item is a link to the actual collection element.
* `offset`: A `0`-based `integer` index indicating the offset in the collection of elements from which the current page is taking elements. This is the base index for the current span of elements (page).
* `limit`: An `integer` indicating the maximum number of elements that will be included in the current span of elements (page). A sensible default value should be considered for this attribute.
* `total`: An `integer` indicating the total number of elements available in the collection.

As collections represent resource types or logical groupings of resources, they should be accessed in simple URIs like:

    /<resource-type-in-plural>.json

For instance, if we wanted to reference the people in a data set, we would use the following URI:

    /people.json

Pagination is achieved by passing an `offset` parameter in the URLs (and optionally a `limit`):

    /people.json?offset=15
    /people.json?offset=20&limit=20

If a collective document has to paginate (i.e. it has more elements than the value of `limit`), it must include up to 2 more links (besides the `self`-pointing one) that will allow for easy navigation of the paginated resources: `previous` and/or `next`.

* If the current `offset` is such that the page is other than the first one, a `previous` link must be provided with the `offset` set to the one corresponding to the previous page and preserving any user-provided additional [resource modifiers](#resource-modifiers), as the `limit`.
* If the current `offset` is such that the page is not the last one in the collection, a `next` link must be provided with the `offset` set to the one corresponding to the next page and preserving any user-provided additional [resource modifiers](#resource-modifiers), as the `limit`.

Here is a complete example which shows a combination of both of the situations described above:

```json
{
    "entries": [
        ...
    ],

    "offset": 15,
    "limit": 15,
    "total": 33,

    "links": {
        "self": {
            "href": "/people.json?offset=15&limit=15"
        },
        "previous": {
            "href": "/people.json?offset=0&limit=15"
        },
        "next": {
            "href": "/people.json?offset=30&limit=15"
        }
    }
}
```

## Individual documents

Any individual resource will be defined in a document with its properties, its `self`-pointing link and an optional set of complimentary documents links, like follows:

```json
{
    "id": "1",
    "anAttribute": "some value for an attribute",
    "isValid": true,

    "links": {
        "self": {
            "href": "/link/to/self.json"
        },
        "items": {
            "href": "/link/to/related/items.json"
        },
        "parent": {
            "href": "/link/to/parent.json"
        }
    }
}
```

The only mandatory attribute of this example is the `links` key along with the `self` link. Any other value is just for explanatory purposes.

Usually, individual documents will be referenced via a simple URI like:

    /<resource-type>/<resource-identifier>.json

For instance, a particular book (belonging to the `books` resource type) could be referenced via its ISBN-10 value with the following URI:

    /books/1449310508.json

## Resource Modifiers

Any argument that would modify the current document but which doesn't identify it, is called a **resource modifier** and must be provided as a query string parameter (i.e. after the `?` in the URL).

A typical example for this could be the `limit` and `offset` arguments.

## Types

Field types must be consistently represented. Following is a list of the standard representation of the different types that may be found on a document:

### Numbers

Numbers can be represented as `integer` elements (`0`, `200`, `4019`), `float` elements (`0.5`, `3.0`, `68622.42`) or as arbitrary precision and longitude numbers which are represented by strings (`8126318281987921632167`, `921763.8126326178321531245632571`).

### Identifiers

Primary Key identifiers must always be represented as `string`, even if they are not big numbers: `1`, `105`, `18291278321678421678321678321`. This ensures consistency accross documents and resource types.

### Boolean

`boolean` values must be represented by `true` or `false` and nothing else. Even if they could be coerced into `boolean` values, using only `true` or `false` will state correctly the fact that the field is a `boolean` value.

### String

`string` values must be encoded using Unicode, as defined in the [JSON specification](http://www.json.org/) using `\uXXXX` escaped characters when needed.

### Date

Dates must be expressed as a `string` in the [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601) standard format: `YYYY-MM-DD` (`YYYY` for a 4-digit year, `MM` for a 2-digit month and `DD` for a 2-digit day of the month). For example, `"2012-12-31"`, `"2013-01-01"`, `"2013-02-28"`.

### Time

Time values must be represented with a `string` in the [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601) standard format: `hh:mm[:ss]` (`hh` for a 2-digit 24-hour part, `mm` for a 2-digit minutes part and an optional `ss` for a 2-digit seconds part). For instance, `"20:03"`, `"00:00:00"` and `"05:31:58"` are valid time values.

### Timestamp

Full timetamp values must be represented with a `string` formatted in the [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601) standard:

    YYYY-MM-DDThh:mm[:ss]
    <  DATE  >T<  TIME  >

Where the `T` is a literall that denotes the separation between both timestamp parts.

`TBD` Is it of value to specify the Timezone in the timestamps?

### URIs and URLs

URIs and URLs must be relative - i.e. no host should be present in them.

This is a **valid** URI:

    /api/books/5678123.json

While this **isn't valid**:

    http://server.example.com/api/books/5678123.json

### Links expansion

By providing the `expand` [resource modifier](#resource-modifiers), the linked resources of a document could also be retrieved in a single request to the server. In a quick overview, the `expand` attribute indicates which links should also be retrieved and injected into the document as new attributes.

#### Syntax

Links to be expanded can be specified with a comma-separated list of values, where each value represents a key under the `links` section in a given scope. Non-existing keys will be ignored and yield no error. The only exception to this is the `entries` key which will match the `entries` field in any collective document, and will replace **in the same place** the contents of the `entries` field with the expanded entries.

Expansion scopes are enclosed by parentheses (`(` and `)`), and they represent the means to recursive link expansion. If the need to recursively expand links should exist, we just need to group the expansions in the same level with parentheses, like this:

    GET /books.json?expand=entries(self(author,publisher))

The above request will get the `books` resources and expand the `entries` via their `self` links, and in each sub-resource ("each book", a sub-scope) it will expand the `author` and `publisher` links (if each of them exists), adding two new properties to the "book": `author` and `publisher`. Please refer to the example in this section for further details into this.

Deeper recursive expansion can be achieved by adding more scopes. For instance:

    GET /authors.json?expand=entries(self(publisher(books(entries(self)))))

This will result in some response like follows - for the sake of simplicity the actual fields are omitted, as we are focusing on the structure of the response:

    GET /authors.json?expand=entries(self(publisher(books(entries(self)))))

```json
{
    "entries": [
        {
            "id": "...",
            "name": "...",
            "publisher": {
                "id": "...",
                "name": "...",
                "books": [
                    {
                        "id": "...",
                        "isbn10": "...",
                        "isbn13": "...",
                        "links": {
                            "self": {
                                "href": "..."
                            },
                            "author": {
                                "href": "..."
                            },
                            "publisher": {
                                "href": "..."
                            }
                        }
                    },
                    {
                        "id": "...",
                        "isbn10": "...",
                        "isbn13": "...",
                        "links": {
                            "self": {
                                "href": "..."
                            },
                            "author": {
                                "href": "..."
                            },
                            "publisher": {
                                "href": "..."
                            }
                        }
                    }
                ]
            }
        }
    ]
}
```

#### Example

If we were to get a book, its author and publishing company, **without links expansion**, we would need to make 3 requests:

    GET /books/1449310508.json

```json
{
    "id": "1449310508",
    "isbn10": "1449310508",
    "isbn13": "978-1449310509",
    "title": "REST API Design Rulebook",
    "language": "English",
    "rating": 2.6,
    "publishedAt": "2011-10-28",

    "links": {
        "self": {
            "href": "/books/1449310508.json"
        },
        "author": {
            "href": "/authors/B005WVDZOU.json"
        },
        "publisher": {
            "href": "/publishers/DJSA3217.json"
        }
    }
}
```

    GET /authors/B005WVDZOU.json

```json
{
    "id": "B005WVDZOU",
    "name": "Mark Masse",
    "bio": "Mark Masse resides in Seattle, where he is a Senior Director of Engineering at ESPN.",

    "links": {
        "self": {
            "href": "/authors/B005WVDZOU.json"
        },
        "books": {
            "href": "/authors/B005WVDZOU/books.json"
        }
    }
}
```

    GET /publishers/DJSA3217.json

```json
{
    "id": "DJSA3217",
    "name": "O'Reilly Media",

    "links": {
        "self": {
            "href": "/publishers/DJSA3217.json"
        },
        "books": {
            "href": "/publishers/DJSA3217/books.json"
        }
    }
}
```

But **with links expansion**, we would only need one request:

    GET /books/1449310508.json?expand=author,publisher

```json
{
    "id": "1449310508",
    "isbn10": "1449310508",
    "isbn13": "978-1449310509",
    "title": "REST API Design Rulebook",
    "language": "English",
    "rating": 2.6,
    "publishedAt": "2011-10-28",

    "author": {
        "id": "B005WVDZOU",
        "name": "Mark Masse",
        "bio": "Mark Masse resides in Seattle, where he is a Senior Director of Engineering at ESPN.",

        "links": {
            "self": {
                "href": "/authors/B005WVDZOU.json"
            },
            "books": {
                "href": "/authors/B005WVDZOU/books.json"
            }
        }
    },

    "publisher": {
        "id": "DJSA3217",
        "name": "O'Reilly Media",

        "links": {
            "self": {
                "href": "/publishers/DJSA3217.json"
            },
            "books": {
                "href": "/publishers/DJSA3217/books.json"
            }
        }
    },

    "links": {
        "self": {
            "href": "/books/1449310508.json?expand=author,publisher"
        },
        "author": {
            "href": "/authors/B005WVDZOU.json"
        },
        "publisher": {
            "href": "/publishers/DJSA3217.json"
        }
    }
}
```

You may notice that the new root attributes `author` and `publisher` correspond to the relation keys indicated in the `expand` resource modifier, and that their content is the response for each link.

### Partial responses

When not all the data in a document is needed, a partial response can help avoiding some unnecessary overhead by using the `fields` [resource modifier](#resource-modifiers) to indicate the only elements needed from the document.

For example, if we were asking for the author document shown above (`/authors/B005WVDZOU.json`) but we only cared for the name of the author, we could make the following request to get a partial response containing only the data we needed:

    GET /authors/B005WVDZOU.json?fields=name

```json
{
    "name": "Mark Masse",

    "links": {
        "self": {
            "href": "/authors/B005WVDZOU.json?fields=name"
        },
        "full": {
            "href": "/authors/B005WVDZOU.json"
        }
    }
}
```

You may notice that the `links` attribute is still included, even when not explicitly requested, but it only includes two relations: `self` (as usual) and `full`, which links to the full resource.

## About this example

This example is intended to model a basic structure.

### Resources

The following resources are available:

    /articles.json
    /editions.json
    /sections.json
    /supplements.json
    /tags.json

They are the collective resources that support the `GET` HTTP verb and return a paginated collection of items, following the structure described in the [Collective documents section](#collective-documents).
