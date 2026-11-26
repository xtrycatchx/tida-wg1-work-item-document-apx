
.. raw:: latex

    \pagestyle{plain}
    \thispagestyle{plain}
    %% Suppress all TOC entries for the overview chapter (sections, subsections,
    %% Copyright addcontentsline) - restored just before \sphinxtableofcontents
    %% so chapters from Introduction onward appear in the TOC normally.
    \let\osiaorigaddcontentsline\addcontentsline
    \renewcommand{\addcontentsline}[3]{}

.. _chapter-overview:

APX: Attestation-Provenance Exchange
=====================================

.. only:: html

    **Exchanging cryptographic provenance proofs for identity credentials**

.. raw:: latex

    \begin{center}\textbf{Exchanging cryptographic provenance proofs for identity credentials}\end{center}

*Draft specification, v0.1*

Abstract
--------

This document describes APX (Attestation-Provenance Exchange), an architecture and information model for exchanging cryptographic provenance proofs for identity credentials. A Registrar submits a signed Envelope that contains build-time statements (in-toto attestations such as a manifest, a provenance statement, or an SBOM). A Workload later proves possession of the proving key registered for that Envelope. A Clerk then binds the two together and returns a verifiable credential. The credential subject is a Workload. Profiles for agent identity, for example CRATB-Auth, may build on this core.

History
-------

.. list-table::
    :header-rows: 1
    :widths: 10 26 18 14 32

    * - Version
      - Document
      - Approval
      - Focus Group
      - Working Group
    * - 0.1
      - Attestation-Provenance Exchange (APX)
      -
      - TIDA
      - WG1

To access this Document as a PDF, open the published documentation site in your web
browser and click the PDF download link, or navigate directly to
https://debora-com.github.io/CUTEspecs/specification.pdf.

Authors
-------

.. list-table::
    :header-rows: 1
    :widths: 30 40 30

    * - Name
      - Affiliation
      - Contact
    * - Debora COMPARIN
      - Thales
      - debora.comparin@thalesgroup.com
    * - Yonatan Alejandro ARGUELLO
      - Thales
      - yonatan.arguello@thalesgroup.com
    * - Ward DUCHAMPS
      - Thales
      - ward.duchamps@thalesgroup.com
    * - Frank VAN DAMME
      - Thales
      - frank.van-damme@external.thalesgroup.com
    * - Hugo BRUNA
      - Thales
      - hugo.bruna@thalesgroup.com
    * - David PRINCE
      - Thales
      - david.prince@thalesgroup.com
    * - Paul Sydney OROZCO
      - Thales
      - paul.orozco@thalesgroup.com

Keywords
--------

agentic workload, provenance, verifiable credentials

.. raw:: latex

    \newpage
    \osiacenternextsection

Foreword
--------

Agentic AI systems now act as autonomous entities capable of reasoning, planning, and
executing tasks across services, infrastructures, and organizational boundaries. This
fundamentally changes the nature of identity in digital systems: identity is no longer
limited to human users and static machine instances but must also encompass autonomous,
dynamic, and goal-oriented agents. And identity alone is insufficient - the core question
extends beyond "what an entity is" to "whether, and under what conditions, that entity
should be trusted to act".

The Focus Group on Trust and Identity for humans and agentic AI (FG-TIDA) addresses trust
management and interoperable digital identity infrastructure for humans and for agentic AI,
supporting the development of secure, trustworthy digital ecosystems in which humans and
agentic AI can safely interact and collaborate.

FG-TIDA was established under Recommendation ITU-T A.7 (Focus groups: Establishment and
working procedures), with ITU-T Study Group 17 as its parent group. Further details are
available in the group's terms of reference.

For more news and updates about the TIDA Community, including its supporter organizations,
please refer to the community's published resources.

.. raw:: latex

    \osiacenternextsection

Note
----

This document is an early draft (v0.1) prepared for contribution to the Focus Group on 
Trust and Identity for humans and agentic AI (FG-TIDA) for discussion. The architecture, 
information model, and profile boundaries are proposals open to review. Title, date, 
copyright statements have not yet been finalised. The contributors intend to gather
feedback from TIDA members on the core exchange and consideration for adoption.

.. Intellectual Property Rights - section commented out. To restore it, remove the
   leading ".." and the indentation below, and re-add a ".. raw:: latex" block with
   "\osiacenternextsection" before the heading (to centre it on the PDF).

   Intellectual Property Rights
   ----------------------------

   ITU draws attention to the possibility that the practice or implementation of this
   Recommendation may involve the use of a claimed Intellectual Property Right. ITU takes no
   position concerning the evidence, validity or applicability of claimed Intellectual Property
   Rights, whether asserted by ITU members or others outside of the Recommendation development
   process.

   As of the date of approval of this Recommendation, ITU had not received notice of intellectual
   property, protected by patents/software copyrights, which may be required to implement this
   Recommendation. However, implementers are cautioned that this may not represent the latest
   information and are therefore strongly urged to consult the appropriate ITU-T databases
   available via the ITU-T website at http://www.itu.int/ITU-T/ipr/.
   Implementers should also be aware that the organization that originated the technically
   equivalent document listed in the Bibliography may have received notices of intellectual
   property required for the implementation of this Recommendation.

.. raw:: latex

    %% Copyright notice commented out. To restore, uncomment the lines below.
    %% \vspace{4em}
    %% \begin{center}\textcopyright{} ITU 2024\end{center}
    %%
    %% \medskip
    %% All rights reserved. No part of this publication may be reproduced, by any means whatsoever, without the
    %% prior written permission of ITU.

    %% Restore \addcontentsline so the TOC below (and Chapter 1 onward) populate.
    \let\addcontentsline\osiaorigaddcontentsline
    \sphinxtableofcontents
    \osiamainmatter

.. toctree::
    :maxdepth: 3
    :hidden:

    chapters/01_introduction
    chapters/02_conventions
    chapters/03_architecture
    chapters/04_data_model
    chapters/05_protocol_flows
    chapters/06_error_handling
    chapters/07_security
    chapters/08_privacy
    chapters/09_versioning_registries
    chapters/10_conformance
    chapters/11_relationships
    chapters/12_references
    chapters/14_appendix_a
    chapters/15_appendix_b
