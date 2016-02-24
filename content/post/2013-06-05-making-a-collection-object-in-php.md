---
author: "Adam Presley"
date: 2013-06-05T07:43:00Z
tags: ["oop", "php", "patterns"]
title: "Making a Collection Object in PHP"
slug: "making-a-collection-object-in-php"
---

Recently I was working on a bit of code where I had an array of
structures (a map or dictionary) that was the result of a query to a
database. This data represented a set of statuses, each record with an
ID and a status. Imagine it looking something like this.

{{< highlight javascript >}}
[
	{ "id": 1, "status": "Active" },
	{ "id": 2, "status": "Inactive" },
	{ "id": 3, "status": "Terminated" }
]
{{< / highlight >}}

<!-- excerpt -->

I had been asked to create some UI elements that would filter a page
based on a couple of specific statuses. Knowing in the past how these
statuses had changed and how the list had grown over time (the list
above being merely a sample) I did not want to hard code the IDs into
button links. Instead I preferred to have a way to refer to the status
by name and get the ID back.

This problem is simple enough to address. Simply do an
**array_search()** or **array_filter()** and call it a day. In this
case though there is a service class that has a method to return an
array of those statuses. My first thought was to make a new method in
this service class that would take this array of statuses and a status
name and return the ID. However it occurred to me that I could create a
new class to represent a *collection* of status structures. Why would I
do that? This actually affords me a couple of abilities.

1.  The class could be created so I could iterate over it like a regular
    array
*   I could also make it so I could reference it like an array by index
*   Finally I could make a method to get the ID of a status by its name

Basically I could create a class that I could reference by index,
iterate over like an array in a **foreach** loop, and attach methods to
for working with a collection of statuses. At this point I had decided
this was the course of action I wanted to take. To do this there are two
interfaces in PHP you need to implement: **ArrayAccess** and
**Iterator**.

The first interface to implement is **ArrayAccess**. This allows you to
reference an instance of your class as an array. This means you can do
something like this.

{{< highlight php >}}
<?php
	$statusCollection = new StatusCollection();
	$statusCollection[] = array("id" => 1, "status" => "Active");
	$statusCollection[] = array("id" => 2, "status" => "Inactive");

	$id = $statusCollection[1]["id"];
?>
{{< / highlight >}}

Notice how we can add to the object like an array, and we can reference
by index just like an array. So how do we make a class to do this?
Implement the **ArrayAccess** interface. Let's see how we would do that
in our **StatusCollection** class.

{{< highlight php >}}
<?php

class StatusCollection implements ArrayAccess {
   private $statuses = array();

   public function __construct($statuses) {
      $this->statuses = $statuses;
   }

   public function offsetExists($offset) {
      return isset($this->statuses[$offset]);
   }

   public function offsetGet($offset) {
      return isset($this->statuses[$offset]) ? $this->statuses[$offset] : null;
   }

   public function offsetSet($offset, $value) {
      if (is_null($offset)) {
         $this->statuses[] = $value;
      } else {
         $this->statuses[$offset] = $value;
      }
   }

   public function offsetUnset($offset) {
      unset($this->statuses[$offset]);
   }
}

?>
{{< / highlight >}}

First and foremost refer to [http://us2.php.net/manual/en/class.arrayaccess.php] as a reference to
the **ArrayAccess** interface. It dictates that your class needs to
implement four methods: *offstExists()*, *offsetGet()*, *offsetSet()*,
and *offsetUnset*. These methods are called when you perform array-like
operations on your class instance variable.

Now we'd like to make it so we can easily iterate over our class
instance variable much like any other array. Basically we want to be
able to do this.

{{< highlight php >}}
<?php

foreach ($statusCollection as $status) {
   printf("<option value=\"%d\">%s</option>\n", $status["id"], $status["status"]);
}

?>
{{< / highlight >}}

To do this we must implement the [**Iterator** interface](http://php.net/manual/en/class.iterator.php). This
interface requires we implement five more methods: *current()*, *key()*,
*next()*, *rewind()*, and *valid()*. When you write code like above to
iterate over an object that implements **Iterator** these methods are
called. The idea here is that we'll keep an internal variable to track
the current iterator position, so when a loop asks for another item we
know which index to return. Here's what that looks like.

{{< highlight php >}}
<?php

class StatusCollection implements ArrayAccess, Iterator {
   private $statuses = array();
   private $position = 0;

   public function __construct($statuses) {
      $this->statuses = $statuses;
      $this->position = 0;
   }

   public function current() {
      return $this->statuses[$this->position];
   }

   public function key() {
      return $this->position;
   }

   public function next() {
      ++$this->position;
   }

   public function offsetExists($offset) {
      return isset($this->statuses[$offset]);
   }

   public function offsetGet($offset) {
      return isset($this->statuses[$offset]) ? $this->statuses[$offset] : null;
   }

   public function offsetSet($offset, $value) {
      if (is_null($offset)) {
         $this->statuses[] = $value;
      } else {
         $this->statuses[$offset] = $value;
      }
   }

   public function offsetUnset($offset) {
      unset($this->statuses[$offset]);
   }

   public function rewind() {
      $this->position = 0;
   }

   public function valid() {
      return isset($this->statuses[$this->position]);
   }
}

?>
{{< / highlight >}}

Finally in my 3rd bullet point I said that I wanted a method to get the
ID of a particular status by name. Here I will simply add a new method
called *getIdByName()* which will do a quick loop to find our status. If
the ID can't be located an exception is thrown.

{{< highlight php >}}
<?php

class StatusCollection implements ArrayAccess, Iterator {
   private $statuses = array();
   private $position = 0;

   public function __construct($statuses) {
      $this->statuses = $statuses;
      $this->position = 0;
   }

   public function current() {
      return $this->statuses[$this->position];
   }

   public function getIdByName($statusName) {
      for ($i = 0; $i < count($this->statuses); $i++) {
         if ($this->statuses[$i]["status"] == $statusName) return $this->statuses[$i]["id"];
      }

      throw new Exception("Status '{$statusName}' not found");
   }

   public function key() {
      return $this->position;
   }

   public function next() {
      ++$this->position;
   }

   public function offsetExists($offset) {
      return isset($this->statuses[$offset]);
   }

   public function offsetGet($offset) {
      return isset($this->statuses[$offset]) ? $this->statuses[$offset] : null;
   }

   public function offsetSet($offset, $value) {
      if (is_null($offset)) {
         $this->statuses[] = $value;
      } else {
         $this->statuses[$offset] = $value;
      }
   }

   public function offsetUnset($offset) {
      unset($this->statuses[$offset]);
   }

   public function rewind() {
      $this->position = 0;
   }

   public function valid() {
      return isset($this->statuses[$this->position]);
   }
}

?>
{{< / highlight >}}

Cool, so now let's see a contrived example of how one might use this.

{{< highlight php >}}
<?php

<select id="statusId" name="statusId">
	foreach ($statusCollection as $status) {
		printf("<option value=\"%d\">%s</option>\n", $status["id"], $status["status"]);
	}
</select>

<button name="btnShowActive"
	id="btnShowActive"
	onclick="window.location='/?statusId=<?= $statusCollection.getIdByStatus('Active') ?>';">Show Active
</button>

?>
{{< / highlight >}}

Happy coding!
