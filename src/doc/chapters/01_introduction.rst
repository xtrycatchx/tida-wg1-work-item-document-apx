
.. _chapter-apx1:

.. _section-1:

Introduction
===============

Software supply chains produce machine-readable claims about a build: an SLSA provenance statement, an SBOM. The in-toto attestation framework defines a Statement format for such claims: each Statement names the artifacts it covers (its subjects) and carries a typed predicate. Statements are commonly wrapped in a DSSE envelope and signed. The signature lets a verifier check who signed the envelope and that its payload has not been altered since.

The underlying standards do not connect these claims to the Workload that runs the artifact (:ref:`Section 1.1 <section-1.1>`). APX adds that connection in two steps. At build time, a Registrar registers a signed Envelope that carries provenance statements. At runtime, the Workload proves possession of the proving key registered for it, and the Clerk issues a credential stating that the Workload holds that key for that provenance.

This document builds on:

* in-toto Statements for the register payload and its inner statement classes (:ref:`Sections 4.2.3 <section-4.2.3>`, :ref:`4.2.4 <section-4.2.4>`)
* SLSA provenance and SPDX documents as predicate types
* SD-JWT for the issued credential
* EdDSA (Ed25519) for Workload signatures (Evidence)
* ECDSA P-256 for the register signature (cosign-compatible, :ref:`Section 4.2.2 <section-4.2.2>`)

APX follows the RATS architecture (RFC 9334), using the Passport model, and profiles its Evidence as a JWT-form EAT (RFC 9711). :ref:`Section 11 <section-11>` states the mapping and the deliberate deviations.

.. _section-1.1:

Problem
-----------

Build provenance statements are verifiable, but they carry no identity: a consumer can check *what* an artifact was built from, not *who* controls the key that the artifact uses. APX describes the exchange that binds build-time provenance to run-time key possession, so that a Clerk can issue an identity credential whose ``sub``/``cnf`` binding is anchored in verified provenance. This is not addressed by the underlying standards (in-toto, SLSA, SD-JWT) individually.

.. _section-1.2:

Objectives
--------------

#. Bind a build-time provenance material to a run-time proof of possession of the proving key registered for the Workload.
#. Keep the Registrar and the Clerk loosely coupled through a pre-shared verification key, without key discovery from the Envelope.
#. Describe a three-message flow (register at build time; challenge and evidence at runtime) with single-use challenges.
#. Keep all messages independent of statement size: statements are bound by digest, never transported outside the Envelope.
#. Remain extensible through explicit registries (:ref:`Section 9.3 <section-9.3>`) without changing the message flow.

.. _section-1.3:

Scope
---------

* **Covered**: Envelope format and validation, registration, custody verification (:ref:`Section 5.1.3 <section-5.1.3>`), Provenance ID assignment, Challenge issuance, Evidence verification, credential issuance (SD-JWT), Clerk discovery (:ref:`Section 3.3 <section-3.3>`), registries for envelope formats and statement classes.
* **Participants**: Registrar, Clerk, Workload. APX ends where the Clerk returns the Attestation Result (the Verdict, :ref:`Section 4.6.1 <section-4.6.1>`): the Workload carries it onward, and appraising it is the Relying Party's act (passport model, :ref:`Section 11.1 <section-11.1>`). APX defines the Attestation Result fields (:ref:`Section 4.6.2 <section-4.6.2>`).
* **Trust boundaries**: Registrar–Clerk trust is pre-shared and assumed (:ref:`Section 7.1 <section-7.1>`); Workloads are untrusted until they present a valid possession proof.
* **Dependencies**: the normative references listed in :ref:`Section 12.1 <section-12.1>`.

.. _section-1.4:

Non-goals
-------------

This document deliberately does not address:

* Key distribution, rotation, or storage of the pre-shared Registrar verification key (:ref:`Section 7.1 <section-7.1>`).
* Credential consumer validation logic beyond the recommendation in :ref:`Section 7.4 <section-7.4>`.
* Authentication mechanisms protecting the Clerk's ``register``, ``challenge``, and ``evidence`` endpoints (:ref:`Section 7.3 <section-7.3>`).
* Transport provisioning, service deployment, or registration store and challenge store implementation.
* Authorization policy between the Clerk and downstream services.
* Agent identity claim vocabularies, which consumer profiles define (:ref:`Section 11.3 <section-11.3>`).
* The transport of a revocation request: the out-of-band interface a Workload owner uses to ask the Clerk to revoke a registration (:ref:`Section 5.2.6 <section-5.2.6>`).
* Presentation of the credential and holder binding at presentation time (:ref:`Section 7.4 <section-7.4>`).
