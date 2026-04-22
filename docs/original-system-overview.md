# Original System Overview – CICS GenApp

This document summarises the original CICS GenApp COBOL application based on inspection of `cicsdev/cics-genapp/base/src`, `base/data`, `base/Architecture.md`, and `base/Reference.md`.

## High-Level Domain Model

GenApp is a sample line-of-business (insurance-style) application centred around:

- **Customers** – insured parties with contact details, stored in a VSAM KSDS.
- **Policies** – insurance policies/quotes associated with customers, stored in a VSAM KSDS.
- **Accounts** – financial/accounting records associated with customers and policies.
- **Statistics/Administration** – utilities and batch-style programs for statistics and setup.

### Core Business Concepts

- `Customer` – identified by a customer number, with name, address, and contact information.
- `Policy` – identified by a policy number, with product code, status, dates, and premium details.
- `Account` – customer- and policy-related financial records.
- `System Setup` – programs to initialise files and environment (e.g., `lgsetup`).
- `Web Services` – a CICS web entry program (`lgwebst5`) and copybooks (`soa*.cpy`, `soav*.cpy`) defining request/response structures.

## Main CICS Transactions and Programs

Entry-point COBOL programs under `base/src` include (one-line purpose based on naming and reference docs):

- `lgacdb01.cbl` – Account database operations (create/update/read account records).
- `lgacdb02.cbl` – Additional account database operations or variants.
- `lgacus01.cbl` – Account/customer maintenance logic (account–customer linkage).
- `lgacvs01.cbl` – Account view/select screen handling.
- `lgapdb01.cbl` – Policy database access and maintenance logic.
- `lgapol01.cbl` – Policy maintenance (create/update policy details).
- `lgapvs01.cbl` – Policy view/select screen handling.
- `lgastat1.cbl` – Statistics or status display utility.
- `lgdpdb01.cbl` – Policy display database program.
- `lgdpol01.cbl` – Policy display logic.
- `lgdpvs01.cbl` – Policy display view/select program.
- `lgicdb01.cbl` – Customer inquiry database access.
- `lgicus01.cbl` – Customer maintenance logic.
- `lgicvs01.cbl` – Customer inquiry/view screen handling.
- `lgipdb01.cbl` – Policy input/new business database program.
- `lgipol01.cbl` – Policy input/new business policy program.
- `lgipvs01.cbl` – Policy input/new business view program.
- `lgsetup.cbl` – System setup and initialisation utilities.
- `lgstsq.cbl` – Statistics sequential/batch-style processing program.
- `lgtestc1.cbl` – Test/customer-related sample program.
- `lgtestp1.cbl` – Test/policy-related sample program (variant 1).
- `lgtestp2.cbl` – Test/policy-related sample program (variant 2).
- `lgtestp3.cbl` – Test/policy-related sample program (variant 3).
- `lgtestp4.cbl` – Test/policy-related sample program (variant 4).
- `lgucdb01.cbl` – Customer update database program.
- `lgucus01.cbl` – Customer update logic.
- `lgucvs01.cbl` – Customer update view program.
- `lgupdb01.cbl` – Policy update database program.
- `lgupol01.cbl` – Policy update logic.
- `lgupvs01.cbl` – Policy update view program.
- `lgwebst5.cbl` – CICS web entry/STS integration program using SOA copybooks.

Copybooks and BMS maps:

- `lgcmarea.cpy` – Common communication area shared between programs.
- `lgpolicy.cpy` – Core policy data structure.
- `pollook.cpy`, `polloo2.cpy` – Policy lookup/list structures.
- `soaic01.cpy`, `soaipb1.cpy`, `soaipe1.cpy`, `soaiph1.cpy`, `soaipm1.cpy` – SOA request/response structures.
- `soavcii.cpy`, `soavcio.cpy`, `soavpii.cpy`, `soavpio.cpy` – Variant SOA copybooks for inbound/outbound payloads.
- `ssmap.bms` – BMS mapset defining the main GenApp screens.

## Data Stores

The primary data stores are VSAM key-sequenced datasets (KSDS):

- `ksdscust` – Customer master file (see `base/data/ksdscust.txt` for sample content).
- `ksdspoly` – Policy master file (see `base/data/ksdspoly.txt` for sample content).

These are used by the `lgac*`, `lgic*`, `lguc*` (customer/account) and `lgap*`, `lgdp*`, `lgip*`, `lgup*` (policy) programs for keyed access by customer or policy number.

## Integration Points

From the COBOL source and documentation:

- **CICS VSAM integration** – Programs issue CICS file control commands (READ/WRITE/REWRITE/DELETE) against VSAM KSDS files for customers and policies.
- **CICS BMS maps** – `ssmap.bms` defines screens for customer/policy maintenance and inquiry; programs send/receive maps to interact with users.
- **CICS Web Services/STS** – `lgwebst5.cbl` is a CICS web entry program that processes web/STS requests using SOA copybooks (`soa*.cpy`, `soav*.cpy`).
- **Batch/statistics** – `lgstsq.cbl` performs sequential processing over data to compute statistics, aligning with a batch-style job.

## Summary

GenApp is a traditional CICS/COBOL VSAM-based insurance sample application, with clear domains:

- Customer management (`lgic*`, `lguc*`, `lgacus*`, VSAM `ksdscust`).
- Policy management (`lgap*`, `lgdp*`, `lgip*`, `lgup*`, `lgpolicy.cpy`, `pollook.cpy`, `polloo2.cpy`, VSAM `ksdspoly`).
- Accounts and financials (`lgacdb*`).
- Statistics and setup (`lgstsq`, `lgsetup`, `lgastat1`).
- Web/STS integration (`lgwebst5`, `soa*.cpy`, `soav*.cpy`).

This document will be used as the baseline for mapping COBOL functions into Java microservices and for driving Jira stories in project CM.
