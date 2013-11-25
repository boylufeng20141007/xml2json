xml2json
========

Translate any XML document to corresponding JSON or vice versa

HISTORY
-------

Most of this code was written by Stefan Gössner and published at:
http://goessner.net/download/prj/jsonxml/

You can read a description of what the library originally did at:
http://www.xml.com/pub/a/2006/05/31/converting-between-xml-and-json.html

I did 2 major changes to the original code:

1. XML attributes are translated as siblings, rather than children, to their
   corresponding "parent" element in JSON. This produces more **deterministic**
   translation.
2. Use JSON list structure more frequently. This ensures **lossless** translation
   in more cases than the original and also is capable of handling some "mixed"
   cases that the original would just pass along as untranslated XML.
   
Details of these changes are explained below. For anything else, read the xml.com
article linked above.

USAGE
-----

For example usage, see xmljson_test.html.

The code itself also works and has been used as a node.js module together
with the libxml parser, but you need to change a few lines to properly integrate
as a node.js module (e.g. module.exports...)


DETAILS
-------

An explanation of changes wrt the original library by Stefan Grössner follows:

<e>text</e>
{
  "e":"text"
}

vs

<e name="value">text</e>
{
  "e":{
    "@name":"value",
    "#text":"text"
  }
}

=> the path to "text" changes depending on whether the element has attributes 
   present or not. Very bad if there could be optional attributes, or attributes
   added over time.

For more examples, see http://goessner.net/download/prj/jsonxml/xmljson_test.html

Solution is to lift out attributes to be on the same level as the element itself:

<e name="value">text</e>
{
  e@name:"value",
  e:"text"
}

Note that for empty elements, we still explicitly print null for clarity:

<e name="value" />
{
  e@name:"value",
  e:null
}

The other fix is to use lists to more closely resemble original xml structure
in case of multiple children with same node name.

Original:

<e> <a>text</a> <a>text</a> </e>
{
  "e":{"a":[
      "text",
      "text"
    ]}
}

Me:


<e> <a>text</a> <a>text</a> </e>
{
  e:[
        {
      a:"text"
    },
        {
      a:"text"
    }
  ]
}

This approach allows to handle "mixed content" elements better, where original
gives up:

<a>x<c/>y</a>
{
  "a":"x<c/>y"
}

Me:

<a>x<c/>y</a>
{
  a:[
    "x",
        {
      c:null
    },
    "y"
  ]
}

(Same fix also supports CDATA sections, should there be any.)


Not yet supported in xml->json:

All types are strings. Preferably types would be defined in xml schema, and 
the xml parser would return node values in correct javascript type. If this is
not the case, we can emulate something by converting strings to other types in 
a separate function to be done later (ie xml schema hard coded into a js function)
For database searches, having the correct types will help.


Not yet supported in json->xml:

I have not yet made the corresponding changes to json2xml.js module, so it's not
actually possible to take the output of xml2json.js and translate it back to xml.

Multi-dimensional arrays. What is the correct xml presentation for this?

{ e : [["a", "b"], ["c", "d"]] }





