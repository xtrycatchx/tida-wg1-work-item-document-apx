
.. _chapter-apx12:

.. _section-12:

References
==============

.. _section-12.1:

Normative references
-------------------------

* RFC 2119, RFC 8174 (BCP 14) - requirement keywords
* RFC 4648 - The Base16, Base32, and Base64 Data Encodings
* RFC 7515 - JSON Web Signature (JWS)
* RFC 7517 - JSON Web Key (JWK)
* RFC 7519 - JSON Web Token (JWT)
* RFC 7800 - Proof-of-Possession Key Semantics for JWTs (``cnf``)
* RFC 8037 - CFRG Curve25519 signatures (EdDSA with Ed25519)
* RFC 8615 - Well-Known Uniform Resource Identifiers (``/.well-known/apx-clerk``, :ref:`Section 3.3 <section-3.3>`)
* RFC 8785 - JSON Canonicalization Scheme (JCS)
* Token Status List - IETF OAuth working group draft-21 (``draft-ietf-oauth-status-list-21``), status list format for the ``status`` claim (:ref:`Section 5.2.6 <section-5.2.6>`)
* RFC 9711 - The Entity Attestation Token (EAT; ``eat_profile``, ``eat_nonce``, JWT-form claims set)
* SD-JWT - OAuth SD-JWT (Selective Disclosure for JWTs)
* DSSE v1.0 - Dead Simple Signing Envelope (PAE, envelope form) - ``https://github.com/secure-systems-lab/dsse``
* Sigstore Bundle v0.3 - ``https://github.com/sigstore/protobuf-specs``
* in-toto Statement v0.1 - ``https://in-toto.io/Statement/v0.1`` (register payload wrapper, :ref:`Section 9.3.3 <section-9.3.3>`)
* in-toto Statement v1 - ``https://in-toto.io/Statement/v1`` (inner statement classes, :ref:`Section 4.2.4 <section-4.2.4>`; register payload wrapper, :ref:`Section 9.3.3 <section-9.3.3>`)
* SLSA Provenance v1 - ``https://slsa.dev/provenance/v1``
* SPDX 2.3 - ``https://spdx.dev/Document``
* CycloneDX 1.6 - ``https://cyclonedx.org/`` (BOM statement classes ``cbom``/``obom``/``mlbom``, :ref:`Section 4.2.4 <section-4.2.4>`)
* OCI Image Format Specification - ``https://github.com/opencontainers/image-spec/blob/main/image-index.md`` (``manifest`` statement class, :ref:`Section 9.3.2 <section-9.3.2>`)

.. _section-12.2:

Informative references
---------------------------

* RFC 9334 - Remote ATtestation procedureS (RATS) Architecture (:ref:`Section 11.1 <section-11.1>`)
* SD-JWT VC - Verifiable Digital Credentials based on SD-JWT, draft-ietf-oauth-sd-jwt-vc-26 (:ref:`Section 11.3 <section-11.3>`)
* CRATB-Auth v0.1 (Draft, experimental) - A2A extension profile for holder binding and presentation of the SD-JWT VC credential issued under :ref:`Section 4.6.2 <section-4.6.2>`: ``https://github.com/a2aproject/experimental-ext-cratb-auth/blob/main/spec/v0.1/extension.md`` (:ref:`Section 7.4 <section-7.4>`)
* cosign - Sigstore signing tool; ``cosign attest-blob`` is the reference producer (:ref:`Appendix A <appendix-a>`)
