---
layout: post
title: Using a Priority Queue in Groovy
date: 2010-11-08 22:04:00
author: Adam Presley
status: Published
tags: development groovy java
slug: using-a-priority-queue-in-groovy
---

Although the title suggests something specific to Groovy the concept
demonstrated in this example will work in good old Java too, with a
couple of tweaks of course. In this blog entry we will look at what a
priority queue is, and how we can use Java to implement one.

Imagine we need to implement a ticket processing system where we need to
process ticket holder information. Further imagine that we need to
process these ticket holders in order of priority, and priority is
defined by the type of ticket that was purchased. For this demonstration
we will have the following ticket types and priorities, with a lower
priority number being a higher priority: Priority Club (1), Season
Ticket-holder (2), Online Purchase (3), and Ticket Booth (4). The first
thing to do is define an enumerator that provides these very definitions
we just laid out.

	:::groovy
	package com.adampresley.PriorityQueueExample

	enum TicketType {
	   TICKET_BOOTH("Ticket Booth", 4),
	   ONLINE("Online Purchase", 3),
	   SEASON_TICKETS("Season Ticket-holder", 2),
	   PRIORITY_CLUB("Priority Club", 1)

	   private final String value
	   private final int priority
	   public TicketType(value, priority) {
	      this.value = value
	      this.priority = priority.toInteger()
	   }

	   public String toString() {
	      value
	   }

	   public int getPriority() {
	      priority
	   }
	}

Now that we have an enumeration for ticket types we should probably
define a ticket holder. For this example a ticket holder will be really
simple, providing a name and the ticket type they hold. We will also
override the **toString()** method to ensure that when we print this
object to the console we see the "pretty" version. Here's that code.

	:::groovy
	package com.adampresley.PriorityQueueExample

	class TicketHolder {
	   def name
	   TicketType ticketType

	   public TicketHolder() {
	      TicketHolder("", TicketType.TICKET_BOOTH)
	   }

	   public TicketHolder(name, ticketType) {
	      this.name = name
	      this.ticketType = ticketType
	   }

	   @Override
	   public String toString() {
	      "${name} (${ticketType.toString()})"
	   }
	}

And once we have this guy defined let's put it to the test. The
**java.util** package offers a class called **PriorityQueue**
that offers a means to add items to a queue that are sorted by some
comparison. By default, if you do not provide a comparator, natural
sorting is used. In our case we will provide a comparator that will
first compare by priority. If the priority between two items is the same
we will then sort by name. In the final example we create this queue,
then add a bunch of ticket holders to it. The final block of code will
loop and call **poll()** until we have no more items. Each call to
**poll()** will give us the next item in the queue.

	:::groovy
	package com.adampresley.PriorityQueueExample

	class Main {
	   static main(args) {
	      /*
	       * Create a priority queue with a comparator to determine what has priority.
	       * If the priority is the same, sort by name as a secondary sort.
	       */
	      def queue = new PriorityQueue(25, new Comparator<ticketholder>() {
	         public int compare(TicketHolder holder1, TicketHolder holder2) {
	            if (holder1.getTicketType().getPriority() == holder2.getTicketType().getPriority())
	               holder1.getName() <=> holder2.getName()
	            else
	               holder1.getTicketType().getPriority() <=> holder2.getTicketType().getPriority()
	         }
	      })

	      /*
	       * Add a couple of regular buyers who bought tickets from
	       * the ticket booth.
	       */
	      queue.add(new TicketHolder("Ben Nadel", TicketType.TICKET_BOOTH))
	      queue.add(new TicketHolder("Ray Camdem", TicketType.TICKET_BOOTH))

	      /*
	       * Now add somebody who bough a ticket online. This person gets
	       * ahead in line before the other two.
	       */
	      queue.add(new TicketHolder("Maryanne Anello", TicketType.ONLINE))

	      /*
	       * Add a "Priority Club" member. This takes precedence in line ahead
	       * of all the other people.
	       */
	      queue.add(new TicketHolder("Adam Presley", TicketType.PRIORITY_CLUB))

	      /*
	       * Now add a season ticket holder, and another online buyer.
	       * Season ticket holders go in line before online.
	       */
	      queue.add(new TicketHolder("Cameron Duke", TicketType.SEASON_TICKETS))
	      queue.add(new TicketHolder("Jesse Roach", TicketType.ONLINE))


	      /*
	       * Display our queue as it currently stands.
	       */
	      def item = null

	      while ((item = queue.poll()) != null) {
	         println item
	      }
	   }
	}

You will notice when you run this example that the items are **not**
in the order we inserted them, but instead are in order of priority.
Pretty cool eh? Happy coding!!
