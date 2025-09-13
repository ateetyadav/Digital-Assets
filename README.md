# Digital-Assets
Digital Asset to build Asset tokenization projects
This Repository has been created to practise around digital Asset.
To start the journey we will use the Canton Network to build simple use cases.

**Tokenised Deposit**
===========================
I will reformat your provided text into a clean, well-structured, and easy-to-read document that's perfect for a GitHub README.

###  Tokenized Deposit Business Process
---------------------------------------------

This document outlines the actors, roles, and a detailed lifecycle of a tokenized deposit system, providing a clear overview of the operational flow from onboarding to redemption.


### Actors and Roles
------------------------

This section identifies the key participants and their responsibilities within the tokenized deposit ecosystem.

* **Bank (Issuer):** Manages the core deposit token functions.
    * Mints and burns deposit tokens.
    * Holds and manages the cash reserves that back the tokens.
    * Handles all compliance and regulatory requirements.

* **Customer (Wallet Holder):** The end-user of the system. This includes retail clients, SMEs, and corporate entities.
    * Requests the minting and redemption of tokens.
    * Transfers tokens for peer-to-peer (P2P) or merchant (P2M) payments.
    * Receives periodic interest payments.

* **Operator/Trustee (Optional):** A third party that provides independent oversight.
    * Ensures the segregation of cash reserves.
    * Oversees operational and financial controls.

* **Compliance/KYC Officer:** A key role in maintaining regulatory standards.
    * Approves the onboarding of new customers.
    * Monitors for AML (Anti-Money Laundering) and sanctions compliance.
    * Applies transaction and wallet limits.

* **Regulator/Auditor:** An external party with oversight capabilities.
    * Receives real-time proofs and reports.
    * Has the authority to request a freeze or a system-wide halt.

* **Network Admin:** Manages the technical aspects of the blockchain network.
    * Can pause or un-pause smart contracts.
    * Manages system upgrades and emergency controls.

###  Lifecycle Phases
-----------------------

The tokenized deposit process is broken down into distinct, sequential phases.

* **1. Onboarding and Account Setup:**
    * Performs **KYC/AML checks** on the customer.(Digital Identity can be consider at later stage for this implementation).
    * Creates a digital wallet for the customer and grants **whitelist** status.
    * Assigns risk tiers, transaction limits, and specific interest/fee plans.

* **2. Funding and Mint Request:**
    * The customer deposits fiat currency into a bank account or transfers funds from an internal core banking system (off-ledger).
    * The customer submits a request to mint a specific amount of tokens.

* **3. Compliance Approval:**
    * The system automatically runs **sanctions, AML, and limit checks**.
    * Transactions flagged for suspicious activity are reviewed manually.
    * The mint request is either approved or rejected.

* **4. Minting:**
    * The bank mints deposit tokens **1:1** against the held reserves.
    * The newly created tokens are issued to the customer's digital wallet.
    * The internal reserve ledger is updated to reflect the new token supply.

* **5. Transfers and Payments:**
    * Customers can transfer tokens to other customers (P2P) or to merchants (P2M).
    * Payments are settled instantly on-ledger with immediate finality.
    * Off-chain notifications and receipts are generated to confirm the transaction.

* **6. Interest Accrual and Fees:**
    * The system periodically calculates and accrues interest based on the customer's plan.
    * Interest is credited to the customer as new tokens or by adjusting their balance.
    * Optional management fees are charged to the customer's wallet.

* **7. Corporate Actions and Controls:**
    * The system has controls to freeze/unfreeze specific wallets or tokens.
    * Addresses can be blacklisted.
    * Per-user, per-token, or cumulative limits can be enforced.
    * In an emergency, the entire system can be paused and unpaused.

* **8. Redemption and Burn:**
    * A customer requests to redeem their tokens for fiat currency, specifying a bank account for payout.
    * The bank **burns** the corresponding number of tokens.
    * The bank initiates a wire transfer of the fiat currency off-ledger.

* **9. Exceptions, Disputes, and Reversals:**
    * The system can flag and investigate suspicious activity.
    * Policies may allow for **clawbacks** of funds under a court or regulator order.
    * The system handles failed transfers and other stuck states.

* **10. Reporting and Attestations:**
    * The system provides real-time dashboards and daily proofs of reserves.
    * All decisions and transactions are logged for a complete **audit trail**.
    * Comprehensive reports are available for regulators and auditors.

###  Core Functions (API Mapping)

This section provides a high-level view of the key functions and their corresponding actions.

* **Onboarding:** `RequestOnboard`, `ApproveOnboard`, `SetLimits`, `AssignPlan`, `Whitelist/Blacklist`
* **Minting:** `RequestMint(amount, ref)`, `ApproveMint`, `Mint(amount, to)`
* **Transfers:** `Transfer(to, amount)`, `BatchTransfer`, `EscrowTransfer(escrow terms)`
* **Controls:** `Freeze(wallet|token)`, `Unfreeze`, `PauseAll`, `UnpauseAll`, `SetLimit`, `SetKycStatus`
* **Interest/Fees:** `AccrueInterest`, `CreditInterest`, `ChargeFee`
* **Redemption:** `RequestRedeem(amount)`, `ApproveRedeem`, `Burn(amount)`, `SetPayoutRef`
* **Exceptions:** `FlagTransaction`, `ResolveFlag`, `Clawback(tokenRef, reason)`
* **Reporting:** `GetBalance`, `GetHolders`, `GetReserveProof`, `GetTxHistory`

###  Governance, Risk, and Compliance (GRC)

These are the foundational principles and controls that govern the system's operation.

* **KYC States:** Each user's status is managed through states: `Pending`, `Approved`, `Rejected`, `Suspended`.
* **Policy Checks:** All sensitive actions (mint, transfer, redeem) are subject to automated policy checks.
* **Sanctions Registry:** The system maintains a versioned registry of blacklisted addresses.
* **Limits:** Per-transaction, daily, wallet, and cumulative limits are enforced to manage risk.
* **Emergency Controls:** The system includes emergency measures such as `pause`, `freeze`, and `clawback` (subject to policy).
* **Auditability:** All decisions are logged with reason codes, providing an immutable and exportable audit trail.

### DAML-Oriented Design Sketch for a Tokenized Deposit
----------------------------------------------------------------

This document outlines the core components and logic for a tokenized deposit system built using DAML. It provides a blueprint for the smart contracts, key choices (actions), and critical checks.


### Core DAML Templates

The system is built on a set of foundational DAML templates that define the digital assets and their associated rights and rules.

* **`Wallet`**
    * **Signatory:** `holder`
    * **Observer:** `bank`
    * **Fields:** `holder`, `kycStatus`, `limits`, `plan`, `frozen`
    * **Choices:** `SetKycStatus`, `SetLimits`, `Freeze`, `Unfreeze`

* **`Token` (DepositToken)**
    * **Signatory:** `bank`
    * **Observer:** `holder`
    * **Key:** `(bank, holder)`
    * **Fields:** `holder`, `balance`, `currency`, `frozenFlag`
    * **Choices:**
        * `Mint` (controlled by `bank`)
        * `Transfer` (controlled by `holder`; performs checks on KYC, limits, and frozen status)
        * `CreditInterest` (controlled by `bank`)
        * `Burn` (controlled by `bank`)
        * `Freeze`/`Unfreeze` (controlled by `bank` with the regulator as an observer)

* **`MintRequest`**
    * **Signatory:** `customer`
    * **Observer:** `bank`
    * **Choices:**
        * `Approve` (by `bank`, leads to token minting)
        * `Reject` (by `bank`)

* **`RedeemRequest`**
    * **Signatory:** `customer`
    * **Observer:** `bank`
    * **Choices:**
        * `Approve` (by `bank`, leads to token burning)
        * `Reject`

* **`PolicyRegistry`**
    * **Signatory:** `bank`
    * **Fields:** `blacklist`, `limits`, `paused`
    * **Choices:** `UpdateBlacklist`, `SetPaused`, `SetLimits`

### Key Checks & Logic

To ensure security and compliance, every action is governed by a series of checks.

* **`Transfer` Requirements:**
    * KYC status must be `Approved`.
    * The holder's wallet must **not** be frozen.
    * The `PolicyRegistry` system-wide pause flag must be `False`.
    * The transaction amount must be within the holder's limits.

* **Interest Accrual:**
    * This is handled by a scheduled off-chain trigger or a DAML script.
    * It calls the `Accrue` or `CreditInterest` choices on the relevant contracts to update balances.

###  Core Contract/Choice Logic

Each choice acts as a function, taking inputs and producing outputs in an atomic transaction.

* **Inputs:** `caller` (party), `amount`, `to` (recipient), `metadata` (references), `policy state`
* **Outputs:** Updated balances, new contracts, and an immutable audit log.
* **Errors Handled:** `NotKycApproved`, `Frozen`, `Paused`, `OverLimit`, `Blacklisted`, `InsufficientBalance`

###  Edge Cases

The design accounts for several potential edge cases to ensure robustness.

* **Zero/Negative Amounts:** The contracts prevent the processing of invalid amounts.
* **Precision and Rounding:** Uses a `Decimal` type to avoid floating-point errors.
* **Race Conditions:** Utilizes DAML's contract key system or UTXO-style contracts to prevent concurrent transfers from causing race conditions.
* **Plan Changes:** The system is designed to handle mid-transfer changes to a holder's plan or status.
* **Redemption:** The system supports both partial redemption and the full burning of a token contract.

###  Starter Templates

If you're interested, this design can be used to generate starter DAML templates and scripts for your repository.

* `deposit/Token.daml` (with `DepositToken` and `Wallet` templates)
* `deposit/Policy.daml` (with the `PolicyRegistry` template)
* `scripts/TokenizedDepositDemo.daml` (an end-to-end demo script)


#


#


#
