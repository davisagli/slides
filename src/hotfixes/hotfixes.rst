:title: Patching Plone
:data-transition-duration: 300
:css: slideshow.css

----

Patching Plone
==============

with the power of Python
------------------------

|

David Glick
-----------

PyCologne, Feb 2014
~~~~~~~~~~~~~~~~~~~

----

The Problem
===========

Software has bugs

----

The Problem
===========

Some of them are security holes

----

The Problem
===========

You can't always upgrade

.. note::

   If we used PHP, we wouldn't have many options. cf. security releases of Drupal, Wordpress.

   We can do better.


----

Monkey Patching
===============

----

.. code:: python

 # frontend.py

 def render_link(url):
     return '<a href="{}">{}</a>'.format(url, url)

----

.. code:: python

 import cgi

 import frontend
 old_render_link = frontend.render_link

 def new_render_link(url):
     sanitized_url = cgi.escape(url)
     return old_render_link(sanitized_url)

 frontend.render_link = new_render_link


----

Plone hotfixes
==============

Organized monkey patching

.. note::

   Benefits:

   * Released on 4-month schedule
   * Can install as egg or simple module
   * Wrap in try/except so the hotfix can safely be added to old systems
   * Log success so admin can check if it applied

----

Problem with monkey patching
============================

If the system imports a module global,
and *then* the hotfix runs and patches that global,
the early imports still reference the unpatched code.

----

Marmoset patching
=================

.. image:: marmoset.jpg

Replace the function's bytecode instead of the
namespace's reference to the function.

----

.. code:: python

 code = """
 def old_render_link(url):
 	 pass
 
 def new_render_link(url):
     sanitized_url = cgi.escape(url)
     return old_render_link(sanitized_url)
 """
 from frontend import render_link
 exec code in frontend.render_link.func_globals
 frontend.old_render_link.func_code = frontend.render_link.func_code
 frontend.render_link.func_code = frontend.new_render_link.func_code

Questions?
==========

Email
  david@glicksoftware.com
IRC
  davisagli
