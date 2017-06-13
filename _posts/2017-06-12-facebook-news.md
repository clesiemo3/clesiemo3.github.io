---
layout: inner
title: 'Facebook News'
date: 2017-06-12 21:37:00
categories: blog
tags: facebook python python3 rss api oauth
lead_text: 'Using Python with the Facebook API to post news'
published: true
---

My church has a community for young adults which I am a part of. I was put in charge of announcements which includes emailing out upcoming events & schedule (who is teaching, in-class announcements, leading prayer, etc.) to the leadership team weekly. Emailing some info weekly isn't too hard right? Well, what about the rest of the group? Wouldn't it be nice to share with everyone? 

Everyone is in our facebook group for news, events, discussion, etc. Knowing there is a Facebook API I figured I could just write some python on a cron job to post weekly announcements.

Here's how it turned out:

code: [clesiemo3/young-adult-news](https://github.com/clesiemo3/young-adults-news)

First things first, tons of credentials to manage. 	

I need the following for this to work:

* Facebook App credentials (`client_id` and `client_secret`)
  * Create an app [link](https://developers.facebook.com/docs/apps/register)
* OAuth Google Credentials 
  * [sheets.py](https://github.com/clesiemo3/young-adults-news/blob/master/sheets.py) almost copy-paste from google sample
  * [google how-to](https://developers.google.com/sheets/api/quickstart/python)
* My Facebook Account OAuth Token (who the bot posts as)
  * Get your first token manually via a link like this in your browser: https://www.facebook.com/v2.8/dialog/oauth?client_id=123456789&redirect_uri=http://localhost&response_type=token&scope=user_managed_groups,publish_actions,user_events
  * [auth_flow.py](https://github.com/clesiemo3/young-adults-news/blob/master/auth_flow.py) exchanges your token for a new one as a part of the main script to ensure it does not expire. 

Credential Files:

* `config.py` handles the facebook app & our sheets id
* `user_token.json` for my facebook account
* `~/.credentials/sheets.googleapis.com-yalt.json` for oauth google creds. 

A tad messy but it works.

Once we're set up with all of that we're good to go!

The news post itself is a basic text string with `\n` for newlines so we just concatenate (`+=`) over and over to build our message string.

There are a couple data sources we want to pull from to include in our news post. 

* Church Events Feed - http://www.efree.org/events/feed/ a simple XML structured RSS feed. The feedparser library makes this easy.
* Facebook Group Events
* Google Sheets Schedule - a simple sheet that looks something like this:

Coming Sunday | Topic                   | Announcements| Prayer   | Teacher
------------- |:-----------------------:|:------------:|:--------:|:-----------:
 6/18/2017    | Teaching - John 12:1-19 | clesiemo3    | john.doe |  bill.jones

First we start with a simple title and notice that this is a bot and people should let me know when it tries to go Terminator on us all:

`message = "Weekly Announcements\nPosted by Bot - Report issues in comments\n"`

Next we get the upcoming week's schedule with the following function: `message += sheets.yalt_schedule()`. Using a bunch of try+except messages we attempt to glean as much info as we can from the schedule. Some weeks are blank due to no class for example. [code](https://github.com/clesiemo3/young-adults-news/blob/master/sheets.py#L74-L95)

Using the Facebook graph api we update our token first: `graph = facebook.GraphAPI(auth_flow.update_token())`. Then we get all events for our group that start in the next two weeks to include in our announcement. 

```
graph = facebook.GraphAPI(auth_flow.update_token())
groups = graph.get_object("me/groups")
group_id = [x for x in groups['data'] if x['name'] == "Young Adult's Ministry at First Free Church, Manchester, MO"][0]['id']
now = dt.now()
two_weeks = dttm.timedelta(days=14)
until = now + two_weeks
events = graph.get_object("195603127146892/events", 
                          since = now.strftime('%Y-%m-%d'), 
                          until = until.strftime('%Y-%m-%d'))
```

A couple datetime masks are set up front for parsing and outputting data values in a consistent format.

```
fb_mask = '%Y-%m-%dT%H:%M:%S%z'
efree_mask = '%a, %d %b %Y %H:%M:%S %z'
output_mask = '%A %B %d %I:%M %p'
```

If we have events show up in the date range, then we try to loop through and add them to the message:

```
url_stub = "https://www.facebook.com/events/"
if events['data'] != []:
    message += "\nYoung Adults Events\n"
    try:
        for event in events['data']:
            message += "\n" + event['name']
            message += "\n" + url_stub + event['id'] + "/"
            message += "\n" + dt.strptime(event['start_time'], fb_mask).strftime(output_mask) + \
                                         " to " + dt.strptime(event['end_time'], fb_mask).strftime('%H:%M %p')
            message += "\nLocation: " + event['place']['name']
            message += "\n\n" + event['description']
    except Exception as e:
        print(e)
else:
    message += "\nNo Young Adult Facebook Events in the next 2 weeks\n"
```

Next we loop through the RSS feed to look for events coming up. Since this is an all church list, I exclude a few junior/senior high phrases as we don't have any kids that age. For items that have a url link to more info, i just swap the long url out with `...[truncated]` as the link is already included in the message.

```
message += "\n\nEfree Events\n"
exclude = re.compile("(Junior High Spring Retreat|Senior High|KampOut)")
for x in feed.entries:
    msg_x = "\n" + x.title
    msg_x += "\n" + x.link
    msg_x += "\n" + dt.strptime(x.published, efree_mask).strftime(output_mask)
    summary = re.sub(r' <a href.+more.+$', '... [truncated]', x.summary)
    msg_x += "\n\n" + summary + "\n"
    if exclude.search(msg_x):
        print("Excluded!")
        print(msg_x)
        continue
    else:
        message += msg_x
```

Lastly, unescape any html from RSS/other sources and we're ready to post! `print(message)` can be used for viewing your post before sending.

```
message = html.unescape(message)

graph.put_wall_post(message = message, profile_id = group_id)
#print(message)
```

In order to make this automated I used a crontab setup on my raspberry pi. While I developed this on my macbook pro with a conda environment, I had a lot of trouble getting conda to work on the pi so I just resorted to trusty pip & virtualenvs to get the job done. That's while you'll see requirements.txt and pip-requirements.txt in the repo.

I ran this in cron: `0 19 * * 3 ~/scripts/ya.news.sh` which in turn runs this:

```
#!/usr/bin/env bash
cd /home/pi/Code/young-adults-news
/home/pi/virtualenvs/py3/bin/python main.py
```

I used the `cd dir` to avoid being smart about referring to files in the repo. 

You can see some sample output in the readme here: [https://github.com/clesiemo3/young-adults-news](https://github.com/clesiemo3/young-adults-news)

Now I get to sit back while my script does all the work and faithfully posts for me at 7 pm on Wednesday. Totally worth it.

Thanks for reading!
