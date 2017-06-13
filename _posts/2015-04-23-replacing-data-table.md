---
layout: inner
title: 'Replacing the Data Table of your Visualization'
date: 2015-04-23 14:55:56
categories: blog
tags: spotfire python ironpython datatable replace visualization
lead_text: 'Replacing the Data Table of your Visualization'
---

This script is designed to swap between two different data tables within the same visualization. However, it can be utilized to replace the data table with a new one you've created/pulled from somewhere or just simply reorder and redesign your table with the column, sort, and column width options.

{% highlight python %}
######################################################################
##Table Swapping Script
##This script swaps data tables and rebuilds your preferences.
##
##Required Spotfire variables:
##viz is the Visualization you are swapping the table for.
##t1 and t2 are the 2 swappable data tables.
##
######################################################################

from Spotfire.Dxp.Application.Visuals import VisualContent
from Spotfire.Dxp.Application.Visuals import Legend
from Spotfire.Dxp.Application.Visuals import TablePlotColumnSortMode

myViz = viz.As[VisualContent]()

#toggle: if 1 then 2; if 2 then 1.
if myViz.Data.DataTableReference == t1:
omyViz.Data.DataTableReference = t2
omyDataTable = t2
elif myViz.Data.DataTableReference == t2:
omyViz.Data.DataTableReference = t1
omyDataTable = t1

#This makes sure your current preferences are applied. 
#Unncessary if you Auto Configure but then you lose things like coloration rules.
myViz.ApplyUserPreferences() 

#Add your columns in order. You can loop through if you want them all.
myViz.TableColumns.Add(myDataTable.Columns['column1'])
if myDataTable == t1:
omyViz.TableColumns.Add(myDataTable.Columns['column2'])
myViz.TableColumns.Add(myDataTable.Columns['column3'])
myViz.TableColumns.Add(myDataTable.Columns['column4'])
myViz.TableColumns.Add(myDataTable.Columns['column5'])

#loop version; add IF statements to exclude items.
#colEnum = myDataTable.Columns.GetEnumerator()
#for col in colEnum:
#myViz.TableColumns.Add(myDataTable.Columns[col])

#sorting
myViz.SortInfos.Add(myDataTable.Columns['column1'], TablePlotColumnSortMode.Ascending)
if myDataTable == t1:
omyViz.SortInfos.Add(myDataTable.Columns['column2'], TablePlotColumnSortMode.Ascending)
myViz.SortInfos.Add(myDataTable.Columns['column3'], TablePlotColumnSortMode.Ascending)

#column width
colEnum = myDataTable.Columns.GetEnumerator()

for col in colEnum:
otabCol = myViz.TableColumns.TryGetTableColumn(myDataTable.Columns[col.Name])[1]
oif tabCol is not NoneType:
oif col.Name == 'column1':
otabCol.Width = 225
oelif col.Name == 'column2' or col.Name == 'column3':
otabCol.Width = 100
oelse:
otabCol.Width = 75

#turn off the legend. I don't like it in a lot of cases personally
myViz.Legend.Visible = False
{% endhighlight %}
