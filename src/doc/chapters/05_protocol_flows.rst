
.. _chapter-apx5:

.. _section-5:

Interaction Flows
=================

.. _section-5.1:

Build time: registration
------------------------

.. _section-5.1.1:

Request
"""""""""""""

The Registrar submits an **Envelope**:

.. code-block::

    POST /v1/apx-clerk/register
    Content-Type: application/vnd.dev.sigstore.bundle.v0.3+json

    <Sigstore bundle v0.3 JSON>

.. _section-5.1.2:

Envelope validation rules
"""""""""""""""""""""""""""""""

The Clerk MUST apply these checks, in order. The first failed check stops processing. Each failure returns the status code for its condition in :ref:`Section 6 <section-6>`.

#. Parse the Sigstore bundle (:ref:`Section 4.2.1 <section-4.2.1>`) and its ``dsseEnvelope``.
#. Verify the DSSE signature over the PAE of ``payloadType``/``payload`` (:ref:`Section 4.2.3 <section-4.2.3>`). The verification key is defined in :ref:`Section 4.2.2 <section-4.2.2>`.
#. Reject a ``payloadType`` other than ``application/vnd.in-toto+json`` and decode the payload. The decoded statement MUST meet the register statement rules of :ref:`Section 4.2.3 <section-4.2.3>`.
#. Reject a ``predicate.exp`` in the past (:ref:`Section 4.2.3 <section-4.2.3>`).
#. Validate each present statement class against :ref:`Section 4.2.4 <section-4.2.4>`.
#. Reject a statement class key that is not in the registry (:ref:`Section 9.3.2 <section-9.3.2>`).
#. Reject a ``predicate.artifact`` (:ref:`Section 4.2.3 <section-4.2.3>`) with a missing name or a malformed ``expected_measurement`` (:ref:`Section 5.2.4 <section-5.2.4>`), or a ``predicate.statement.provenance`` subject digest that does not equal ``artifact.digest``.
#. Validate ``predicate.jwk`` as a well-formed Ed25519 public JWK.
#. Reject any predicate member not defined in :ref:`Section 4.2.3 <section-4.2.3>`.
#. Verify the custody evidence (:ref:`Section 5.1.3 <section-5.1.3>`). Then assign the Provenance ID (:ref:`Section 5.1.4 <section-5.1.4>`) and look up the stored record for it (:ref:`Section 5.1.6 <section-5.1.6>`).

.. _section-5.1.3:

Custody verification
""""""""""""""""""""

The Clerk MUST verify the custody evidence before it assigns a Provenance ID (:ref:`Section 5.1.4 <section-5.1.4>`).

* ``software-sealed``, evidence type ``kms-wrapped``: the Clerk MUST check that the wrapping key reference is well-formed and that ``public_key`` equals ``predicate.jwk``.
* ``hardware-bound``, evidence type ``tee-quote``: the Clerk MUST verify the quote signature chain to the TEE vendor root. It MUST check that ``public_key`` equals ``predicate.jwk``. It MUST check the launch measurement against ``artifact.expected_measurement`` (:ref:`Section 5.2.4 <section-5.2.4>`). It MUST check that the quote's report data equals the SHA-256 of the JCS-canonicalized ``predicate.jwk`` (:ref:`Section 4.3 <section-4.3>`).

The Registrar MUST NOT hold export or signing rights on a ``hardware-bound`` key.

.. _section-5.1.4:

Provenance ID assignment
""""""""""""""""""""""""""

The Clerk assigns a Provenance ID to the registration. Its generation and concrete shape are implementation-defined. Registrations with the same artifact digest and proving key MUST receive the same Provenance ID; any other registration MUST receive a different one. The Provenance ID MUST NOT depend on the statements, ``iat``, or ``exp``.

.. _section-5.1.5:

Response
""""""""""""""

On success the Clerk returns an **Envelope Receipt**:

.. code-block::

    200 OK
    {
      "provenance_id": "<opaque Provenance ID assigned by the clerk>"
    }

On failure, the Clerk returns the :ref:`Section 6 <section-6>` error contract.

.. _section-5.1.6:

Registration store
""""""""""""""""""

The record is not re-verifiable evidence. A party that needs to verify the register signature MUST obtain the Envelope from the Registrar.

The Clerk MUST NOT register the same Envelope twice: an identical content hash MUST be rejected, never a refresh. A re-registration that is assigned the same Provenance ID replaces the stored record only when its ``iat`` is strictly later than the stored ``iat``. An equal or earlier ``iat`` MUST be rejected.

.. _section-5.2:

Runtime: proof of possession
----------------------------

The Workload submits a **Challenge Request** (:ref:`Section 4.4 <section-4.4>`):

.. code-block::

    POST /v1/apx-clerk/challenge
    Content-Type: application/json

The Clerk resolves the ``provenance_id`` (:ref:`Section 5.2.1 <section-5.2.1>`) and returns a **Challenge** (:ref:`Section 5.2.2 <section-5.2.2>`).

.. _section-5.2.1:

Resolution rules
""""""""""""""""""""""

The Clerk MUST resolve the ``provenance_id`` to a registration record:

* Look up the registration by exact Provenance ID match; an unknown ID is rejected.

.. _section-5.2.2:

Nonce rules
"""""""""""""""""

The Clerk generates:

* ``eat_nonce``: 32 random bytes from a CSPRNG, hex-encoded.
* ``challenge_id``: 8 random bytes from a CSPRNG, hex-encoded.

The Clerk MUST bind each Challenge to the ``provenance_id`` of its Challenge Request. An unconsumed Challenge expires after the challenge TTL (:ref:`Section 9.2 <section-9.2>`).

.. _section-5.2.3:

Evidence verification
"""""""""""""""""""""

The Workload submits **Evidence** (:ref:`Section 4.5 <section-4.5>`):

.. code-block::

    POST /v1/apx-clerk/evidence
    Content-Type: application/jose

    <compact JWS>

The Clerk MUST apply these checks in order. The first failed check stops processing. Each failure returns the status code for its condition in :ref:`Section 6 <section-6>`.

#. Validate the Evidence claims (:ref:`Section 4.5 <section-4.5>`).
#. Resolve the challenge. Not found or already consumed.
#. Check the challenge TTL. Expired.
#. Compare ``challenge_id`` and ``eat_nonce`` to the stored values, using constant-time comparison.
#. Resolve the registration from the ``provenance_id`` claim, which MUST equal the one bound to the challenge.
#. Verify the JWS signature against the registration's ``jwk``.
#. Appraise the runtime measurement (:ref:`Section 5.2.4 <section-5.2.4>`).
#. Issue the credential (:ref:`Section 4.6.2 <section-4.6.2>`).
#. Consume the challenge after all checks and issuance succeed. Consumption MUST be atomic: concurrent Evidence messages carrying the same challenge yield at most one successful Verdict, and every other attempt is rejected. A rejected or failed Evidence attempt leaves the challenge reusable until TTL so the Workload can retry.

On success: ``200`` with the **Verdict**. On any failure, the Workload MUST NOT receive credentials, and the Clerk SHOULD log the rejection reason.

.. _section-5.2.4:

Runtime measurement appraisal (software-rooted)
"""""""""""""""""""""""""""""""""""""""""""""""

APX provides the runtime evidence that RATS expects through a **software-rooted** self-measurement (:ref:`Section 11.1 <section-11.1>`).

The Envelope's ``artifact.expected_measurement`` is the reference value for the runtime measurement, not the container image digest. How it is derived is implementation-defined. Both values are compared as lowercase hex, without the ``sha256:`` prefix.

The Evidence's ``measurement`` claim is the SHA-256 digest of the Workload's executing code. The Workload MUST fail closed when it cannot measure that code.

For ``software-sealed``, the Clerk compares the two on every Evidence message. This is a consistency check that catches a mis-built or mis-deployed Workload. It does not resist a hostile Workload, which can report the expected value. The measurement is self-reported, so deployments MUST NOT represent it as hardware attestation.

For ``hardware-bound``, the Clerk instead compares the launch measurement from the TEE quote (:ref:`Section 5.1.3 <section-5.1.3>`) with ``artifact.expected_measurement``, and does not use the self-reported claim.

In both cases, a missing or malformed ``measurement``, or unequal values, is rejected.

.. _section-5.2.5:

Credential re-issuance
""""""""""""""""""""""

One handshake yields one Verdict, and one Verdict covers exactly one Evidence. A Workload that restarts repeats the challenge/evidence flow and receives a fresh Verdict with a freshly issued credential. Re-issuance without a new challenge MUST NOT happen.

.. _section-5.2.6:

Credential status and revocation
""""""""""""""""""""""""""""""""

* The Clerk MUST assign each issued credential a unique ``idx`` in its status list, and MUST record that ``idx`` against the registration that produced the credential.
* The Clerk MUST publish list updates within a maximum publication interval set by the deployment. The interval MUST be stated in the status list. It bounds how long a revocation takes to reach consumers.
* Revoking a registration revokes every credential issued from it. The Clerk MUST set the bit for each ``idx`` recorded against that registration.
