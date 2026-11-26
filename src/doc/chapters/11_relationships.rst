

.. _chapter-apx11:

.. _section-11:

Relationship to Other Specifications
=========================================

.. _section-11.1:

Relationship to RATS
-------------------------

INFORMATIVE. This section states how APX maps onto the IETF Remote Attestation procedures architecture (RFC 9334) and the EAT (RFC 9711), and which deviations are deliberate.

.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - RATS concept (RFC 9334)
      - APX counterpart
    * - Attester
      - The Workload (:ref:`Section 2.2 <section-2.2>`)
    * - Attesting Environment
      - The Workload runtime
    * - Verifier
      - The Clerk (:ref:`Section 2.2 <section-2.2>`): the ``register``, ``challenge`` and ``evidence`` endpoints (:ref:`Section 3.3 <section-3.3>`)
    * - Relying Party
      - Not exercised (:ref:`Section 1.3 <section-1.3>`); the role's duties sit with downstream credential consumers (:ref:`Section 7.4 <section-7.4>`)
    * - Evidence
      - The Evidence (:ref:`Section 4.5 <section-4.5>`)
    * - Attestation Results
      - The Verdict (:ref:`Section 4.6.1 <section-4.6.1>`)
    * - Reference Values
      - The Envelope's ``artifact.expected_measurement`` (:ref:`Section 5.2.4 <section-5.2.4>`)
    * - Endorsements / trust anchor
      - The pre-shared Registrar verification key (:ref:`Section 4.2.2 <section-4.2.2>`) and, for ``hardware-bound``, the TEE vendor root (:ref:`Section 5.1.3 <section-5.1.3>`)

Topology: APX implements the passport model (RFC 9334 Section 5.1) - the Workload caches the credential and presents it onward to consumers, who verify it under their own consumer profile, including holder binding (:ref:`Section 7.4 <section-7.4>`).

Declared deviations:

* **Registrar role**: RFC 9334 defines no role for build-time registration; the Registrar is a provisioning role with one pre-shared key and no endorsement chain (:ref:`Sections 4.2.2 <section-4.2.2>`, :ref:`7.1 <section-7.1>`).
* **Credential issuance on the result**: the Attestation Result is hybrid: an appraisal decision plus an issued SD-JWT credential that RATS does not model (:ref:`Section 4.6.2 <section-4.6.2>`).
* **Possession-plus-measurement appraisal**: the Clerk appraises the possession proof and the runtime measurement, not general state of the Target Environment (RFC 9334 Section 8.1; :ref:`Section 5.2.4 <section-5.2.4>`).

.. _section-11.2:

EAT alignment
------------------

INFORMATIVE. The Evidence is profiled as a JWT-form EAT (RFC 9711). JWT is a peer encoding of the EAT claims set; no CBOR is required. The mapping:

.. list-table::
    :header-rows: 1
    :widths: 25 25 50

    * - APX Evidence claim
      - EAT claim (RFC 9711)
      - Notes
    * - ``eat_profile``
      - ``eat_profile``
      - Identifies the profile by URI (:ref:`Section 4.5 <section-4.5>`).
    * - ``eat_nonce``
      - ``eat_nonce``
      - The single-use challenge nonce carries the EAT freshness semantics (:ref:`Section 5.2.2 <section-5.2.2>`).
    * - ``iat``/``exp``
      - JWT time claims (RFC 7519)
      - Standard JWT time claims.
    * - ``challenge_id``, ``provenance_id``
      - profile extensions
      - APX-specific claims.

.. _section-11.3:

SD-JWT VC alignment
------------------------

Alignment with SD-JWT Verifiable Digital Credentials (draft-ietf-oauth-sd-jwt-vc-26, INFORMATIVE; the credential of :ref:`Section 4.6.2 <section-4.6.2>`):

* **Conformant**: the credential follows the draft's rules, which :ref:`Section 4.6.2 <section-4.6.2>` states.
* **Agent identity profiles**: agent identity claim vocabularies are defined by the consumer profile (CRATB-Auth, :ref:`Section 12.2 <section-12.2>`), not by APX.
