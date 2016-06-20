Usage
=====

Installing
----------

Installation is quite simple and can be done via pip::

    pip install django-afip

Do keep in mind that requirements.txt points to a fork of ``suds-py``, since
upstream is broken and *will fail*.

You'll then need to configure your project to use it by adding it to
settings.py::

    INSTALLED_APPS = (
        ...
        'django_afip',
        ...
    )

Getting started
---------------

First of all, you'll need to create a TaxPayer instance, and upload the related
ssl key and certificate (for authorization).

django-afip includes admin views for every model included, and it's the
recommended way to create TaxPayer objects.

Once you have created a TaxPayer, you'll need its points of sales. This, again,
can be done via the admin by selecting "fetch points of sales'. You may also
do this programmatically via ``TaxPayer.fetch_points_of_sales``.

Finally, you'll need to pre-populate certain models with AFIP-defined metadata
(ReceiptTypes, DocumentTypes and a few others).

Rather than include fixtures which require updating over time, we fetch this
information from AFIP's web services via an included django management command.
This command is idempotent, and running it more than once will not create any
duplicate data. To fetch all metadata, simply run::

    python manage.py afipmetadata

This metadata can also be downloaded programmatically, via
``models.populate_all()``.

You are now ready to start creating and validating receipts. While you may do
this via the admin as well, you probably want to do this programmatically or via
some custom view.

PDF Receipts
------------

Version 1.2.0 introduced PDF-generation for validated receipts. These PDFs are
backed by the ``ReceiptPDF`` model.

There are two ways of creating these objects; you can do this manually, or via
these steps:

* Creating a ``TaxPayerProfile`` object for your ``TaxPayer``, with the right
  default values.
* Create the PDFs via ``ReceiptPDF.objects.create_for_receipt()``.
* Add the proper ``ReceiptEntry`` objects to the ``Receipt``. Each
  ``ReceiptEntry`` represents a line in the resulting PDF file.

The PDF file itself can then be generated via::

    # Save the file as a model field into your MEDIA_ROOT directory:
    receipt_pdf.save_pdf()
    # Save to some custom file-like-object:
    receipt_pdf.save_pdf_to(file_object)

The former is usually recommended since it allows simpler interaction via
standard django patterns.

Exposing receipts
~~~~~~~~~~~~~~~~~

Generated PDF files may be exposed both as pdf or html with an existing view,
for example, using::

    url(
        r'^invoices/pdf/(?P<pk>\d+)?$',
        views.ReceiptPDFView.as_view(),
        name='receipt_view',
    ),
    url(
        r'^invoices/html/(?P<pk>\d+)?$',
        views.ReceiptHTMLView.as_view(),
        name='receipt_view',
    ),

You'll generally want to subclass this view, and add some authorization checks
to it. If you want some other, more complex generation (like sending via
email), these views should serve as a reference to the PDF API.

The template used for the HTML and PDF receipts is found in
``templates/django_afip/invoice.html``. If you want to override the default (you
probably do), simply place a template with the same path/name inside your own
app, and make sure it's listed *before* ``django_afip`` in ``INSTALLED_APPS``.