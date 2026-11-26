
.. _chapter-apx7:

.. _section-7:

Security Considerations
==========================

.. _section-7.1:

Registrar-Clerk trust
-------------------------

This protocol assumes the Registrar and the Clerk operator trust each other. The pre-shared verification key is the trust anchor for the register signature (:ref:`Section 4.2.2 <section-4.2.2>`). Distributing, rotating, and storing that key is a deployment concern and is out of scope for this document.

.. _section-7.2:

Registration channel
------------------------

The transport between Registrar and Clerk SHOULD be HTTPS.

.. _section-7.3:

Endpoint authentication and load
------------------------------------

Authentication mechanisms for the ``register``, ``challenge``, and ``evidence`` endpoints are out of scope. A deployment MAY add one, but the protocol's security does not rely on it: it rests on the Envelope signature (:ref:`Section 4.2.2 <section-4.2.2>`) and the Evidence possession proof (:ref:`Section 4.5 <section-4.5>`).

Without it, a caller can create challenges without bound, which is a denial-of-service risk. The Clerk SHOULD cap the number of stored challenges and SHOULD apply per-source rate limits.

.. _section-7.4:

Credential presentation
--------------------------------------

APX does not define a presentation step. Anyone who holds a copy of the Verdict credential can present it unless the consumer requires holder binding. The ``cnf.jwk`` claim does not by itself prove that the presenter holds the key. The consumer profile defines holder binding and replay protection (for example, CRATB-Auth). A consumer that relies on the credential needs to check its status against the status list (:ref:`Section 5.2.6 <section-5.2.6>`). What it does when the status cannot be checked is left to its own policy.

