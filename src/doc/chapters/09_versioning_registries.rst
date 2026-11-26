
.. _chapter-apx9:

.. _section-9:

Versioning, Registries and Constants
====================================

.. _section-9.1:

Versioning rules
--------------------

The wire messages carry no protocol version member; the Evidence format is anchored by the ``typ`` value ``apx+jwt`` (:ref:`Section 4.5 <section-4.5>`) and the Envelope format by its media type (:ref:`Section 4.1 <section-4.1>`). This document is draft v0.1.


.. _section-9.2:

Fixed constants
-------------------

The fixed constants of this version: challenge TTL of 30 seconds; evidence window of at most 30 seconds; ``eat_nonce`` of 32 bytes; ``challenge_id`` of 8 bytes; the two digests of :ref:`Section 4.3 <section-4.3>`.

.. _section-9.3:

Registries
--------------

.. _section-9.3.1:

Envelope formats
""""""""""""""""""""""

The register Envelope format types (:ref:`Section 4.2 <section-4.2>`, advertised via the discovery document's ``accepts.envelope_formats``, :ref:`Section 3.3 <section-3.3>`):

.. list-table::
    :header-rows: 1
    :widths: 20 80

    * - Type
      - Definition
    * - ``sigstore-bundle``
      - Sigstore bundle v0.3 (DSSE envelope, ``application/vnd.in-toto+json`` payload); register signature: ECDSA P-256 over SHA-256, ASN.1 DER, signing input = the DSSE PAE (:ref:`Section 4.2 <section-4.2>`)

Implementations reject unknown format types.

.. _section-9.3.2:

Statement class keys
""""""""""""""""""""""""""

.. list-table::
    :header-rows: 1
    :widths: 12 40 48

    * - Key
      - predicateType
      - Format defined by
    * - ``manifest``
      - unconstrained (no pinned predicate type - build-tool manifests vary across builders and deployments)
      - OCI Image Format Specification
    * - ``provenance``
      - ``https://slsa.dev/provenance/v1``
      - in-toto attestation v1 statement; SLSA Provenance v1
    * - ``sbom``
      - ``https://spdx.dev/Document``
      - SPDX 2.3; in-toto SPDX predicate
    * - ``cbom``
      - ``https://cyclonedx.org/bom``
      - CycloneDX 1.6 (cryptographic-asset components); in-toto predicate
    * - ``obom``
      - ``https://cyclonedx.org/bom``
      - CycloneDX 1.6 (operating-system/container components); in-toto predicate
    * - ``mlbom``
      - ``https://cyclonedx.org/bom``
      - CycloneDX 1.6 (machine-learning-model components); in-toto predicate

.. _section-9.3.3:

Register statement types
""""""""""""""""""""""""""

.. list-table::
    :header-rows: 1
    :widths: 40 60

    * - ``_type``
      - Format defined by
    * - ``https://in-toto.io/Statement/v0.1``
      - in-toto Statement v0.1 (the form emitted by stock cosign)
    * - ``https://in-toto.io/Statement/v1``
      - in-toto Statement v1

Implementations reject unknown statement types.

