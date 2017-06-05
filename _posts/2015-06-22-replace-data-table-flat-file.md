---
layout: inner
title: 'Replace Data Table with Flat File Source'
date: 2015-06-22 09:00:48
categories: blog
tags: spotfire python ironpython datatable flatfile
lead_text: 'Replace Data Table with Flat File Source'
---

This script enables you to swap out your data table for a file that you have either on your local machine or a network share. This can be useful if you have multiple files for different users or code environments that you want to switch between on demand. Provided your column names and types are the same there should not be any noticeable change to your visualizations other than the data itself. This can be done similarly with an information link data source (InformationLinkDataSource(Guid)) or other such data sources created through the Spotfire.Dxp.Data.Import Namespace.

```python
#Replace tables from file
from Spotfire.Dxp.Data import *

#setup data source from selected file
myDataManager = Document.Data
ds=myDataManager.CreateFileDataSource('C:\Users\CLESIEMO3\Documents\Business\Finances\mySalary.xlsx')
#network shares work too!
ds=myDataManager.CreateFileDataSource('\\Network\CLESIEMO3\Documents\Business\Finances\mySalary.xlsx')

#replace a data table
table1 = Document.Data.Tables["T1"]
table1.ReplaceData(ds)
```
`