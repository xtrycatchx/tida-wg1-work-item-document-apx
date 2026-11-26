
.. _chapter-apx2:

.. _section-2:

Conventions and Terminology
==============================

.. _section-2.1:

Requirements language
-------------------------

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 (RFC 2119, RFC 8174) when, and only when, they appear in all capitals.

.. _section-2.2:

Roles
---------

* **Registrar**: The party that registers an Envelope. Typically a CI/CD pipeline.
* **Clerk**: The verifier service (RFC 9334 Section 4.1). The Clerk validates Envelopes, holds registrations in a registration store (volatile or durable - :ref:`Section 5.1.6 <section-5.1.6>`), issues Challenges, verifies Evidence, and produces the Attestation Result (the Verdict, :ref:`Section 4.6.1 <section-4.6.1>`).
* **Workload**: The workload that requests a credential by proving possession of the proving key registered at registration time.

.. _section-2.3:

Messages and artifacts
--------------------------

* **Envelope**: A Sigstore bundle (:ref:`Section 4.2 <section-4.2>`) signed by the Registrar that carries provenance statements, DSSE-wrapped in an in-toto Statement (:ref:`Section 4.2.3 <section-4.2.3>`).
* **Envelope Receipt**, **Challenge Request**, **Challenge**, **Evidence**, **Verdict**: The message names defined in :ref:`Section 4.1 <section-4.1>`.
* **Register statement**: The in-toto Statement that the DSSE payload of an Envelope carries (:ref:`Section 4.2.3 <section-4.2.3>`).
* **Statement class value**: The in-toto Statement v1 object held under a key of the ``statement`` member (:ref:`Section 4.2.4 <section-4.2.4>`).

.. _section-2.4:

Other defined terms
-----------------------

* **Provenance ID**: The opaque value that names a registration (:ref:`Section 5.1.4 <section-5.1.4>`). It appears as ``provenance_id`` in credentials and responses.
* **Registration record**: The Clerk's stored record of an accepted Envelope: the Envelope without statement contents, its statement digest, the Provenance ID and the proving key (:ref:`Section 5.1.6 <section-5.1.6>`). It proves past acceptance.
* **Registration store**: The Clerk-side storage that holds registration records (:ref:`Section 5.1.6 <section-5.1.6>`).
* **Registry**: A value registry defined in :ref:`Section 9.3 <section-9.3>`.
* **Proving key**: The public key carried in the Envelope's ``predicate.jwk`` (:ref:`Section 4.2.3 <section-4.2.3>`) and in the credential's ``cnf`` (:ref:`Section 4.6.2 <section-4.6.2>`).
* **Registrar verification key**: The pre-shared ECDSA P-256 public key against which the Clerk verifies the Envelope signature (:ref:`Section 4.2.2 <section-4.2.2>`).
* **Challenge TTL**: The lifetime of a Challenge (:ref:`Sections 5.2.2 <section-5.2.2>`, :ref:`9.2 <section-9.2>`).
* **Evidence window**: The maximum period between the ``iat`` and ``exp`` of Evidence (:ref:`Sections 4.5 <section-4.5>`, :ref:`9.2 <section-9.2>`).
* **Handover**: The out-of-band step in which the Registrar delivers key material, a ``kid``, or the Provenance ID to the Workload (:ref:`Section 3.2 <section-3.2>`).
* **Custody model**: The model that states where the proving key lives (:ref:`Section 3.2 <section-3.2>`).
* **Custody evidence**: The material that supports a custody model, verified by the Clerk (:ref:`Section 5.1.3 <section-5.1.3>`).
