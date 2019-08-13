# Examples

## Load an XML File

Basic XML file loading.

The basic syntax to load an XML file from
disk and check for an error. (ErrorID()
will return 0 for no error.)

```cpp
int example_1()
{
    XMLDocument doc;
    doc.LoadFile( "resources/dream.xml" );
    return doc.ErrorID();
}
```

## Parse an XML from char buffer

Basic XML string parsing.

The basic syntax to parse an XML for
a char* and check for an error. (ErrorID()
will return 0 for no error.)

```cpp
int example_2()
{
    static const char* xml = "<element/>";
    XMLDocument doc;
    doc.Parse( xml );
    return doc.ErrorID();
}
```

## Get information out of XML

In this example, we navigate a simple XML
file, and read some interesting text. Note
that this example doesn't use error
checking; working code should check for null
pointers when walking an XML tree, or use
XMLHandle.

(The XML is an excerpt from "dream.xml").

```xml
<?xml version=\"1.0\"?>
<!DOCTYPE PLAY SYSTEM \"play.dtd\">
<PLAY>
  <TITLE>A Midsummer Night's Dream</TITLE>
</PLAY>;
```

The structure of the XML file is:

<ul>
	<li>(declaration)</li>
	<li>(dtd stuff)</li>
	<li>Element "PLAY"</li>
	<ul>
		<li>Element "TITLE"</li>
		<ul>
		    <li>Text "A Midsummer Night's Dream"</li>
		</ul>
	</ul>
</ul>

For this example, we want to print out the
title of the play. The text of the title (what
we want) is child of the "TITLE" element which
is a child of the "PLAY" element.

We want to skip the declaration and dtd, so the
method FirstChildElement() is a good choice. The
FirstChildElement() of the Document is the "PLAY"
Element, the FirstChildElement() of the "PLAY" Element
is the "TITLE" Element.

```cpp
XMLDocument doc;
doc.Parse( xml );
XMLElement* titleElement = doc.FirstChildElement( "PLAY" )->FirstChildElement( "TITLE" );
```

We can then use the convenience function GetText()
to get the title of the play.

```cpp
const char* title = titleElement->GetText();
printf( "Name of play (1): %s\n", title );
```

Text is just another Node in the XML DOM. And in
fact you should be a little cautious with it, as
text nodes can contain elements.

```html
Consider: A Midsummer Night's <b>Dream</b>
```

It is more correct to actually query the Text Node
if in doubt:

```cpp
XMLText* textNode = titleElement->FirstChild()->ToText();
title = textNode->Value();
printf( "Name of play (2): %s\n", title );
```

Noting that here we use FirstChild() since we are
looking for XMLText, not an element, and ToText()
is a cast from a Node to a XMLText.

## Read attributes and text information.

There are fundamentally 2 ways of writing a key-value
pair into an XML file. (Something that's always annoyed
me about XML.) Either by using attributes, or by writing
the key name into an element and the value into
the text node wrapped by the element. Both approaches
are illustrated in this example, which shows two ways
to encode the value "2" into the key "v":

```cpp
bool example_4()
{
    static const char* xml =
        "<information>"
        "   <attributeApproach v='2' />"
        "   <textApproach>"
        "       <v>2</v>"
        "   </textApproach>"
        "</information>";

```

TinyXML-2 has accessors for both approaches.

When using an attribute, you navigate to the XMLElement
with that attribute and use the QueryIntAttribute()
group of methods. (Also QueryFloatAttribute(), etc.)

```cpp
XMLElement* attributeApproachElement = doc.FirstChildElement()->FirstChildElement( "attributeApproach" );
attributeApproachElement->QueryIntAttribute( "v", &v0 );
```

When using the text approach, you need to navigate
down one more step to the XMLElement that contains
the text. Note the extra FirstChildElement( "v" )
in the code below. The value of the text can then
be safely queried with the QueryIntText() group
of methods. (Also QueryFloatText(), etc.)

```cpp
XMLElement* textApproachElement = doc.FirstChildElement()->FirstChildElement( "textApproach" );
textApproachElement->FirstChildElement( "v" )->QueryIntText( &v1 );
```
