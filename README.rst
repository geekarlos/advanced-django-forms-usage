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
    
Stuff:

* The big Anti-pattern
* Using forms for defining NoSQL schema
