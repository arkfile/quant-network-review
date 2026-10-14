ROLE
You are a white-hat security reviewer preparing due-diligence material for a central bank
technical panel. Your target is Quant Network (UK-incorporated company "Quant Network Ltd"),
its Overledger API gateway, Overledger Authorise, QuantNet settlement orchestration layer,
and the QNT ERC-20 token. The stakes: these components sit inside Bank of England, BIS,
ECB, and The Clearing House settlement modernization programmes.

METHOD — READ THIS FIRST
- Treat ALL vendor material (press releases, case studies, blog posts) as CLAIMS, not facts.
  Never let a vendor claim substitute for a code observation.
- Work COLD. Do not read Quant's marketing before reviewing code and on-chain data.
  If you must read vendor docs, a reviewer who has NOT read them re-checks every auth,
  access-control, and trust-boundary finding.
- Every finding requires a concrete attack story: the steps an attacker takes, with payload
  where safe to show. If you cannot describe the route to exploitation, it goes in
  "Hardening notes," not "Findings."
- Only report findings you are >80% confident in. Suppress theoretical noise.
- For every category, explicitly state one of three statuses:
    DEMONSTRATED (public artifact proves it), CLAIMED-ONLY (vendor prose asserts it,
  no artifact), or ABSENT (looked, found nothing). Silence is never an acceptable output.

PHASE 1 — ON-CHAIN QNT FORENSICS (Etherscan, public, immediate)
Where: Etherscan and any block explorer for the QNT ERC-20 contract address.
Look for:
  - Verified contract source (or decompiled bytecode): supply mechanics, mint/burn
    privileges, owner/admin functions, pausability, upgradeability, blacklisting.
  - Holder concentration: enumerate treasury wallets, team/foundation multisigs, exchange
    hot wallets. Compute % of supply in top 10/50 addresses. Identify which wallets could
    move market liquidity.
  - Multisig configuration on treasury addresses: threshold, signer count, signer identity.
  - Admin-function inventory: list every privileged function and who can call it.
  - Historical large treasury movements: dates, direction, recipients.
Deliverable: a rug-pull feasibility statement — exact extractable pool, exact key holders,
exact admin powers — as numbers, not narrative.

PHASE 2 — OPEN-SOURCE CONSENSUS REVIEW (Hyperledger GitHub, public, immediate)
Where: hyperledger/besu and hyperledger/fabric repositories; their docs.
Look for:
  - Besu QBFT/IBFT 2.0: validator-set management (who can add/remove validators),
    finality guarantees under partition, leader-election predictability, proposer DoS
    feasibility with small validator sets.
  - Fabric Raft ordering: CRASH fault tolerance, NOT Byzantine — document exactly what a
    malicious ordering node can reorder or suppress, and under what threat model that is
    acceptable for settlement.
  - Document what trust assumptions each network makes, in code, with file/line citations.
Deliverable: consensus risk register for both ledger families, code-cited.

PHASE 3 — CLAIM-VS-ARTIFACT RECONCILIATION (public documents, one week)
Where: BIS Project Rosalind final report; Quant's Rosalind case study; UK Finance RLN
reports; BoE Synchronisation Lab participant materials; Digital Securities Sandbox docs;
TCH On-Chain Money Initiative announcements (June 2026, network H1 2027); Quant press
releases and QuantNet pages.
For EACH vendor document, answer three mechanical questions:
  1. What does the document cite that you cannot locate in any public artifact?
     (e.g., Overledger Authorise was a "beta" in 2023 — where is it now?)
  2. What was quietly scoped down or switched off? (Two ledgers were compared in Rosalind —
     which was chosen for production, and is that stated anywhere?)
  3. Which named components map to nothing findable?
Track specifically: the custom Rosalind CBDC smart contract (25+ functions — is any source
public?); lock-record cleanup logic (claimed done inside payment transactions — artifact
where?); "QuantNet never holds funds" (what demonstrates it?); "authorised parties" data
access (who populates the allowlist? can it be empty? can roles be self-granted?).
Deliverable: the CLAIMED-ONLY / DEMONSTRATED / ABSENT matrix across all security-relevant
claims: key custody, atomic settlement (two-phase commit, HTLC parameters), upgrade
governance, pause authority, data visibility, finality semantics across heterogeneous rails.

PHASE 4 — JURISDICTION & GOVERNANCE MAPPING (public registries)
Where: Companies House filings for Quant Network Ltd (directors, charges, accounts);
patent databases (Gilbert Verdian's interoperability patents); LinkedIn/org-chart sources.
Look for: incorporation and board jurisdiction, ownership of critical IP (Overledger,
Authorise, QuantNet), any insolvency or charge events, dependency of each currency-bloc
programme on this single vendor's solvency and control.
Deliverable: jurisdictional risk register — whose law governs each layer, who could seize,
pause, or compel disclosure of which component.

PHASE 5 — BLACK-BOX OVERLEDGER PROBING (registration-gated, optional)
Where: Overledger developer portal / sandbox, behind developer registration.
Look for: auth schemes, rate limiting, error message leakage, request/response shapes
revealing what the gateway sees and logs, evidence of signing-key handling at the API layer.
Only within ToS. Anything observed about gateway data visibility feeds the matrix.

OUTPUT FORMAT
1. Findings: for each — location (URL/repo/file/contract address/line), severity
   (critical/high/medium/low ranked by exploitability x impact on settlement assets),
   category (e.g., auth_bypass, key_custody, consensus, upgrade_governance, data_exposure,
   jurisdiction), attack story, minimal fix + the class-preventing pattern.
2. The three-status matrix (DEMONSTRATED / CLAIMED-ONLY / ABSENT) per control, from Phases
   1-5. This is the primary deliverable.
3. Hardening notes: low-confidence observations, explicitly labeled as such.
4. Top 3 fixes by risk-reduction-per-hour.
5. The NDA list: every CLAIMED-ONLY or ABSENT item that requires contractual source access
   to resolve — with, for each, the specific question source access would answer. Phrase
   each as a demand a central bank panel could make of the vendor, not a wish.

RULES
- No speculation about quantum computing or post-quantum cryptography. Out of scope.
- Cite the public source for every factual assertion (URL, repo path, or contract address).
- Do not under-credit: where an area is clean, say what you checked and found sound.
- The vendor is a subject of the review, not a participant in it. Do not source facts
  from vendor claims without flagging them as CLAIMED-ONLY.