API
===

.. contents::

Core models
-----------

These are the code models which will normally be used for  ``Receipt`` validation.

.. autoclass:: django_afip.models.PointOfSales

    .. attribute:: number
      :type: int

      The number given to the POS by AFIP, usually presented in receipts as the
      first four digits.

    .. attribute:: issuance_type
      :type: str

      Indicates if this POS emits using CAE or CAE. This value is populated
      upon POS creation, but should never be outdated since this value is not
      supposed to change.

    .. attribute:: blocked
      :type: bool

      Indicates if this POS has been blocked. This value is populated upon POS
      creation, so may end up being out of date.

    .. attribute:: drop_date
      :type: datetime.date

      The date when this POS was dropped, in case it's been dropped. This value
      is populated upon POS creation, so may end up being out of date.

    .. attribute:: owner
      :type: TaxPayer

      The :class:`~.TaxPayer`: who owns this POS. The POS will only emit
      receipts on behalf of that taxpayer.

    .. attribute:: receipts

      All the receipts related to this POS, including those that have not been
      validated.

.. autoclass:: django_afip.models.Receipt

    .. attribute:: point_of_sales
      :type: PointOfSales

      The :class:`~.PointOfSales`: that emitted (or is going to emit) this
      receipt. It also holds the relationship to the TaxPayer who emits this
      receipt.

      The number given to the POS by AFIP, usually presented in receipts as the

    .. attribute:: receipt_type
      :type: ReceiptType

      The type for this receipt (e.g.: invoice, credit note, etc.). See
      :class:`~.ReceiptType`: for futher details.

    .. attribute:: concept
      :type: ConceptType

      Describes what type of exchange this receipt relates to: a product or a
      service. See :class:`~.ConceptType`: for futher details.

    .. attribute:: document_type
      :type: DocumentType

      The document type for the *recipient* of this receipt.

    .. attribute:: document_number
      :type: int

      The document number for the *recipient* of this receipt.

    .. attribute:: receipt_number
      :type: int

      The document number for this receipt. You *should not* set this field
      yourself; when you :meth:`~.validate` this receipt it will be set
      properly for you.

      Since the webservice expects consecutive numbers, whenever a new receipt
      is validated, its number is set to the following unused number. If you
      set this yourself, and don't validate this immediately, you DB will be
      inconsistent and further validations will fail.

    .. attribute:: issued_date
      :type: date

      Then date when this receipt is emmited. When validating a receipt, this
      date may be diverge from the current date by 5 days for good, and up to
      10 days for services.

    .. attribute:: total_amount
      :type: Decimal

      The total amount for this receipt. Must be equal to the sum of
      :attr:`~.net_untaxed`, :attr:`~.exempt_amount`, :attr:`~.net_taxed`, and
      all the related :class:`~.Tax` and :class:`~.Vat` objects.

    .. attribute:: net_untaxed
      :type: Decimal

      The total amount to for which taxes do not apply.
      For C-type receips, this value must be zero.
      This is what AFIP calls "ImpTotConc".

    .. attribute:: net_taxed
      :type: Decimal

      The total amount to which taxes apply.
      For C-type receipts, this is equal to the subtotal.
      This is what AFIP calls "ImpNeto".

    .. attribute:: exempt_amount
      :type: Decimal

      Only for categories which are tax-exempt.
      For C-type receipts, this must be zero.
      This is what AFIP calls "ImpOpEx".

    .. attribute:: service_start
      :type: datetime.Date

      The date when the service started to be performed.
      This field has no meaning when if :attr:`~concept` is not service.

    .. attribute:: service_end
      :type: datetime.Date

      The date when the service was completed.
      This field has no meaning when if :attr:`~concept` is not service.

    .. attribute:: expiration_date
      :type: datetime.date

      The expiration date for this receipt. That is, the date before money must
      have exchanged hands.

    .. attribute:: currency
      :type: Currency

      The :class:`~Currency` in which this receipt is issued. If this field is
      set, then :attr:`~currency_quote` should be set to the appropiate value
      for that date.

    .. attribute:: related_receipts

      Receipts related to this one. Generally, when making a refund, the credit
      note should be related to the original invoice.

      It is unclear if you are free to use this field to reflect other kind of
      relationships or not.

    .. autoproperty:: total_vat

    .. autoproperty:: total_tax

    .. autoproperty:: formatted_number

    .. autoproperty:: is_validated

    .. autoproperty:: validate

.. autoclass:: django_afip.models.ReceiptValidation
    :members:
.. autoclass:: django_afip.models.Tax
    :members:
.. autoclass:: django_afip.models.TaxPayer
    :members:
.. autoclass:: django_afip.models.Vat
    :members:

PDF-related models
------------------

These models are used only for PDF generation, or can be used for storing
additional non-validated metadata. You DO NOT need any of these classes
unless you intend to generate PDFs for receipts.

.. autoclass:: django_afip.models.ReceiptEntry
    :members:
.. autoclass:: django_afip.models.ReceiptPDF
    :members:
.. autoclass:: django_afip.models.TaxPayerProfile
    :members:
.. autoclass:: django_afip.models.TaxPayerExtras
    :members:

Metadata models
---------------

These models represent metadata like currency types or document types. Their
tables should be populated from AFIP's webservices, using the ``afipmetadata``
command.

.. autofunction:: django_afip.models.populate_all

.. autoclass:: django_afip.models.ConceptType
    :members:
.. autoclass:: django_afip.models.CurrencyType
    :members:
.. autoclass:: django_afip.models.DocumentType
    :members:
.. autoclass:: django_afip.models.Observation
    :members:
.. autoclass:: django_afip.models.ReceiptType
    :members:
.. autoclass:: django_afip.models.TaxType
    :members:
.. autoclass:: django_afip.models.VatType
    :members:

Managers
~~~~~~~~

Managers should be accessed via models. For example, ``ReceiptManager``
should be accessed using ``Receipt.objects``.

.. autoclass:: django_afip.models.ReceiptManager
    :members:
.. autoclass:: django_afip.models.ReceiptPDFManager
    :members:

QuerySets
---------

QuerySets are generally accessed via their models. For example,
``Receipt.objects.filter()`` will return a ``ReceiptQuerySet``.

.. autoclass:: django_afip.models.ReceiptQuerySet
    :members:
