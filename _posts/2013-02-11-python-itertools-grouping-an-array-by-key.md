---
layout: post
title: Python itertools - Grouping an array by key
date: 2013-02-11 23:13:00
author: Adam Presley
status: Published
tags: python itertools
slug: python-itertools-grouping-an-array-by-key
---

Have you ever had a flat dataset and you needed a portion of the data
grouped into a sub-array? I came across such a need this weekend and
thought I'd share my experience. In this code sample we'll see how to
take an array of dictionaries (similar to what might come out of a
database), sort it, then perform a grouping. This data is a list of
people and their phone numbers. The trick here is that each person can
have one or more phone numbers of different types. For example Jessica
Alba has a home, work and cell phone number. What I want to end up with
a a single array entry for each person, with a sub-array of all their
phone numbers. First let's start with the code.  
  
    :::python
    from itertools import groupby

    people = [
      {"firstName": "Adam", "lastName": "Presley", "phoneType": "Home Phone", "phoneNumber": "555-7844"},
      {"firstName": "Jessica", "lastName": "Alba", "phoneType": "Home Phone", "phoneNumber": "555-7833"},
      {"firstName": "Adam", "lastName": "Presley", "phoneType": "Work Phone", "phoneNumber": "555-1122"},
      {"firstName": "Bob", "lastName": "Hope", "phoneType": "Home Phone", "phoneNumber": "555-9987"},
      {"firstName": "Jessica", "lastName": "Alba", "phoneType": "Cell Phone", "phoneNumber": "555-0915"},
      {"firstName": "Jessica", "lastName": "Alba", "phoneType": "Work Phone", "phoneNumber": "555-4821"}
    ]

    people = sorted(people, key=lambda k: k["firstName"])
    groupedResult = []

    for key, person in groupby(people, lambda k: k["firstName"]):
      row = next(person)

      newRow = dict([(k, v) for k, v in row.items() if not k in ("phoneType", "phoneNumber")])
      newRow["phoneNumbers"] = [{"phoneType": row["phoneType"], "phoneNumber": row["phoneNumber"]}] + [{"phoneType": ph["phoneType"], "phoneNumber": ph["phoneNumber"]} for ph in person]

      groupedResult.append(newRow)

    for person in groupedResult:
      print "%s %s:" % (person["firstName"], person["lastName"])

      for phoneNumber in person["phoneNumbers"]:
         print "\t%s: %s" % (phoneNumber["phoneType"], phoneNumber["phoneNumber"])

From the beginning you will see the array of dictionaries as I described above. There are multiple records for each person who has more than one phone number. The first thing I want to do with this array is to sort it, since our group method will need them sorted. This is accomplished using the **sorted()** method. The first argument is the array to sort, and the *key* argument is a function, or a lambda expression in this case that specifies what key to use in the sorting. In our case we are
going to sort by first name.  
  
Now it is time to loop over our collection. Notice the use of **groupby()**. This is a nifty method from the *itertools* module that
returns a key and a grouper object. The grouper object is iterable and basically contains the current group of items as grouped by the current key. The next line retrieves our first item from the current group. This I use to seed the new row we are creating.  
  
To create the row I am doing a bit of list comprehension. Using the **dict()** method to create a dictionary from an array of lists I can
create the resulting row, which should result in a dictionary (very similar to the rows in the people array). In this list comprehension
though I am getting all the items from our current row *except* for the *phoneType* and *phoneNumber* keys. I don't want them until the next line.  
  
From here I then create a new key called *phoneNumbers* (notice the plural) that will house a sub-array of all phone numbers for the current person. We can break this line up into two parts, or list comprehension actually. The first creates an array of a single dictionary consisting of the first row we've already retrieved in the variable named *row*.  
  
    :::python
    [{"phoneType": row["phoneType"], "phoneNumber": row["phoneNumber"]}]

The next list comprehension assembles an array of the remaining rows
from the **person** grouper iterable object.  
  
    :::python
    [{"phoneType": ph["phoneType"], "phoneNumber": ph["phoneNumber"]} for ph in person]

Then you'll notice that we are using an addition operation to combine the two arrays into one. Finally the new row is added to the
**groupedResult** array and we do a quick loop to show off our results.  
  
I found the itertools to be not only useful, but very powerful. I have a lot more to learn about what they can do. Cheers, and happy coding!

