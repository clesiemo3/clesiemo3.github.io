---
layout: inner
title: 'Change Filter Value Across Multiple Tables and Schemes'
date: 2015-06-29 09:00:58
categories: blog
tags: spotfire python ironpython filter tables schemes
lead_text: 'Change Filter Value Across Multiple Tables and Schemes'
---

This script will allow you to set all of your filters of a certain type (Radio Button in this example) to a specific value or Document Property. Combine this with a dropdown list linked to a Document Property and your users can filter with your custom drop down across the document.

```python
from Spotfire.Dxp.Application.Filters import RadioButtonFilter
from Spotfire.Dxp.Data import DataManager

#get the Data Tables collection.
coll = Application.GetService[DataManager]().Tables
#get the number of schemes we need to loop through.
#if you only want certain schemes you can call them
#out instead in Document.FilteringSchemes["stringName"]
numSchemes = Document.FilteringSchemes.Count

for i in range(0,numSchemes):
    for tab in coll:
    #get our RadioButtonFilter; Other filter types work as well
        filt = Document.FilteringSchemes[i].Item[tab].Item[tab.Columns.Item["col1"]].As[RadioButtonFilter]()
        #filt will be NoneType if that column is a different filter type
        if filt is not None:
            filt.Value = Document.Properties["col1DD"]
```
