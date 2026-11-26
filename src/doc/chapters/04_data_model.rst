
.. _chapter-apx4:

.. _section-4:

Data Model and Message Formats
=================================

.. _section-4.1:

Message taxonomy
--------------------

.. list-table::
    :header-rows: 1
    :widths: 14 16 30 40

    * - Message
      - Name
      - Media type
      - Carries
    * - register request
      - **Envelope**
      - ``application/vnd.dev.sigstore.bundle.v0.3+json``
      - The Registrar's claims for a registration, DSSE-wrapped in an in-toto Statement (:ref:`Section 4.2.3 <section-4.2.3>`): artifact, statements (:ref:`Sections 4.2.3 <section-4.2.3>`, :ref:`4.2.4 <section-4.2.4>`)
    * - register response
      - **Envelope Receipt**
      - ``application/json``
      - The Provenance ID, or an error
    * - challenge request
      - **Challenge Request**
      - ``application/json``
      - The ``provenance_id`` subject
    * - challenge response
      - **Challenge**
      - ``application/json``
      - Single-use ``eat_nonce`` + ``challenge_id``, bound to the ``provenance_id``
    * - evidence request
      - **Evidence**
      - ``application/jose``
      - Workload-signed possession proof bound to the Challenge, as a JWT-form EAT (:ref:`Section 4.5 <section-4.5>`)
    * - evidence response
      - **Verdict**
      - ``application/json``
      - The Attestation Result produced by the Clerk's appraisal, carrying the issued credential

Error responses use the body ``{"error": "<reason>"}`` with an error status code. :ref:`Section 6 <section-6>` defines the complete error contract.

.. _section-4.2:

Envelope
------------

The register message (the **Envelope** of the :ref:`Section 4.1 <section-4.1>` taxonomy) is a **Sigstore bundle v0.3**. Its DSSE envelope carries an in-toto Statement (:ref:`Section 4.2.3 <section-4.2.3>`) whose ``predicate`` holds the Registrar's claims. :ref:`Appendix A <appendix-a>` shows a reference producer.

.. _section-4.2.1:

Media type
""""""""""""""""

Producers MUST emit the Envelope media type (:ref:`Section 4.1 <section-4.1>`). Parsers that accept the older alias ``application/vnd.dev.sigstore.bundle+json;version=0.3`` MUST treat it as the same version.

.. _section-4.2.2:

Verification material
"""""""""""""""""""""""""""

The Registrar signs the bundle with an ECDSA P-256 key in cosign-compatible form. The Clerk verifies the DSSE signature against a verification key **pre-shared out of band** (cosign-compatible PEM, SubjectPublicKeyInfo form). The key is never discovered from the message. A ``publicKey.hint`` in ``verificationMaterial`` MAY be present; it is metadata, and the Clerk MUST NOT treat it as a trust decision.

.. _section-4.2.3:

Payload (the DSSE-wrapped in-toto Statement)
""""""""""""""""""""""""""""""""""""""""""""""""""

The bundle MUST carry a ``dsseEnvelope`` with:

.. list-table::
    :header-rows: 1
    :widths: 15 12 73

    * - Member
      - Requirement
      - Contents
    * - ``payloadType``
      - REQUIRED
      - ``application/vnd.in-toto+json``. The Clerk MUST reject other values.
    * - ``payload``
      - REQUIRED
      - Standard base64 of an in-toto Statement JSON object (RFC 4648 base64, DSSE v1.0).
    * - ``signatures``
      - REQUIRED
      - Exactly one signature: an ECDSA P-256 signature (SHA-256 digest, ASN.1 DER-encoded) over the DSSE PAE (DSSE v1.0 pre-authentication encoding) of ``payloadType`` and ``payload``. The Clerk MUST verify it against the verification key (:ref:`Section 4.2.2 <section-4.2.2>`).

The DSSE payload decodes to an in-toto Statement JSON object with these members. Two statement layers exist: this register statement uses the legacy v0.1 ``_type`` that stock cosign emits, while the inner statement classes under ``predicate.statement`` are in-toto Statement v1, carried verbatim from their own producers (:ref:`Section 4.2.4 <section-4.2.4>`).

.. list-table::
    :header-rows: 1
    :widths: 15 12 73

    * - Member
      - Requirement
      - Contents
    * - ``_type``
      - REQUIRED
      - One of the values in the register statement registry (:ref:`Section 9.3.3 <section-9.3.3>`). The Clerk MUST reject any other value.
    * - ``predicateType``
      - REQUIRED
      - The register predicate type identifier ``https://apx.dev/register/v0.1``, defined for this document (the ``--type`` value of the reference producer, Appendix A). The Clerk MUST reject a missing or different value. It is distinct from the ``statement`` class keys (:ref:`Section 4.2.4 <section-4.2.4>`), which live under ``predicate``.
    * - ``subject``
      - REQUIRED
      - Non-empty array. Its first element's ``digest.sha256`` MUST equal the predicate's ``artifact.digest``. The Clerk MUST enforce the cross-check.
    * - ``predicate``
      - REQUIRED
      - A JSON object with the members below.

The ``predicate`` members (the Registrar's claims):

.. list-table::
    :header-rows: 1
    :widths: 12 12 76

    * - Member
      - Requirement
      - Contents
    * - ``iss``
      - REQUIRED
      - A Registrar identifier string. It is opaque to the Workload and MUST NOT be treated as a key-discovery handle. Example: ``ci/<project>``.
    * - ``iat``
      - REQUIRED
      - Issuance time, NumericDate (RFC 7519). It orders replacements of the same Provenance ID (:ref:`Section 5.1.6 <section-5.1.6>`).
    * - ``exp``
      - REQUIRED
      - Expiry time, NumericDate. The Clerk MUST reject an expired Envelope at registration. The window SHOULD be short (minutes, not days).
    * - ``jwk``
      - REQUIRED
      - The public key (RFC 7517) of the proving key (:ref:`Section 3.2 <section-3.2>`); the Workload proves possession of the private part. The JWK MAY carry a ``kid`` (RFC 7517 Section 4.5), which uniquely identifies the key within the deployment. This key is distinct from the register signing key (:ref:`Section 4.2.2 <section-4.2.2>`).
    * - ``artifact``
      - REQUIRED
      - A JSON object: ``{"name": "<image name>", "digest": "sha256:<64 hex>", "expected_measurement": "sha256:<64 hex>"}``. The Registrar MUST carry ``expected_measurement`` (:ref:`Section 5.2.4 <section-5.2.4>`).
    * - ``custody``
      - REQUIRED
      - An object: ``{"model": "<custody model>", "evidence": {...}}``. ``model`` is ``software-sealed`` or ``hardware-bound`` (:ref:`Section 3.2 <section-3.2>`). ``evidence`` is the custody evidence for that model (:ref:`Section 5.1.3 <section-5.1.3>`): ``kms-wrapped`` for ``software-sealed``, or ``tee-quote`` for ``hardware-bound``.
    * - ``statement``
      - REQUIRED
      - A JSON object of statement classes (:ref:`Section 4.2.4 <section-4.2.4>`).

The signature covers the entire DSSE payload.

.. _section-4.2.4:

The statement member
""""""""""""""""""""""""""

Each member of ``statement`` is a key-value pair. The key names the statement class. The value is a JSON object that conforms to in-toto Statement v1 (``_type`` value ``https://in-toto.io/Statement/v1``). The class values are third-party producer output (build-tool SBOM attestations, SLSA provenance statements, BOM documents from third-party BOM generators) emitted as in-toto Statement v1 and carried verbatim.

The statement class keys and their predicate types are registered in :ref:`Section 9.3.2 <section-9.3.2>` and are not part of the discovery document.

Rules:

* ``statement`` MUST contain at least one member.
* Every statement value MUST conform to in-toto Statement v1: a ``_type`` member equal to ``https://in-toto.io/Statement/v1`` and a ``predicateType`` member.
* ``statement.sbom``, if present, MUST carry an SPDX 2.3 JSON document as its predicate. The Clerk MUST reject a document that does not validate against SPDX 2.3.
* The Clerk MAY delegate full subject validation to the credential consumer (out of scope for APX).

.. _section-4.3:

Content hash and statement digests
--------------------------------------

Both digests are SHA-256 over the JSON Canonicalization Scheme (RFC 8785) serialization of the relevant object.

* **Content hash**: the digest of ``predicate`` (:ref:`Section 4.2.3 <section-4.2.3>`), computed by the Clerk after DSSE signature verification.
* **Statement digest**: the digest of the ``predicate.statement`` member (:ref:`Section 4.2.4 <section-4.2.4>`), carried as ``sha256:<64 lowercase hex>``. It covers every member of ``statement``, so an absent class changes the digest. The Clerk binds it into the registration record (:ref:`Section 5.1.6 <section-5.1.6>`) and the issued credential (:ref:`Section 4.6.2 <section-4.6.2>`).

.. _section-4.4:

Challenge Request and Challenge
-----------------------------------

The **Challenge Request** has this body:

.. code-block:: json

    {"provenance_id": "<provenance_id>"}

* ``provenance_id``: REQUIRED. The Provenance ID received in the Envelope Receipt (:ref:`Section 5.1.5 <section-5.1.5>`).

.. _section-4.5:

Evidence
------------

The **Evidence** (the evidence request message of :ref:`Section 4.1 <section-4.1>`) is a compact JWS (RFC 7515). The Workload signs it with the private proving key registered for it. The Evidence keeps its own JOSE header (``alg: EdDSA`` (RFC 8037), ``typ: apx+jwt``; the Clerk MUST reject other values). The ``typ`` value and the ``eat_profile`` value are defined by this document (:ref:`Section 11.2 <section-11.2>`). Only the Evidence (:ref:`Section 4.5 <section-4.5>`) and the issued credential (:ref:`Section 4.6.2 <section-4.6.2>`) use JOSE. The JWT claims are:

.. list-table::
    :header-rows: 1
    :widths: 15 12 73

    * - Claim
      - Requirement
      - Contents
    * - ``challenge_id``
      - REQUIRED
      - The value returned by the challenge.
    * - ``eat_nonce``
      - REQUIRED
      - The value returned by the challenge. The name follows RFC 9711 Section 4.1, which requires ``eat_nonce`` in JSON-encoded EATs and forbids the JWT ``nonce`` claim.
    * - ``provenance_id``
      - REQUIRED
      - The Provenance ID of this registration. It MUST equal the ``provenance_id`` used in the Challenge Request.
    * - ``eat_profile``
      - REQUIRED
      - An EAT profile URI (RFC 9711 Section 4.3.2, ``eat_profile`` claim) that anchors this Evidence as a JWT-form EAT profile. It MUST equal the EAT profile identifier ``https://apx.dev/profiles/attest-v0.1``, defined for this document. The Clerk MUST reject other values.
    * - ``iat``
      - REQUIRED
      - NumericDate.
    * - ``exp``
      - REQUIRED
      - NumericDate. The window MUST be at most the evidence window (:ref:`Section 9.2 <section-9.2>`); a longer window is rejected.
    * - ``measurement``
      - REQUIRED
      - A runtime measurement claim (:ref:`Section 5.2.4 <section-5.2.4>`): ``{"alg": "sha256", "value": "<64 lowercase hex>"}``. The Clerk MUST reject a missing or malformed measurement.

.. _section-4.6:

Verdict and issued credential
---------------------------------

.. _section-4.6.1:

Verdict
"""""""""""""

The **Verdict** has two REQUIRED members: the ``provenance_id`` and the issued credential. It is the Attestation Result; a Relying Party consumes its ``credential`` member.

.. list-table::
    :header-rows: 1
    :widths: 20 15 65

    * - Member
      - Requirement
      - Contents
    * - ``provenance_id``
      - REQUIRED
      - The Provenance ID of the registration.
    * - ``credential``
      - REQUIRED
      - A compact SD-JWT, :ref:`Section 4.6.2 <section-4.6.2>`.

Example:

.. code-block:: json

    {
      "provenance_id": "<provenance_id>",
      "credential": "<compact SD-JWT>"
    }

.. _section-4.6.2:

SD-JWT structure
""""""""""""""""""""""

The credential is an SD-JWT Verifiable Digital Credential (aligned with draft-ietf-oauth-sd-jwt-vc; see :ref:`Section 11.3 <section-11.3>`). Its rules:

* The credential ``iss`` MUST be the Clerk's HTTPS base URL (:ref:`Section 3.3 <section-3.3>`).
* The Clerk MUST publish its signing keys (``jwks`` or ``jwks_uri``) in JWT VC Issuer Metadata at ``/.well-known/jwt-vc-issuer`` on the origin of ``iss``. Like the discovery document, this endpoint is unauthenticated (:ref:`Section 3.3 <section-3.3>`).
* ``sub`` MUST be the provenance_id of the registration.
* ``vct`` MUST be present. Its value is defined by the consumer profile (:ref:`Section 11.3 <section-11.3>`).
* The protected header MUST carry ``typ: dc+sd-jwt``.
* The Clerk MUST set a finite ``exp``.
* ``cnf`` MUST be present and MUST carry ``jwk``: the public key of the proving key (RFC 7800).
* The Clerk MAY add other representations of the proving key (for example a ``kid`` in ``cnf``); each MUST be derived from ``cnf.jwk``.
* ``status`` MUST be present, as a Token Status List reference (:ref:`Section 12.1 <section-12.1>`; :ref:`Section 5.2.6 <section-5.2.6>`). Each credential MUST get a unique ``idx``, and the Clerk MUST publish the status list at ``uri``.
* Other claims are out of scope and are interpreted under the consumer profile. Any identifier that the credential asserts for its subject MUST be consistent with ``sub``.
* ``statement_digest`` MUST be present and carries the statement digest (:ref:`Section 4.3 <section-4.3>`).
* The Clerk MUST sign the SD-JWT with its issuer key (the signing algorithm is the issuer's choice). Issuer key management is out of scope, except that a key which signed a still-valid credential MUST remain published.
* ``key_protection_level`` MUST match the custody model: ``software`` for ``software-sealed`` and ``tee`` for ``hardware-bound``. Consumers MAY apply their own policy per level.
