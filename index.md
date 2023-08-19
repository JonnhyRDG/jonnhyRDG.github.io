---
layout: default
---

```python
### THIS CODE HAS TO BE EXECUTED FROM A PYTHON SCRIPT NODE IN SOLARIS

from pxr import Usd, UsdGeom, Gf
import hou
import json

node = hou.pwd()


class blockbuild():
    def __init__(self):
        self.dictread()
    
    def dictread(self):
        self.citydict = open("P:/AndreJukebox/assets/sets/city/publish/xml/block_builder.json")
        self.cityread = json.load(self.citydict)
    
    def blockslist(self):
        for keys in self.cityread:
            blockhierarchy = keys.rsplit("_",1)[0]
            block_stage = f'P:/AndreJukebox/assets/sets/{blockhierarchy}/publish/usd/{blockhierarchy}.usd'
            self.stage = Usd.Stage.CreateNew(block_stage)
            group_prim_path = f'/{keys}'
            group_prim = self.stage.DefinePrim(group_prim_path,'Xform')
            self.createRefs(blocks=keys,refsnum=len(self.cityread[keys]['assets']))
            group_model = Usd.ModelAPI(group_prim)
            group_model.SetKind("group")
            self.stage.GetRootLayer().Save()
            # print('________DONE_________')
    
    def createRefs(self, blocks, refsnum):    
        iteration = 0
        for buildings in self.cityread[blocks]['assets']:
            xform = (self.cityread[blocks]['assets'][buildings]['xform'])
            assetname = buildings.rsplit("_",1)[0]
            new_building = f"/{blocks}/{buildings}"
            
            print(new_building)

            new_prim = self.stage.DefinePrim(new_building)
            assetabc = (self.cityread[blocks]['assets'][buildings]['abcpath'])
            new_prim.GetReferences().AddReference(assetabc)
            new_prim.SetInstanceable(True)
            building_model = Usd.ModelAPI(new_prim)
            building_model.SetKind("component")
            
            # assign the matrix to each building
            target_path = self.stage.GetPrimAtPath(new_building)
            xformable = UsdGeom.Xformable(target_path)
            transform_matrix = eval(xform)
            final_matrix = Gf.Matrix4d(transform_matrix)
            xformable.ClearXformOpOrder()
            xformable.AddTransformOp().Set(value=final_matrix)


blockbuild().blockslist()

```



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
