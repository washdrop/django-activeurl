================
django-activeurl
================

Easy-to-use active URL highlighting for Django

.. image:: https://img.shields.io/travis/hellysmile/django-activeurl.svg
    :target: https://travis-ci.org/hellysmile/django-activeurl

.. image:: https://img.shields.io/codecov/c/github/hellysmile/django-activeurl.svg
    :target: https://codecov.io/gh/hellysmile/django-activeurl

.. image:: https://img.shields.io/pypi/v/django_activeurl.svg
    :target: https://pypi.python.org/pypi/django-activeurl

Updates
=======

This version of django-activeurl is compatible with django
up to version 4.1 and dropped support for version before
3.2. Therefore python > 3.8 is required.

Features
========

* automatic highlighting of currently active ``<a>`` tags via CSS class
* automatic highlighting of parent ``<a>`` tags for menus
* removes boring / hardcoded stuff from your life!

* href="#" never causes highlighting for compatibility with techniques such as
  bootstrap nav.

Usage
=====

After loading the template library via

.. code-block:: html+django

    {% load activeurl %}

the following code snippet will be rendered like this if `request.full_path()` starts with `/some_page/`:

.. code-block:: html+django

    {% activeurl %}
        <ul>
            <li> <!-- this <li> will render as <li class="active"> -->
                <a href="/some_page/">
                    some_page
                </a>
            </li>
            <li>
                <a href="/another_page/">
                    another_page
                </a>
            </li>
        </ul>
    {% endactiveurl %}

**Note:**
The content of ``{% activeurl %}…{% endactiveurl %}`` must have valid root tag (i.e.
``<ul>`` or ``<div>``, etc) -- otherwise an exception will be raised.

Installation
============

Python 2.7+, 3.4+ are supported.

1. Install the *stable* version using ``pip``::

       pip install django-activeurl

   or install the *in-development* version using ``pip``::

       pip install -e git+git://github.com/hellysmile/django-activeurl#egg=django_activeurl


2. In your ``settings.py`` add the following:

   .. code-block:: python

       INSTALLED_APPS = (
           ...
           'django_activeurl',
           ...
       )

       TEMPLATE_CONTEXT_PROCESSORS = (
           ...
           'django.core.context_processors.request',
           ...
       )

3. The *lxml* library is required and thus additional libraries need to be installed to build it:

   * Ubuntu::

       sudo apt-get install libxml2 libxml2-dev libxslt-dev build-essential python-dev
       sudo ldconfig

   * Fedora::

       sudo yum groupinstall 'Development Tools'
       sudo yum install libxslt-devel libxml2 libxml2-devel python-devel
       sudo ldconfig

   * MacOS X::

       brew install libxml2 libxslt
       sudo update_dyld_shared_cache -force

   * Windows:
     A pre-built *lxml* binary can be found `here <http://www.lfd.uci.edu/~gohlke/pythonlibs/>`_

   * Clouds:
     There's a 99.99% chance that *lxml* will build out of the box.

Options
=======

menu ="yes|no" (default: "yes")
-------------------------------

Should hierarchical menus be supported? There are two different ways to declare an *active* status:

* the *starts-with* logic toggles the active state if ``request.get_full_path()`` starts with the
  contents of the ``<a href=`` attribute.

* the *equals* logic toggles the active state if ``request.get_full_path()`` is identical to the
  contents of the ``<a href=`` attribute.

You might want to use **starts-with logic** in hierarchical menus/submenus to not only highlight the current position but also every parent position. So, with ``request.get_full_path()`` being `/menu/submenu` the following snippet will render accordingly:

.. code-block:: html+django

    {% activeurl menu="yes" parent_tag="div" %}
        <div>
            <div>  <!-- This will render as <div class="active"> -->
                <a href="/menu/">
                    menu
                </a>
                <div>  <!-- This will also render as <div class="active"> -->
                    <a href="/menu/submenu/">
                        submenu
                    </a>
                </div>
            </div>
        </div>
    {% endactiveurl %}

The **equals** logic works best for non-hierarchical menus where only those items should be highlighted whose ``href``-attribute perfectly match ``request.get_full_path()``:

.. code-block:: html+django

    {% activeurl menu="no" parent_tag="div" %}
        <div>
            <div>
                <a href="/menu/">
                    menu
                </a>
            </div>
            <div>
                <a href="/menu/submenu/">
                    submenu
                </a>
            </div>
        </div>
    {% endactiveurl %}

ignore_params ="yes|no" (default: "no")
---------------------------------------

``ignore_params`` will ignore GET parameters of URLs, e.g.
*/accounts/login/* will match */accounts/login/?next=/accounts/signup/*.

parent_tag ="div|li|self|…" (default: "li")
-------------------------------------------

``parent_tag`` defines that a parent element -- and not the ``<a>`` tag itself -- should be declared *active* when there's a match in URLs. When you need to change the CSS class of the ``<a>`` tag, just enter "self".

css_class ="<string>" (default: "active")
-----------------------------------------

Defines what CSS class to add to an active element.

Configuration
=============

The default options can be set in ``settings.py`` as well:

.. code-block:: python

    ACTIVE_URL_KWARGS = {
        'css_class': 'active',
        'parent_tag': 'li',
        'menu': 'yes',
        'ignore_params': 'no'
    }
    ACTIVE_URL_CACHE = True
    ACTIVE_URL_CACHE_TIMEOUT = 60 * 60 * 24  # 1 day
    ACTIVE_URL_CACHE_PREFIX = 'django_activeurl'

By default *django-activeurl* will try to retrieve a previously rendered HTML node from Django's caching backend before active URLs are looked for and a new HTML tree is built. You can disable the cache with ``ACTIVE_URL_CACHE = False``.

In addition, ``ACTIVE_URL_CACHE_TIMEOUT`` can be used to define a timeout for keys to expire. The default value is one day.

The last configuration option is ``ACTIVE_URL_CACHE_PREFIX`` (which is ``django_activeurl`` by default) and defines which name to use in Django's caching backend.

Tests
-----

::

    pip install tox
    tox


Jinja2
======

Vanilla `Jinja2 <https://github.com/mitsuhiko/jinja2>`_ configuration:

.. code-block:: python

    from jinja2 import Environment

    from django_activeurl.ext.django_jinja import ActiveUrl


    env = Environment(
        extensions=[ActiveUrl]
    )

Options can be omitted:

.. code-block:: jinja

    {% activeurl css_class="active", menu="yes", parent_tag="li", ignore_params="no" %}
        <ul>
            <li>
                <a href="/page/">page</a>
            </li>
            <li>
                <a href="/other_page/">other_page</a>
            </li>
        </ul>
    {% endactiveurl %}

If you're using `django-jinja <https://github.com/niwibe/django-jinja>`_
you need to load the ``ActiveUrl`` in ``settings.py``.

Django 1.8+ Jinja2 environment loader example can be found in `tests <https://github.com/hellysmile/django-activeurl/blob/master/tests/jinja_config.py>`_.

Background
==========

For building the HTML element tree *django-activeurl* uses `lxml
<http://pypi.python.org/pypi/lxml/>`_, which is one of the best HTML parsing
tools around. More info and benchmarks can be found at `habrahabr.ru
<http://habrahabr.ru/post/163979/>`_ (in russian). Note that there's no
content rebuilding inside the template tag when no active URLs are found, so
there's no impact on performance.
