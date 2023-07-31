---
layout: page
title: GenerateTree
permalink: /GenerateTree
---

# Visualise Newick Trees

## Setup

```python
from ete3 import Tree
from ete3.treeview.main import NodeStyle
import ipywidgets as widgets
from IPython.display import Image, display, Markdown
import os
os.environ['QT_QPA_PLATFORM']='offscreen'
```

```python
t = None
tree_name = None
selection = None
colour_map = {}
```

```python
def select_click(b):
    output2.clear_output(wait=True)
    colour_map.update({colour.value : selection.value}) 
    for select in selection.value:
        with output2:
            printmd(f'Added **<font color={colour.value}>{select}</font>**')
```

```python
def printmd(string):
    display(Markdown(string))
```

```python
colour = widgets.ColorPicker(
                             concise=False,
                             description='Pick a colour:',
                             value='blue',
                             disabled=False
                            )

select_btn = widgets.Button(description="Submit")
select_btn.on_click(select_click)
                            
output = widgets.Output()
output2 = widgets.Output()
```

```python
newick = widgets.FileUpload(
                            accept='.newick',
                            multiple=False
                            )
newick_btn = widgets.Button(description="Submit")

def newick_submit(b):
    global t
    global selection
    with output:
        print(newick.value[0]["name"], " loading...")
    uploaded_file = newick.value[0]
    with open("input.newick", "wb") as fp:
        fp.write(uploaded_file.content)
    t = Tree("input.newick")
    nodes = t.get_leaves()
    names = [node.name for node in nodes]
    selection = widgets.SelectMultiple(
                                       options=names,
                                       layout={'width': 'max-content'},
                                       description='Pick the nodes to colour:',
                                       disabled=False,
                                      )
    with output:
        display(colour)
        display(selection, select_btn)
    
newick_btn.on_click(newick_submit)
```

```python
name_tree = widgets.Text(
                         value='New_Tree',
                         placeholder='Start typing',
                         description='Filename (without .png):',
                         disabled=False   
                        )
name_btn = widgets.Button(description="Submit")

def name_submit(b):
    tree_name = name_tree.value()

name_btn.on_click(name_submit)
```

```python
def NewNodeStyle(colour: str) -> NodeStyle:
    ns = NodeStyle()
    ns['shape'] = "sphere"
    ns['size'] = 10
    ns['fgcolor'] = colour
    return ns
```

```python
final_output = widgets.Output()
```

```python
def colour_nodes():
    for colour, names in colour_map.items():
        nodes = []
        for name in names:
            nodes.extend(t.search_nodes(name=name))
        for node in nodes:
            node.set_style(NewNodeStyle(colour))
```

```python
def generate(b):
    def quick_render(img_file):
        t.render(img_file, w=250, h=700)
    with final_output:
        print("Generating, this takes a moment...")
    colour_nodes()
    img_file = f"{tree_name}.svg"
    quick_render(img_file)
    img_file = f"{tree_name}.png"
    quick_render(img_file)
    with final_output:
        display(Image(img_file))

generate_btn = widgets.Button(description="Generate tree")
generate_btn.on_click(generate)
```

## Make your tree

Upload your newick file:

```python
display(newick, newick_btn)
```


    FileUpload(value=(), accept='.newick', description='Upload')



    Button(description='Submit', style=ButtonStyle())


Ctrl/Cmd click to select multiple, click once with multiple selected to revert to one:

```python
display(output)
display(output2)
```


    Output()



    Output()


Name your file:

```python
display(name_tree)
```


    Text(value='New_Tree', description='Filename (without .png):', placeholder='Start typing')


Click generate once you're finished:

```python
display(generate_btn)
display(final_output)
```


    Button(description='Generate tree', style=ButtonStyle())



    Output()

