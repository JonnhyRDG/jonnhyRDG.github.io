---
layout: default
---
# Solaris/Python : Re building an existing set in Maya to USD

Use case: I had a whole city set built in maya, and I wanted to transform it into USD taking advantage of it´s very simple instance and reference systems.

To start, I´ll share the code I used inside Solaris, this plus a dictionary extracted from an xml file, is all that´s needed to re build.
I'll explain each python file by phases.

[JonnhyRDG/ajhoudini](https://github.com/JonnhyRDG/ajhoudini)

Context. I have a set that is composed of a group of assets. So in that regard, I need to generate each part separately. This parts will be called.
Blocks = groups of buildings/houses.
City = group of Blocks (plus trees and other elements, that are also grouped)
So this is what the hierarchy looks like.
City
 |_Block01_A_01
 |_Block01_A_02
          |_building01_A_01
          |_building01_A_02
So you'll see in the python scripts, the first step is to generate the blocks, and then the city.

## Step 1: Exporting the XML file from Maya.
Here's a snippet I got from chatGPT to export xml data with matrices from maya. 
It can be customized to export whatever you want from it, but you have to dig into it a bit.
I do have a custom code/api a TD did for me, but this is the gist of it.
```python
import xml.etree.ElementTree as ET
import maya.cmds as cmds

# Create the root element
root = ET.Element("Scene")

# Get a list of all objects in the scene
objects = cmds.ls(type="transform")

# Loop through each object and extract its matrix information
for obj in objects:
    matrix = cmds.xform(obj, q=True, matrix=True, ws=True)
    
    obj_element = ET.SubElement(root, "Object")
    obj_element.set("name", obj)
    
    matrix_element = ET.SubElement(obj_element, "Matrix")
    matrix_text = " ".join(str(value) for value in matrix)
    matrix_element.text = matrix_text

# Create an ElementTree and write to a file
tree = ET.ElementTree(root)
output_path = "output.xml"
tree.write(output_path)

print("XML exported successfully.")
```
## **Step 2: Extracting and converting the XML data into an easily readable dictionary**
```python
import xml.etree.ElementTree as ET
import json
```
We'll be using both **xml.etree** and **json** modules for this little snippet.

```python
xmlblock = ET.parse("P:/AndreJukebox/assets/sets/city/publish/xml/block_builder.xml")
xmlcity = ET.parse("P:/AndreJukebox/assets/sets/city/publish/xml/city_builder.xml")
```
Both *xmlblock* and *xmlcity* are variables that wrap the xml objects.

And below we'll create the empty dictionaries that we will fill looping through our xml objects above and combine to write out a json file.
```python
blocksdict = {}
assetdict = {}
```
Now we're going to create a function and pass a variable for each XML object we have.
```python
def assetListFromXML(xml):
```
This will allow us to have a couple of *if* statements, so according to which xml object we pass, it will act differently.





Text can be **bold**, _italic_, or ~~strikethrough~~.

[Link to another page](./another-page.html).

There should be whitespace between paragraphs.

There should be whitespace between paragraphs. We recommend including a README, or a file with information about your project.

# Header 1

This is a normal paragraph following a header. GitHub is a code hosting platform for version control and collaboration. It lets you and others work together on projects from anywhere.

## Header 2

> This is a blockquote following a header.
>
> When something is important enough, you do it even if the odds are not in your favor.

### Header 3

```js
// Javascript code with syntax highlighting.
var fun = function lang(l) {
  dateformat.i18n = require('./lang/' + l)
  return true;
}
```

```ruby
# Ruby code with syntax highlighting
GitHubPages::Dependencies.gems.each do |gem, version|
  s.add_dependency(gem, "= #{version}")
end
```

#### Header 4

*   This is an unordered list following a header.
*   This is an unordered list following a header.
*   This is an unordered list following a header.

##### Header 5

1.  This is an ordered list following a header.
2.  This is an ordered list following a header.
3.  This is an ordered list following a header.

###### Header 6

| head1        | head two          | three |
|:-------------|:------------------|:------|
| ok           | good swedish fish | nice  |
| out of stock | good and plenty   | nice  |
| ok           | good `oreos`      | hmm   |
| ok           | good `zoute` drop | yumm  |

### There's a horizontal rule below this.

* * *

### Here is an unordered list:

*   Item foo
*   Item bar
*   Item baz
*   Item zip

### And an ordered list:

1.  Item one
1.  Item two
1.  Item three
1.  Item four

### And a nested list:

- level 1 item
  - level 2 item
  - level 2 item
    - level 3 item
    - level 3 item
- level 1 item
  - level 2 item
  - level 2 item
  - level 2 item
- level 1 item
  - level 2 item
  - level 2 item
- level 1 item

### Small image

![Octocat](https://github.githubassets.com/images/icons/emoji/octocat.png)

### Large image

![Branching](https://guides.github.com/activities/hello-world/branching.png)


### Definition lists can be used with HTML syntax.

<dl>
<dt>Name</dt>
<dd>Godzilla</dd>
<dt>Born</dt>
<dd>1952</dd>
<dt>Birthplace</dt>
<dd>Japan</dd>
<dt>Color</dt>
<dd>Green</dd>
</dl>

```
Long, single-line code blocks should not wrap. They should horizontally scroll if they are too long. This line should be long enough to demonstrate this.
```

```
The final element.
```
