# Microdata and JSON-LD
Recently, [Markus Lanthaler](http://www.markus-lanthaler.com) [asked on the WHATWG mailing list](http://lists.w3.org/Archives/Public/public-whatwg-archive/2012Aug/0073.html) if [JSON-LD](http://www.w3.org/TR/json-ld-syntax/) had been considered as the normative serializaition format, in preference to the microdata-specific JSON format. The W3C HTML5 Working Group recently submitted an [application](http://www.ietf.org/mail-archive/web/ietf-types/current/msg01714.html) to register `application/microdata+json`, which is defined in the [HTML Microdata specification](http://www.w3.org/TR/microdata/#json).

An example of this can be seen in extracting JSON from the following HTML+microdata snippet:

    <div itemscope itemtype="http://schema.org/Person">
      <span itemprop="name">Jane Doe</span>
      <img src="janedoe.jpg" itemprop="image" alt="">

      <span itemprop="jobTitle">Professor</span>
      <div itemprop="address" itemscope itemtype="http://schema.org/PostalAddress">
        <span itemprop="streetAddress">
          20341 Whitworth Institute
          405 N. Whitworth
        </span>
        <span itemprop="addressLocality">Seattle</span>,
        <span itemprop="addressRegion">WA</span>
        <span itemprop="postalCode">98052</span>
      </div>
      <span itemprop="telephone">(425) 123-4567</span>
      <a href="mailto:jane-doe@xyz.edu" itemprop="email">
        jane-doe@xyz.edu</a>

      Jane's home page:
      <a href="www.janedoe.com" itemprop="url">janedoe.com</a>

      Graduate students:
      <a href="www.xyz.edu/students/alicejones.html" itemprop="colleagues">
        Alice Jones</a>
      <a href="www.xyz.edu/students/bobsmith.html" itemprop="colleagues">
        Bob Smith</a>
    </div>

This snippet using the [schema.org][http://schema.org/] vocabulary to define a person, with name, address and relationship information. Transforming this to JSON microdata yields the following:

    {
      "items": [
        {
          "type": ["http://schema.org/Person"],
          "properties": {
            "name": ["Jane Doe"],
            "image": ["http://example.com/janedoe.jpg"],
            "jobTitle": ["Professor"],
            "address": [
              {
                "type": ["http://schema.org/PostalAddress"],
                "properties": {
                  "streetAddress": [
                    "\n      20341 Whitworth Institute\n      405 N. Whitworth\n    "
                  ],
                  "addressLocality": ["Seattle"],
                  "addressRegion": ["WA"],
                  "postalCode": ["98052"]
                }
              }
            ],
            "telephone": ["(425) 123-4567"],
            "email": ["mailto:jane-doe@xyz.edu"],
            "url": ["http://example.com/www.janedoe.com"],
            "colleagues": [
              "http://example.com/www.xyz.edu/students/alicejones.html",
              "http://example.com/www.xyz.edu/students/bobsmith.html"
            ]
          }
        }
      ]
    }

Basically, microdata JSON does a straightforward serialization of microdata encoded HTML into a JSON format, principally to support copy-and-paste and drag-and-drop payloads (see [Hixie's](http://ian.hixie.ch) [response](http://lists.w3.org/Archives/Public/public-whatwg-archive/2012Aug/0080.html)). The argument is that having a special mime-type helps processors trying to distinguish a drag-and-drop payload from any other JSON paload in dropzone filtering.

However, what this misses, is that microdata is, principally, a metadata format, used almost exclusively for encoding [schema.org][http://schema.org/] classes and properties.

## Microdata as RDF
Late in 2011, the W3C Technical Architecture Group launched an HTML Data Task Force to examine metadata in HTML in general, and more specifically to come up with a normative mechanism for translating microdata into [RDF](http://www.w3.org/RDF/). The result of this is the W3C Semantic Working Group note on [Microdata to RDF](http://www.w3.org/TR/microdata-rdf/).

Given that extracing RDF from HTML microdata is something that is important to developers, it can be inferred that extracing RDF from JSON microdata is also of interest, however one of the [criticisms](http://www.jenitennison.com/blog/node/164) of the JSON microdata format is that it lacks semantic fidelity, principally loosing language and datatype information for literal values, and confusing IRIs (URLs) with being literal values instead of object references.

In contrast, [JSON-LD]() provides full support for literals with data-types and languages as well as distinguishing object (or node) references from literals. The above example could be written in JSON-LD as follows:

    {
      "@context": {
        "@vocab": "http://schema.org/",
        "items": "@graph",
        "type": "@type",
        "image": {"@id": "http://schema.org/image", "@type": "@id"},
        "address": {"@id": "http://schema.org/address", "@type": "@id"},
        "email": {"@id": "http://schema.org/email", "@type": "@id"},
        "url": {"@id": "http://schema.org/url", "@type": "@id"},
        "colleagues": {"@id": "http://schema.org/colleagues", "@type": "@id"},
      },
      "items": [
        {
          "type": ["http://schema.org/Person"],
          "name": ["Jane Doe"],
          "image": ["http://example.com/janedoe.jpg"],
          "jobTitle": ["Professor"],
          "address": [
            {
              "type": ["http://schema.org/PostalAddress"],
              "streetAddress": [
                "\n      20341 Whitworth Institute\n      405 N. Whitworth\n    "
              ],
              "addressLocality": ["Seattle"],
              "addressRegion": ["WA"],
              "postalCode": ["98052"]
            }
          ],
          "telephone": ["(425) 123-4567"],
          "email": ["mailto:jane-doe@xyz.edu"],
          "url": ["http://example.com/www.janedoe.com"],
          "colleagues": [
            "http://example.com/www.xyz.edu/students/alicejones.html",
            "http://example.com/www.xyz.edu/students/bobsmith.html"
          ]
        }
      ]
    }
