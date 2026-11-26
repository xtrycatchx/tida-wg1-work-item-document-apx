
.. _chapter-apx6:

.. _section-6:

Error Handling
=================

Error responses use the body defined in :ref:`Section 4.1 <section-4.1>`.

Required status codes per endpoint. The Clerk MUST return the listed code for each condition; conditions not listed here are deployment concerns.

.. list-table::
    :header-rows: 1
    :widths: 28 10 62

    * - Endpoint
      - Status
      - Condition
    * - ``register``
      - ``400``
      - Structurally or semantically invalid Envelope (:ref:`Section 5.1.2 <section-5.1.2>` checks 1, 3–9), or custody evidence that fails :ref:`Section 5.1.3 <section-5.1.3>`
    * - ``register``
      - ``401``
      - Envelope DSSE signature verification failed (:ref:`Section 5.1.2 <section-5.1.2>` check 2)
    * - ``register``
      - ``409``
      - Identical content hash already registered, or a re-registration whose ``iat`` is not later than the stored ``iat`` (:ref:`Section 5.1.6 <section-5.1.6>`)
    * - ``register``
      - ``500``
      - Registration store failure
    * - ``challenge``
      - ``400``
      - Malformed Provenance ID (:ref:`Section 5.2.1 <section-5.2.1>`)
    * - ``challenge``
      - ``404``
      - Unknown ``provenance_id`` (:ref:`Section 5.2.1 <section-5.2.1>`)
    * - ``evidence``
      - ``400``
      - Invalid JWS structure or header (:ref:`Section 4.5 <section-4.5>`; the Evidence keeps its own JOSE header)
    * - ``evidence``
      - ``401``
      - Evidence signature verification failed (:ref:`Section 5.2.3 <section-5.2.3>` check 6)
    * - ``evidence``
      - ``403``
      - Unknown, consumed, or expired challenge; claim mismatch; ``exp`` − ``iat`` longer than the evidence window; missing, unsupported ``eat_profile``; missing, malformed, or mismatched measurement (:ref:`Section 5.2.3 <section-5.2.3>` checks 1–5, 7; :ref:`Section 5.2.4 <section-5.2.4>`)
    * - ``evidence``
      - ``404``
      - Unknown registration (:ref:`Section 5.2.3 <section-5.2.3>` check 5)
    * - ``evidence``
      - ``500``
      - Internal service failure

Reason strings inside ``{"error": ...}`` are informative: deployments MAY choose their phrasing, and clients MUST NOT branch on specific reason text. The status code is the only normative signal.
