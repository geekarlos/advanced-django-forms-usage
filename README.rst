===========================
advanced-django-forms-usage
===========================

My talk for PyCon 2013: https://us.pycon.org/2013/schedule/presentation/101/

Right now this is just notes and ideas. 

Architecture of forms.Form:


    
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

Dynamically adding fields to a FBV
====================================

.. code-block:: python

    def my_form_view(request, template_name="myapp/form.html"):
    
        form = MyForm(request.POST or None)
        form['favorite_icecream'] = forms.ChoiceField(
            label="What is your favorite flavor from this list?",
            choices=((0, "Chocolate"), (1, "Vanilla"), (2, "Berry")),
            widget=forms.RadioSelect,
            required=True
        )
        if form.is_valid():
            # lets get user's favorite ice cream.
            # You can do anything you want with it
            favorite_icecream = form.cleaned_data['favorite_icecream']
    
            form.save()
            return redirect('home')
        return form

Dynamically adding fields to a CBV
====================================

.. code-block:: python

    def flavor_field():
        return forms.ChoiceField(
                label="What is your favorite flavor from this list?",
                choices=((0, "Chocolate"), (1, "Vanilla"), (2, "Berry")),
                widget=forms.RadioSelect,
                required=True
            )
        

    class MyCustomFormView(UpdateView)
    
        def get_form(self, form_class):
            """ Overloads the default FormMixin.get_form() method
            """
            form = form_class(**self.get_form_kwargs())
            form = flavor_field()
            return form

        def form_valid(self, form):
            # lets get user's favorite ice cream.
            # You can do anything you want with it
            favorite_icecream = form.cleaned_data['favorite_icecream']

            return super(MyCustomFormView, self).form_valid(form)


Forms Validate Dictionaries
===========================

.. code-block:: python

    {"title": "Advanced Django Forms Usage"}
    {"title": ""}


.. code-block:: python

    import logging
    
    from django.http import HttpResponse
    from django.http.request import QueryDict
    from django.utils.datastructures import MultiValueDict
    
    logger = logging.getLogger(__main__)
    
    def my_post_view(request):
        logger.debug(isinstance(request.POST, QueryDict)  # True
        logger.debug(issubclass(QueryDict, (MultiValueDict, dict))  # True
        return HttpResponse()
        
Form Object Architecture
=========================

The Basics
------------

.. code-block:: python

    forms.Form # class
    form = forms.Form() # object
    form.fields # iterable
    form.fields['title'] # dictionary?

Example
----------

.. code-block:: python

    import logging
    from django.http import HttpResponse
    from .forms import MyForm

    logger = logging.getLogger(__main__)

    def my_view(request):
        # instantiate the MyForm class
        form = MyForm(request.POST or None)  
        
        # An iterable of the form fields in order of display
        logger.debug(form.fields)
        logger.debug(form.fields['title'])
        return HttpResponse()

request.POST or None
=====================

Sample Form:

.. code-block:: python

    from django import forms

    class MyForm(forms.MyForm):
        name = forms.CharField()
        

Standard View:

.. code-block:: python

    from django.shortcuts import render, redirect
    
    from .forms import MyForm
    
    def my_view(request, template_name="myapp/my_form.html"):
    
        if request.method == 'POST':
            form = MyForm(request.POST)  # Form #1!
            if form.is_valid(): # nested if!
                # Custom logic here
                return redirect('/')
        else:
            form = MyForm()  # Form #2!
        return render(request, template_name, {'form': form})

Shortcut view:

.. code-block:: python

    from django.shortcuts import render, redirect

    from .forms import MyForm
    
    def my_view(request, template_name="myapp/my_form.html"):
    
        form = MyForm(request.POST or None)
        if form.is_valid():
            # custom logic here
            return redirect('/')
        return render(request, template_name, {'form': form})

Shortcut or anti-pattern
========================

.. code-block:: python

    if True:
        do_x()
    if False:
        do_y()
        
CBV: Modifying is_valid/invalid
=================================

.. code-block:: python

    class MyView(FormView|CreateView|UpdateView):
        def form_valid(self, form):
            # Do custom logic here
            return super(MyView, self).form_valid(form)
        
        def form_invalid(self, form):
            # Do custom logic here
            return super(FlavorCreateView, self).form_invalid(form)

Don't Rewrite Models
======================

.. code-block:: python

    from django.db import models
    
    class MyModel(models.Model):
    
        name = models.CharField(max_length=50, blank=True)
        age = models.IntegerField(blank=True, null=True)
        profession = models.CharField(max_length=100, blank=True)
        bio = models.TextField(blank=True)

The Wrong Way
--------------

.. code-block:: python

    from django import forms
    
    from .models import MyModel
    
    class MyModelForm(forms.ModelForm):
    
        title = forms.CharField(max_length=100, required=True)
        age = forms.IntegerField(required=True)
        profession = forms.CharField(required=True)
        bio = forms.TextField(required=True)

        class Meta:
            model = MyModel

The Right Way
--------------

.. code-block:: python

    from django import forms
    
    from .models import MyModel

    class MyModelForm(forms.ModelForm):
        
        def __init__(self, *args, **kwargs):
            super(MyModelForm, self).__init__(*args, **kwargs)
            self.fields['name'].required = True
            self.fields['age'].required = True
            self.fields['profession'].required = True
            self.fields['profession'].help_text = "Hello, PyCon!"

        class Meta:
            model = MyModel


NoSQL Form Example
--------------------

.. code-block:: python

    from django import forms
    
    import nosql  # Use the nosql library of your choice
    
    class NoSqlBaseFormMixin(object):

        def save(self, commit=True):
            if form.errors:
                raise ValueError("No form save because of invalid data")
            if commit:
                if 'pk' in self.cleaned_data.pk: 
                    # add data
                    instance = NoSqlLib.update(
                        **self.cleaned_data
                    )
                else:
                    # update data
                    instance = nosql.insert(
                        **self.cleaned_data
                    )
                return instance
            return self.cleaned_data
            
.. code-block:: python

    from django import forms
    
    from nosqlforms import NoSqlBaseFormMixin
    
    class NoSqlForm(NoSqlBaseFormMixin, forms.Form):
    
        title = forms.CharField(max_length=100, required=True)
        age = forms.IntegerField(required=True)
        profession = forms.CharField(required=True)
        bio = forms.TextField(required=True)

.. code-block:: python

    from django.shortcuts import render, redirect

    from .forms import NoSqlForm
    
    def data_form(request, template_name="data/data_form.html"):
        form = NoSqlForm(request.POST or None)
        if form.is_valid():
            form.save()
            return redirect('/')
        return render(request, template_name, {'form': form})
    

.. code-block:: python

    from django.views.generic import FormView
    
    from .forms import NoSqlForm
    
    class DataFormView(FormView):
    
        form_class = NoSqlForm
        template_name = "data/data_form.html"
        
        def form_valid(self, form):
            form.save()
            return super(DataFormView, self).form_valid(form)

request.POST or None with files
================================

.. code-block:: python

    from django.shortcuts import render, redirect
    
    def my_view(request, template_name="myapp/my_form.html"):
        """ Code donated by Audrey Roy """
    
        form = MyForm(request.POST or None, request.FILES or None)
        if form.is_valid():
            # do something with the file here
            return redirect('home')
        return render(request, template_name, {'form': form})

request.POST or None with ModelForms
====================================

.. code-block:: python

    from django.shortcuts import (
        render, redirect, get_object_or_404
    )
    
    from .models import MyModel

    def my_view(request, slug=slug, template_name="myapp/my_form.html"):
        """ Code donated by Audrey Roy """
        mymodel = get_object_or_404(MyModel, slug=slug)
        form = MyForm(request.POST or None, instance=mymodel)
        if form.is_valid():
            mymodel = form.save(commit=False)
            mymodel.edited_at_pycon = true
            mymodel.save()
            return redirect('home')
        return render(request, template_name, {'form': form})

Try it with inheritance!
====================================

.. code-block:: python

    class BaseEmailForm(forms.Form):
        email = forms.EmailField(_("Email"))
        confirm_email = forms.EmailField(_("Email 2"))

    class ContactForm(BaseEmailForm):
        message = forms.CharField(_("Message"))

        def __init__(self, *args, **kwargs):
            super(ContactForm, self).__init__(*args, **kwargs):
            self.fields['confirm_email'].label = _("Confirm your email")
            self.fields['confirm_email'].help_text = _("We want to make sure!")

