---
layout: ajhoudini
---
# **Solaris/Python : Re building an existing set in Maya to USD**

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

## **Step 1: Exporting the XML file from Maya.**
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

