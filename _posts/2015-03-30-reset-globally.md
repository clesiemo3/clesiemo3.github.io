---
layout: inner
title: 'Reset Markings and Filters in Spotfire'
date: 2015-03-30 16:05:25
categories: blog
tags: spotfire python ironpython reset marking filter
lead_text: 'Resetting your markings and filters globally'
---

I wrote this script to clear out my markings and filters when I am  setting new values or switching views. This avoids the necessity to click in white space to reset your marking if you're using details visualizations driven by your markings. This can also be used as a general purpose reset button for your users in which you could include Document Property changes as well.

Potential improvements could be to bring in bookmarks that have been set and after resetting all the filters and markings you reapply your bookmark as a starting position.

```python
#This is a generic script to reset your filters for 1 scheme and clear your marking for a specific table
#required Spotfire variable: dataTable to clear your marking on.

from Spotfire.Dxp.Data import IndexSet #used for marker clearing
from Spotfire.Dxp.Data import RowSelection #used for marker clearing
from Spotfire.Dxp.Data import DataManager #used for marker clearing

#reset all filters
scheme = Document.FilteringSchemes[0]
scheme.ResetAllFilters()

#clear marking
marking1=Application.GetService[DataManager]().Markings["Marking - 1"]
marking2=Application.GetService[DataManager]().Markings["Marking - 2"]
#set up rowsToMark as an empty row selection by initializing and clearing; There is probably a cleaner way to do this but it works
rowCount = dataTable.RowCount
#sets an IndexSet the size of your table with no values marked (False)
rowsToMark = IndexSet(rowCount,False)
#sets our selection to our empty IndexSet
marking1.SetSelection(RowSelection(rowsToMark),dataTable)
marking2.SetSelection(RowSelection(rowsToMark),dataTable)

###########################
#Same script except there is no need to specify the table or scheme.
#We will loop through all our data tables and filtering schemes.

from Spotfire.Dxp.Data import IndexSet
from Spotfire.Dxp.Data import RowSelection
from Spotfire.Dxp.Data import DataManager

#reset all filters
for i in range(0,Document.FilteringSchemes.Count):
Document.FilteringSchemes[i].ResetAllFilters()

#Get collections
marks = Application.GetService[DataManager]().Markings
tabs = Application.GetService[DataManager]().Tables

for m in marks:
for t in tabs:
#set up rowsToMark as an empty row selection by initializing and clearing; There is probably a cleaner way to do this but it works
rowCount = t.RowCount
#sets an IndexSet the size of your table with no values marked (False)
rowsToMark = IndexSet(rowCount,False)
m.SetSelection(RowSelection(rowsToMark),t)
```

Let me know if you have any questions!