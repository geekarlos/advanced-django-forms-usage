===========================
advanced-django-forms-usage
===========================

My talk for PyCon 2013: https://us.pycon.org/2013/schedule/presentation/101/

Right now this is just notes and ideas. 

Architecture of forms.Form:

.. code-block:: python

    forms.Form # class
    form = forms.Form() # object
    form.fields # list
    form.fields[0] # dictionary?
    
Abstract
========

Django forms are really powerful but there are edge cases, especially with class based views, that can cause a bit of anguish. This talk will go over how to handle many common solutions not currently described in the core documentation. It will also cover useful third-party libraries.

Material covered::

1. Django forms in functional views [5 min]

    * Economize your code with the request.POST practice
    
    * Understanding the ramifications of the request.POST practice
    
2. Django forms in class based views [5 min]

    * UpdateView without a slug
    * Two or more forms in a single view

3. Modifying forms on the fly [5 min]

    * Adding fields
    * Changing widgets
    
4. Using forms to provide schema for schemaless datastores [5 min]

5. Coverage on two 3rd party form libraries you should consider using [5 min]

    * django-crispy-forms
    * django-floppy-forms
    
6.Third party libraries that get twitter bootstrap into your forms [2 min]

7. Adding date widgets the easy way [3 min]


Forms Validate Dictionaries
===========================

.. code-block:: python

    import logging
    
    from django.http import HttpResponse
    from django.http.request import QueryDict
    from django.utils.datastructures import MultiValueDict
    
    logger = logging.getLogger(__main__)
    
    def my_form_view(request):
        logging.debug(isinstance(request.POST, QueryDict)  # True
        logging.debug(issubclass(QueryDict, (MultiValueDict, dict))  # True
        return HttpResponse