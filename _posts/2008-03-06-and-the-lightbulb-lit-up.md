---
layout: post
title: And the lightbulb lit up...
date: 2008-03-06 19:36:00
author: Adam Presley
status: Published
tags: development c++ opengl
slug: and-the-lightbulb-lit-up
---

In my last post I mentioned I was having issue with remembering the
order to draw vertices for quads. Ha... the light came on tonight while
I was working on the various tutorials I've come across. The bottom and
back faces of a cube I am actually drawing the quad from the perspective
of viewing the back face. Apparently when drawing a quad (or triangle
for that matter) you draw the vertices in counter-clockwise order to
draw the **FRONT** face as the side facing the observer, or you draw the
vertices clockwise to draw the **BACK** face as the side facing the
observer. Boy, once I got that, it sure seemed a lot easier!

As an example of drawing a basic cube, colored all pretty and green.

![OpenGL Cubes](http://s3.amazonaws.com/www.adampresley.com/posts/openglblocksscreenie.jpg)

{% highlight c++ %}
glBegin(GL_QUADS);

// Front face
glNormal3f(0.0f, 0.0f, 1.0f);
glColor3f(0.0f, 1.0f, 0.0f);
glVertex3f(-1.2f, -0.5f, 0.5f);
glVertex3f(1.2f, -0.5f, 0.5f);
glVertex3f(1.2f, 0.5f, 0.5f);
glVertex3f(-1.2f, 0.5f, 0.5f);

// Back face
glNormal3f(0.0f, 0.0f, -1.0f);
glColor3f(0.0f, 1.0f, 0.0f);
glVertex3f(-1.2f, -0.5f, -0.5f);
glVertex3f(-1.2f, 0.5f, -0.5f);
glVertex3f(1.2f, 0.5f, -0.5f);
glVertex3f(1.2f, -0.5f, -0.5f);

// Top face
glNormal3f(0.0f, 1.0f, 0.0f);
glColor3f(0.0f, 1.0f, 0.0f);
glVertex3f(-1.2f, 0.5f, -0.5f);
glVertex3f(-1.2f, 0.5f, 0.5f);
glVertex3f(1.2f, 0.5f, 0.5f);
glVertex3f(1.2f, 0.5f, -0.5f);

// Bottom face
glNormal3f(0.0f, -1.0f, 0.0f);
glColor3f(0.0f, 1.0f, 0.0f);
glVertex3f(-1.2f, -0.5f, -0.5f);
glVertex3f(1.2f, -0.5f, -0.5f);
glVertex3f(1.2f, -0.5f, 0.5f);
glVertex3f(-1.2f, -0.5f, 0.5f);

// Right face
glNormal3f(1.0f, 0.0f, 0.0f);
glColor3f(0.0f, 1.0f, 0.0f);
glVertex3f(1.2f, -0.5f, -0.5f);
glVertex3f(1.2f, -0.5f, 0.5f);
glVertex3f(1.2f, 0.5f, 0.5f);
glVertex3f(1.2f, 0.5f, -0.5f);

// Left face
glNormal3f(-1.0f, 0.0f, 0.0f);
glColor3f(0.0f, 1.0f, 0.0f);
glVertex3f(-1.2f, -0.5f, -0.5f);
glVertex3f(-1.2f, -0.5f, 0.5f);
glVertex3f(-1.2f, 0.5f, 0.5f);
glVertex3f(-1.2f, 0.5f, -0.5f);

glEnd();
{% endhighlight %}