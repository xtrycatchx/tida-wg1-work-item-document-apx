
.. _chapter-apx3:

.. _section-3:

Architecture
===============

.. _section-3.1:

Architecture overview
---------------------

APX runs in two stages. Registration happens at build time. Proof of possession happens at runtime and repeats each time the Workload starts (:ref:`Section 5.2.5 <section-5.2.5>`). The message flow is shown in :numref:`Figure %s <figure-3.1>`, and the steps below follow the numbering in the figure.

.. .. uml::

..     hide footbox
..     participant "Registrar (CI/CD)" as R
..     participant "Clerk" as C
..     participant "Workload" as W

..     note over R, C: out of scope: pre-established trust

..     rect rgb(70, 110, 220, 0.08)
..         note over R, W: build time
..         R -> C: 1. POST /v1/apx-clerk/register\n(APX Envelope, DSSE-signed Sigstore bundle) ·\napplication/vnd.dev.sigstore.bundle.v0.3+json
..         C --> R: 2. Envelope Receipt (provenance_id), or error · application/json
..         opt the handover mechanism is implementation-specific
..             R -> W: 3. registrar hands Provenance ID · kid · key material (optional, Section 3.2)
..         end
..     end

..     rect rgb(40, 160, 90, 0.08)
..         note over C, W: runtime
..         W -> C: 4. POST /v1/apx-clerk/challenge\n{provenance_id} · application/json
..         C --> W: 5. challenge_id, eat_nonce · application/json
..         W -> C: 6. POST /v1/apx-clerk/evidence\n(workload-signed JWT) · application/jose
..         C --> W: 7. Verdict (credential) · application/json
..     end

.. _figure-3.1:

.. code-block:: text
   :caption: Architecture overview: message flow between the Registrar, the Clerk and the Workload

         +-------------+                               +-------------+
         |             |         BUILD TIME            |             |
         |  Registrar  |----[1] Envelope ------------->|             |
         |   (CI/CD)   |<..... Provenance ID [2] ......|             |
         |             |                               |             |
         +-------------+                               |             |
                |                                      |             |
                |                                      |             |
       [3] Provenance ID                               |             |
                |                                      |    Clerk    |
                v                                      |             |
         +-------------+                               |             |
         |             |           RUNTIME             |             |
         |             |----[4] Challenge Request ---->|             |
         |   Agentic   |<......... eat_nonce [5] ......|             |
         |   Workload  |                               |             |
         |             |----[6] Evidence ------------->|             |
         |             |<............ Verdict [7] .....|             |
         |             |                               |             |
         +-------------+                               +-------------+ 

**Registration (build time)**

1. The Registrar submits an Envelope to the Clerk (:ref:`Section 5.1.1 <section-5.1.1>`).
2. The Clerk validates the Envelope (:ref:`Section 5.1.2 <section-5.1.2>`) and returns a Provenance ID.
3. The Registrar hands the Provenance ID to the Workload, out of band (:ref:`Section 3.2 <section-3.2>`).

**Proof of possession (runtime)**

4. The Workload requests a Challenge for its Provenance ID.
5. The Clerk returns a single-use Challenge with a nonce.
6. The Workload submits Evidence signed with its proving key.
7. The Clerk verifies the Evidence (:ref:`Section 5.2.3 <section-5.2.3>`) and returns a Verdict with a credential bound to the proving key.

.. _section-3.2:

Key custody
---------------

The Registrar names a custody model in the ``custody`` member of the Envelope (:ref:`Section 4.2.3 <section-4.2.3>`). Two models exist.

* ``software-sealed``: The artifact holds only an encrypted key, which the Workload unwraps at startup with a KMS decrypt call under its runtime identity. A copy outside that identity cannot unwrap the key. Binding the key to the artifact identifies the artifact but does not prevent use.
* ``hardware-bound``: The private proving key is generated inside, and remains inside, a trusted execution environment (TEE) that does not permit export of the key. A copy that runs a different image is rejected.

For ``hardware-bound``:

* The Workload MUST request each Evidence signature from the TEE (:ref:`Section 4.5 <section-4.5>`). The TEE MUST attest its launch measurement, which includes the artifact digest, and the key MUST be bound to that measurement.
* APX does not otherwise verify the TEE hardware. The Clerk verifies the TEE quote against the vendor root (:ref:`Section 5.1.3 <section-5.1.3>`).

* The Provenance ID (``provenance_id``) is mandatory - it is the APX subject (challenge/evidence binding, credential ``sub``).
* The Envelope's ``jwk`` (:ref:`Section 4.2.3 <section-4.2.3>`) is the wire trust anchor. The out-of-band handover, which is implementation-specific, MAY carry a ``kid`` - a key-selection handle the Workload resolves against its own key store - which correlates with the ``jwk``'s key (RFC 7517 Section 4.5) but is not, and MUST NOT be treated as, a trust decision: no party on the wire authenticates it.

.. _section-3.3:

Clerk discovery
-------------------

The Clerk MUST serve a discovery document at ``GET /.well-known/apx-clerk`` (RFC 8615 well-known URI):

.. code-block::

    GET /.well-known/apx-clerk
    Content-Type: application/json

Example (using the :ref:`Section 9.3 <section-9.3>` registries):

.. code-block:: json

    {
      "spec_version": "0.1",
      "endpoints": {
        "register": "/v1/apx-clerk/register",
        "challenge": "/v1/apx-clerk/challenge",
        "evidence": "/v1/apx-clerk/evidence"
      },
      "accepts": {
        "envelope_formats": [
          {
            "type": "sigstore-bundle",
            "version": "0.3",
            "media_type": "application/vnd.dev.sigstore.bundle.v0.3+json",
            "dsse_payload_type": "application/vnd.in-toto+json",
            "signature": {
              "algorithm": "ECDSA_P256_SHA256",
              "encoding": "ASN.1 DER",
              "signed_input": "DSSE-PAE(payload_type, payload)"
            }
          }
        ]
      }
    }

Rules:

* **Paths**: The ``endpoints`` values are paths, resolved against the well-known URL's origin. This document names the endpoints ``register``, ``challenge`` and ``evidence``. The Clerk MUST NOT nominate hosts or schemes, and clients MUST NOT trust a discovered host. The Clerk's base URL is pre-shared.
* **accepts.envelope_formats**: The register Envelope formats the deployment accepts, keyed by ``type`` (:ref:`Section 9.3.1 <section-9.3.1>`). Clients use it to pre-validate. The entries are fixed protocol constants (:ref:`Section 4.2 <section-4.2>`).
* **Unknown members**: Clients MUST ignore unknown members outside ``endpoints`` and ``accepts``. Discovery never authorizes anything.
* **Access**: The Clerk MUST serve the discovery document without client authentication.
