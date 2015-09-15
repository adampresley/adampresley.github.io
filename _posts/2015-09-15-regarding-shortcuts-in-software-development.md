---
layout: post
title: Regarding Shortcuts in Software Development...
date: 2015-09-15 16:00
author: Adam Presley
tags: development
comments: true
---
Many times we as software engineers and architects are asked, or even required, to come up with *shortcuts*, or to be *creative* in our solutions. This is often the case when external factors and pressures, such as time commitments, budget constraints, and more. There must always be a balance between what is needed to achieve an implementation and the needs of the business. However, shortcuts, in my experience, often lead to painful results.

<!-- excerpt -->

When shortcuts are taken during the planning and implementation of a software system a sacrifice is often made. That sacrifice usually comes in the form of some requirement not being met. For the sake of this discussion let's define a requirement as all *functional*, *non-functional*, *architectural*, and *design* objectives of a software system. A software system is considered **correct** when it meets a set of requirements. When a software system meets its requirements it is **correct** and will perform its job and function for users, systems, or whoever the intended audience is.

Requirements come in many forms. For example, a *functional* requirement may be that the software provide a means for a user to log in and view past shopping transactions. A *non-functional* requirement might further dictate that these transactions must be secured, both in transit and at rest. Your architecture team may impose further requirements that all transaction payment processing be done asynchronously. As such the design requirement may state that a queue system be put in place where all such transactions must be processed through.

With all of this in mind, any software system that does not meet all of these requirements can be said to **not be correct**. Therefore it could be said that any *shortcut* taken to account for external constraints and factors, without considering balance and the impact to the software system, can result in *incorrect software*. It is our duty as stewards of a software system to be mindful of these trade-offs, and to call out such issues to key decision makers when the word "shortcut" is thrown around.

How do *you* feel about shortcuts when developing software?