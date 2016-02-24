---
author: "Adam Presley"
date: 2008-03-06T04:38:00Z
tags: ["development", "c++", "opengl"]
title: "Learning OpenGL"
slug: "learning-opengl"
---

In the last two days I have blown the dust film off of the old C++
compiler and jumped back into the world of graphic programming in
OpenGL. I have maintained a fantasy for many years now of getting into
game programming. Being such a fascinating and challenging area of
development, as well as the fact that I enjoy video games, has prompted
me to start looking into higher education on the matter.

I am not a college graduate and I have worked hard to get into the
software development arena, but now I find that I must seek higher
education to get into the arena of game development. I have looked into
a couple of schools, mainly [SMU's GuildHall](http://guildhall.smu.edu/) and [UAT's Bachelor of
Science: Game Programming](http://www.gamedegree.com/) degree programs. There are a couple of
others I've seen, so I haven't made a choice yet, as I still need to
investigate further, not to mention calling all these people. Meanwhile,
to get started, I am refreshing my C++ skills, and jumping onto every
OpenGL tutorial I can find.

Right now I am getting my head wrapped around
correctly mapping vertex points in the 3D space, my biggest confusion
being what corner vertex to start with based on the direction the front
face points, and then which direction to continue adding vertices
(counter-clockwise I am told). Once I get that straight I should have no
issue with texture mapping coordinates, as I understand their origin and
directions, and how to map them to a vertex.

So basically the coordinate system works with the center screen as
origin. The X axis runs to the left and right, with the left side of the
origin going into the negative numbers, and the right side running
positive. The Y axis runs up and down, with the up direction being
positive, and the down direction being negative. The Z axis runs into
and out of the screen. The further into the direction of the screen you
go the lower the number (negative numbers), and the further out you
come, toward the viewer (me), the higher the number (positive).

Now I am learning about texture mapping, and the first thing I had to
overcome there was the fact that a lot of tutorials I was looking at
were using GLAUX for loading images for textures. It appears that
support for GLAUX has been dropped, so I had to spend a little time
looking for either a good reference on loading various image formats, or
find a nice library that does that for me. Well, I settled on [DevIL](http://openil.sourceforge.net/)
which is a full featured image loading and manipulation library written
in C++. Quite featured and offers native OpenGL support for loading
textures. Once that was solved it was a simple matter of learning to map
a corner of a texture to a vertex on a quad. I'm still a little fuzzy on
some of the details, but it will come with practice.

The part that I dread the most? All of the complex math. I've never been
a good math student, and here I will have to learn oodles of it! Ah
well, that is the sacrifice I must make to live the dream baby!
