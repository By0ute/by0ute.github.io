---
layout: post
title:  "How to PyAlembic - My problems and their solutions to Alembic Python bindings"
date:   2016-02-10 23:14:00
last_modified_at:  2016-02-10 23:14:00
excerpt: "I recently needed to recompile the Alembic library and its Python version, under Fedora at work, and encountered several issues I finally overcame. So I thought about sharing it..."
categories: fix
tags:  alembic, c++, python, pyalembic, ilmbase, pyilmbase, fedora
image:
  feature: pixellou.jpg
  topPosition: 0px
bgContrast: dark
bgGradientOpacity: darker
syntaxHighlighter: no
---

How I finally recompile Alembic and PyAlembic
===================


Hey! *My office finally changed the OS of the workstations from an old OpenSuse to a newer Fedora*. So due to incompabilities with some system libraries, I had to **rebuild the Alembic library** for productions. 

[Get me to the solutions please!](#problems-vs-solutions)

What's [Alembic](http://alembic.io/)?
---------------

![Alembic Logo](http://opensource.imageworks.com/images/large/alembic.jpg)

> At the highest, most simplistic level, Alembic is "merely" a hierarchical sampled data storage format. It is intended to be used to store a baked representation of scene data, in the same vein as GTO or OBJ. It was designed to facilitate handoff of data between disciplines, vendors, and applications.
> At a lower, less simplistic level, Alembic is designed to be very efficient with its use of memory and disk space. Much effort has been directed towards automatic and transparent mechanisms for data de-duplication and reading/writing as little data as possible in order to fully represent the data that has been given to it.
> Alembic is not a live scenegraph, an asset management system, a renderer, or replacement for OpenGL.


**Basically, Alembic is a format for graphics data that became a standard, made by [Sony Pictures Imageworks](http://www.imageworks.com/) and [Industrial Light & Magic](http://www.ilm.com/)** 

So, if you want to get deep with it, there is a library in both C++ and Python to play with those files.

And there my quest began...
---------------------------

First of all, since [Google Code is dead](http://google-opensource.blogspot.fr/2015/03/farewell-to-google-code.html) (and [Alembic Google Code](https://code.google.com/archive/p/alembic/) with it), I went on the [Alembic Github](https://github.com/alembic/alembic). 
I realized that the master branch is way ahead of the last release ([1.5.8](https://github.com/alembic/alembic/releases/tag/1.5.8)) and that some of the fixes we had made inhouse were already integrated through pull requests.
So, I decided to try with the version on master instead of the lastest release.
I reused the scripts I had made to compile. After some tiny tricks for a smooth compilation, I met problems that other people faced according to the Alembic forum. But without any viable solution.
But luckily, I'm Taurus so I beat the code.

Problems vs Solutions
---------------------

*Coming Soon...*
