
.. _chapter-apx10:

.. _section-10:

Conformance
===============

An implementation conforms to this specification if it performs every check and rule marked MUST for its role, in the stated order where one is given.

**Registrar** (build time):

* The Registrar MUST NOT attempt to claim or supply a Provenance ID (:ref:`Section 5.1.4 <section-5.1.4>`).
* The Registrar SHOULD NOT hold decrypt rights on the wrapping key for ``software-sealed``.

**Clerk**:

* The Clerk MUST use its ``accepts`` as the enforcement set for the register format (:ref:`Section 3.3 <section-3.3>`).

**Workload** (run time):

* The Workload MUST NOT issue its own identity credential (:ref:`Section 4.6.2 <section-4.6.2>`).

**Credential consumer** (informative, out of scope for the wire protocol):

* A credential that lacks a REQUIRED claim (:ref:`Section 4.6.2 <section-4.6.2>`) is malformed.
* A credential consumer MAY verify the ``statement_digest`` against statement material it obtains out of band, by recomputing the digest (:ref:`Section 4.3 <section-4.3>`).
