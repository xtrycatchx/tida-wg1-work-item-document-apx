
.. _chapter-apx14:

.. _appendix-a:

Appendix A. Reference producer (non-normative)
==============================================

``cosign attest-blob`` is a known producer of the Envelope: it signs the pushed artifact's manifest bytes with the :ref:`Section 4.2.3 <section-4.2.3>` members as ``--predicate`` and writes the Sigstore bundle with ``--bundle``. Because the attested blob is the manifest, the DSSE ``subject`` digest is the artifact digest itself (:ref:`Section 4.2.3 <section-4.2.3>` cross-check), and cosign emits the v0.1 in-toto wrapper and the pinned top-level ``predicateType`` (the ``--type`` value, :ref:`Section 4.2.3 <section-4.2.3>`).

Keys: ``cosign generate-key-pair`` output works as-is; converting an external key into cosign's key format is a producer concern.