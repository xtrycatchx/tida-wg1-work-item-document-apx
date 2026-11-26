
.. _chapter-apx15:

.. _appendix-b:

Appendix B. Example flow
========================

The examples walk one continuous flow: a Registrar's Envelope, the Clerk's Envelope Receipt, a Challenge, the Workload's Evidence, and the Verdict with the issued credential. Every response shown is success; on any failure the Clerk returns the :ref:`Section 4.1 <section-4.1>`/:ref:`Section 6 <section-6>` error contract and no credential.

.. _section-B.1:

B.1 Envelope (the Sigstore bundle; the DSSE payload shown decoded)
------------------------------------------------------------------

.. code-block:: json

    {
      "mediaType": "application/vnd.dev.sigstore.bundle.v0.3+json",
      "verificationMaterial": { "publicKey": { "hint": "..." } },
      "dsseEnvelope": {
        "payloadType": "application/vnd.in-toto+json",
        "payload": "<base64 of the in-toto Statement below>",
        "signatures": [ { "sig": "<base64 ASN.1 DER ECDSA over the DSSE PAE>", "keyid": "" } ]
      }
    }

The decoded DSSE payload (the in-toto Statement - ``iat``/``exp``/``jwk`` are the Registrar's ``predicate`` claims, not Envelope-level members; the top-level ``predicateType`` is the pinned register predicate, :ref:`Section 4.2.3 <section-4.2.3>`):

.. code-block:: json

    {
      "_type": "https://in-toto.io/Statement/v0.1",
      "predicateType": "https://apx.dev/register/v0.1",
      "subject": [ { "name": "simple-workload", "digest": { "sha256": "..." } } ],
      "predicate": {
        "iss": "ci/<project>",
        "iat": 1762140000,
        "exp": 1762140600,
        "jwk": {
          "kty": "OKP",
          "crv": "Ed25519",
          "x": "11qYAYKxCrfVS_7TyWQHOg7hcvPapiMlrwIaaPcHURo"
        },
        "custody": {
          "model": "software-sealed",
          "evidence": {
            "type": "kms-wrapped",
            "wrapping_key": "<opaque reference to the KMS key version>",
            "wrapped_key": "<base64 encrypted private key>",
            "public_key": { "kty": "OKP", "crv": "Ed25519", "x": "11qYAYKxCrfVS_7TyWQHOg7hcvPapiMlrwIaaPcHURo" }
          }
        },
        "artifact": {
          "name": "simple-workload",
          "digest": "sha256:...",
          "expected_measurement": "sha256:..."
        },
        "statement": {
          "provenance": {
            "_type": "https://in-toto.io/Statement/v1",
            "predicateType": "https://slsa.dev/provenance/v1",
            "subject": [ { "name": "simple-workload", "digest": { "sha256": "..." } } ],
            "predicate": { "...": "..." }
          },
          "sbom": {
            "_type": "https://in-toto.io/Statement/v1",
            "predicateType": "https://spdx.dev/Document",
            "subject": [ { "name": "simple-workload", "digest": { "sha256": "..." } } ],
            "predicate": { "...": "..." }
          },
          "cbom": {
            "_type": "https://in-toto.io/Statement/v1",
            "predicateType": "https://cyclonedx.org/bom",
            "subject": [ { "name": "simple-workload", "digest": { "sha256": "..." } } ],
            "predicate": {
              "bomFormat": "CycloneDX",
              "specVersion": "1.6",
              "components": [
                {
                  "type": "cryptographic-asset",
                  "name": "openssl",
                  "properties": [ { "name": "cdx:crypto:protocol", "value": "tls-1.3" } ]
                }
              ]
            }
          },
          "obom": {
            "_type": "https://in-toto.io/Statement/v1",
            "predicateType": "https://cyclonedx.org/bom",
            "subject": [ { "name": "simple-workload", "digest": { "sha256": "..." } } ],
            "predicate": {
              "bomFormat": "CycloneDX",
              "specVersion": "1.6",
              "components": [
                { "type": "operating-system", "name": "alpine", "version": "3.21" }
              ]
            }
          },
          "mlbom": {
            "_type": "https://in-toto.io/Statement/v1",
            "predicateType": "https://cyclonedx.org/bom",
            "subject": [ { "name": "simple-workload", "digest": { "sha256": "..." } } ],
            "predicate": {
              "bomFormat": "CycloneDX",
              "specVersion": "1.6",
              "components": []
            }
          }
        }
      }
    }

Note: the Envelope carries no Provenance ID; the Clerk assigns one at registration time (:ref:`Section 5.1.4 <section-5.1.4>`). The BOM classes are optional-absent: a Registrar that did not generate them omits the keys.

.. _section-B.2:

B.2 Envelope Receipt (register response)
----------------------------------------

On success the Clerk returns the Provenance ID (:ref:`Section 5.1.5 <section-5.1.5>`):

.. code-block::

    200 OK  ·  application/json

    {
      "provenance_id": "<provenance_id>"
    }

.. _section-B.3:

B.3 Challenge (request and response)
-------------------------------------

The Workload requests a Challenge for its ``provenance_id`` (:ref:`Section 4.4 <section-4.4>`):

.. code-block::

    POST /v1/apx-clerk/challenge  ·  application/json

    {
      "provenance_id": "<provenance_id>"
    }

On success the Clerk returns the single-use Challenge (:ref:`Section 5.2.2 <section-5.2.2>`):

.. code-block::

    200 OK  ·  application/json

    {
      "challenge_id": "0f4a2b6c8d9e1f3a",
      "eat_nonce": "5eb63bbbe01eeed093cb22bb8f5acdc3e7f1a9b8c2d3e4f51677889aaabbccdd"
    }

``challenge_id`` is 8 random bytes and ``eat_nonce`` is 32 random bytes, both hex-encoded (16 and 64 lowercase hex characters).

.. _section-B.4:

B.4 Evidence (the Workload-signed JWT)
--------------------------------------

.. code-block::

    POST /v1/apx-clerk/evidence
    Content-Type: application/jose

    <compact JWS: <header>.<payload>.<signature>>

The JOSE header (the Clerk rejects other ``alg``/``typ`` values, :ref:`Section 4.5 <section-4.5>`):

.. code-block:: json

    { "alg": "EdDSA", "typ": "apx+jwt" }

The decoded claims. ``exp - iat`` MUST be at most the evidence window (:ref:`Section 9.2 <section-9.2>`):

.. code-block:: json

    {
      "challenge_id": "0f4a2b6c8d9e1f3a",
      "eat_nonce": "5eb63bbbe01eeed093cb22bb8f5acdc3e7f1a9b8c2d3e4f51677889aaabbccdd",
      "provenance_id": "<provenance_id>",
      "eat_profile": "https://apx.dev/profiles/attest-v0.1",
      "measurement": { "alg": "sha256", "value": "abababababababababababababababababababababababababababababababab" },
      "iat": 1762140600,
      "exp": 1762140625
    }

.. _section-B.5:

B.5 Verdict
-----------

.. code-block:: json

    {
      "provenance_id": "<provenance_id>",
      "credential": "eyJhbGciOiJFZERTQSIsInR5cCI6ImRjK3NkLWp3dCI...<SD-JWT>"
    }

The decoded credential payload (:ref:`Section 4.6.2 <section-4.6.2>`). ``cnf.jwk`` is the registered proving key. A Clerk MAY add further informational representations of that key (for example a ``kid`` member of ``cnf``):

.. code-block:: json

    {
      "iss": "https://<clerk-base-URL>",
      "sub": "<provenance_id>",
      "iat": 1762140600,
      "exp": 1762227000,
      "vct": "<vct>",
      "cnf": {
        "jwk": { "kty": "OKP", "crv": "Ed25519", "x": "11qYAYKxCrfVS_7TyWQHOg7hcvPapiMlrwIaaPcHURo" }
      },
      "status": { "status_list": { "idx": 0, "uri": "<status list URI>" } },
      "statement_digest": "sha256:<64 lowercase hex>",
      "key_protection_level": "software"
    }

