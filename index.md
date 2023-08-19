---
layout: ajhoudini
---
# **Rebuilding in Solaris/USD an existing set assembled in Maya (nested references), with python**

Use case: I had a whole city set built in maya, and I wanted to transform it into USD taking advantage of it´s very simple and efficient instance and reference systems.
* * *
Also, I didn't want to manually place a single building, because they are too many and I'm lazy. We're talking about 200 buildings, and I don't even know how many other set dressing elements.

To start, I´ll share the code I used inside Solaris, this plus a dictionary extracted from an xml file, is all that´s needed to re build.
I'll explain each python file by phases.

[JonnhyRDG/ajhoudini](https://github.com/JonnhyRDG/ajhoudini)

Context. I have a set that is composed of a group of assets. So in that regard, I need to generate each part separately. This parts will be called.

Blocks = groups of buildings/houses.
City = group of Blocks (plus trees and other elements, that are also grouped)
So this is what the hierarchy looks like.

```python
City
 |_Block01_A_01
 |_Block01_A_02
          |_building01_A_01
          |_building01_A_02
```

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

And now, we're going to access the root of the xml file, and keep it in a variable. For a better overall understanding on how python process, reads and writes xml files I'd recommend watching this:
[Parse XML Files with Python - Basics in 10 Minutes by "max on tech"](https://www.youtube.com/watch?v=5SlemSWGD1g&t=1s&ab_channel=maxontech)
Subscribe to the guy by the way, he's cool.

```python
    root = xml.getroot()
```
So to understand some of the loops below I'll give a sample of the content of my xml file, or maybe just straight up upload the file later. At least it will be useful to build the dicionary.

Here's an asset written in the xml.
```xml
<scenegraphXML version="0.1.0">
      <instanceList>
            <instance name="root_loc" type="group">
            <xform value="1.0 0.0 0.0 0.0 0.0 1.0 0.0 0.0 0.0 0.0 1.0 0.0 0.0 0.0 0.0 1.0" />
      <instanceList>
```
For each level of hierarchy you can see above, I have to dig into it with a for loop. There will be plenty in different levels.

So here we dive:

```python
    for instanceList in root:
        for instance in instanceList:
            for childInstance in instance[2]:
                groupIteration = f'{(int(childInstance.attrib["name"].split(":")[0].rsplit("_",1)[1])):04d}'
                if xml == "xmlblock":
                    group = f'{childInstance.attrib["name"].split(":")[0].rsplit("_",1)[0]}_{groupIteration}'
                else:
                    group = f'{childInstance.attrib["name"].split(":")[0].rsplit("_",1)[0]}'
                groupXform = childInstance[1].attrib["value"]
                groupasset = f'{childInstance.attrib["name"].split(":")[0].rsplit("_",1)[0]}'
                groupusd = f'P:/AndreJukebox/assets/sets/{groupasset}/publish/usd/{groupasset}.usd'
```
Observe how everything inside <instance> like "name" can called through the function "attrib" and then call it as if it was a dictionary, with the attribute name in [""]
```python
childInstance.attrib["name"]
```
The xform is in another level inside the instance. Hence the "[1]" and then the attrib name again.
```pytho
childInstance[1].attrib["value"]
```
The xform is one of the most important pieces of info, because this is the data that alter we'll transform in a matrix, to transform, scale and rotate each single object.

And now we arrive to a very very important point. Later on, the usd python api will request a very specific format for the matrix info, which is different from the one you get in an xml.
Here's the difference:

```xml
Xform from XML
1.0 0.0 0.0 0.0 0.0 1.0 0.0 0.0 0.0 0.0 1.0 0.0 0.0 0.0 0.0 1.0
```
```xml
Matrix for USD
((1.0,0.0,0.0,0.0),(0.0,1.0,0.0,0.0),(0.0,0.0,1.0,0.0),(0.0,0.0,0.0,1.0))
```
Yeah I know, it doesn't look like much, but it matters, and a lot.
Thankfully, the format in the Xform in the XML file is consistently separated by " ".
That means that a little loop and a f' statement is all we need.

First we split the line by spaces, with the python's split function.
```python
tr = groupXform.split(" ")
```
That will turn tr in a list of separated values.

Then we'll create an empty dictionary
```python
tdict = {}
```

And an iteration variable, you'll see what for below.
```python
iter = 0
```

Now what we're going to do, is to iterate through each value of the list, and assign an index to it. That index will become a dictionary key.
```python
  for i in tr:
      iter = iter + 1
      tdict[iter] = i
```
This will give us a dictionary that bascially looks like this:
tdict = {1:value1},{2:value2}...etc

And then we just have to place each value in the new structure, calling the keys in order:

```python
  group_matrix = f'( ({tdict[1]},{tdict[2]},{tdict[3]},{tdict[4]}),({tdict[5]},{tdict[6]},{tdict[7]},{tdict[8]}),({tdict[9]},{tdict[10]},{tdict[11]},{tdict[12]}),({tdict[13]},{tdict[14]},{tdict[15]},{tdict[16]}) )'
```
And that's it, we're now storing all matrices in the correct format. This will be VERY important later.