---
layout: inner
title: 'Logging Usage in Spotfire'
date: 2015-01-29 20:46:04
categories: blog
tags: spotfire python ironpython logging
lead_text: 'Logging Usage in Spotfire with Ironpython'
---

This is a script I wrote to help with tracking the usage of our Spotfire analyses. Originally I was utilizing the System library to get the UserName of the user. However, I found that when the user accesses Spotfire through the Web Player this variable shows up as the web player server. Utilizing the mini-hack of programmatically creating a bookmark you can find out the username as well.  

No variables are required from Spotfire to make this work and you should be able to use this in any Spotfire analysis. The code is commented throughout but please feel free to ask questions or make suggestions.  

```python
########
#imports
import time
from Spotfire.Dxp.Application import DocumentMetadata
from Spotfire.Dxp.Application import BookmarkComponentFlags  

#variable setup
dmd = Application.DocumentMetadata #Get MetaData
now = time.strftime("%c") #Get DateTimeStamp
path = str(dmd.LoadedFromLibraryPath) #Get Path
#Find "/" from the right and grab the position of the character past it
sub = path.rfind("/") + 1
length = len(path) #get length of path
#The specific analysis is the slice of the path after the farthest right '/' character.
analysis = path[sub:length] 

#old username grab; does not work on web player but works well otherwise
#from System import Environment
#username = Environment.UserName

#create temp bookmark to grab username from CreatedBy and then delete temp bookmark.
#The system requires an explicit bookmark name for your new 
#bookmark even though the AddNew function doesn't appear to use it.
bookmarkName = "tempBookmark"
#creates  new bookmark
tmpBookmark = Document.Bookmarks.AddNew("a1",BookmarkComponentFlags.FilterSettings)
#Finds who created the most recent bookmark
username = Document.Bookmarks[Document.Bookmarks.Count-1].CreatedBy
Document.Bookmarks.Remove(tmpBookmark) #cleanup

#write to variable; writes a simple comma delimited string to add as our row to a CSV file
logRow = username + ',' + now + ',' + analysis + ',' + path + '\n' 
#don't forget the \n or you'll be writing on the same line forever.

#write to csv
file = open('\\\\network_share\\sub_folder_1\\username\\Public\\Spotfire Logging\\Log.csv','a') 
#a is for appending instead of overwriting. \ is used to escape \ hence the doubles.
file.write(logRow) #writes our comma separated string as a row.
file.close()

#Optional Setup:
#I typically have a landing page on my Spotfire Analyses that forces the user to hit this button 
#once rather than embedding this script in something used frequently or not every time they open the analysis.
#Since they won't need the landing page after the first click I move them to the true starting page
#(page 0 in this script) and set the navigation mode as Tabs over my default "History" mode.

#change page
Document.ActivePageReference = Document.Pages[0]
#change navigation
Document.Pages.NavigationMode = Document.Pages.NavigationMode.Tabs
```