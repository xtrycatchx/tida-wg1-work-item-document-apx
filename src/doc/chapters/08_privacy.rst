
.. _chapter-apx8:

.. _section-8:

Privacy Considerations
=========================

* The Provenance ID is opaque. Because its generation is implementation-defined (:ref:`Section 5.1.4 <section-5.1.4>`), deployments SHOULD ensure that it carries no personal data. Because it is stable (:ref:`Section 5.1.4 <section-5.1.4>`), deployments SHOULD treat it as a correlation handle across requests.
* Statement payloads are Registrar-provided and may embed supplier, organization, or dependency names (e.g. an SBOM); deployments SHOULD treat statement contents as potentially sensitive.
* The credential carries only the statement digest, not statement content (:ref:`Section 4.6.2 <section-4.6.2>`). The digest is not a confidentiality control: a guessable statement can be recognised from its digest. The Clerk keeps no statement content (:ref:`Section 2.4 <section-2.4>`), so a party that needs it must obtain it from the Registrar or the statement producers.
