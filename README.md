# Law Firm Feed

34 outbound DLP rules for the legal industry, plus one optional rule. English and German language coverage. They cover lawyers leaving for another firm, mail to personal accounts, Microsoft Purview sensitivity labels, German identity numbers, misdirected email and document management system (DMS) stamps.

Every rule is a standard `type: "rule"`, so the set loads into any Sublime tenant today. It does not depend on the DLP module or the `type: "dlp"` rules in the official feed. All rules use built-in lists (`$org_domains`, `$free_email_providers`, `$org_display_names`, `$org_vips`, `$free_file_hosts`), so nothing is tenant-specific.

## How to use it

1. Add this repository as a feed in Sublime: `https://github.com/williammirelesSS/law-firm-dlp-feed`, branch `main`, detection rules. The rules live in `detection-rules/`. Or copy that folder into your own feed repository.
2. Leave the rules without actions. They are there for visibility and hunting, not blocking.
3. To hunt, open any rule, copy its `source`, and run it in Hunt over the longest window you have. Every source runs as-is.
4. Start with the six hunts below. They are the highest-signal for a law firm and need no setup.

## Suggested first hunts

| Order | Rule | Why first |
|---|---|---|
| 1 | `purview_label_inventory_external` | Shows which sensitivity labels are actually in use and how much labelled mail goes external. Feeds the label names into the other Purview rules. |
| 2 | `leaver_self_send_name_match` | Attachments sent to a personal address matching the sender's own name. The quietest leaver signal. |
| 3 | `leaver_work_backup` | "Backup", "meine Unterlagen", "private Kopie" to GMX, Gmail and similar. Near-zero noise. |
| 4 | `legal_client_matter_lists_freemail` | Mandantenliste, Aktenliste, Honorarumsatz and timesheets to personal email. The book of business. |
| 5 | `leaver_client_move_announcement` | A lawyer telling clients they are moving firm, or talking about taking a mandate with them. |
| 6 | `exfil_bulk_document_set_freemail` | Three or more PDFs, Word files or saved emails to a personal address in one message. |

## Setup that improves precision

| Rule | What to set |
|---|---|
| `purview_label_confidential_freemail`, `purview_label_internal_only_external`, `purview_label_attachment_freemail` | Replace the label-name alternation in the regex with your real Purview label names (run the inventory hunt first). |
| `dms_document_id_freemail` | Replace the generic NetDocuments / iManage patterns with your DMS's actual document-number format. |
| `dms_document_metadata_freemail` | Add your DMS or practice-management system name if it is not in the list. |
| `misdirected_vip_sender_display_name` | Needs VIPs configured in the platform. |
| `de_iban_bulk` | Raise the threshold above the number of IBANs in your own email footer, if that is 5 or more. |
| `optional/leaver_upcoming_leavers_list` | Create a string list called `org_upcoming_leavers` (sender addresses, ideally synced from an Entra ID group), then move the file into `detection-rules/`. It fails validation until the list exists. |

## Design notes

- **Personal accounts.** "Personal email" means `$free_email_providers` plus an explicit list of German webmail domains (GMX, web.de, T-Online, freenet, Posteo, mailbox.org and others), checked across To, Cc and Bcc. A personal address on Bcc is a common way to keep a copy without the thread noticing.
- **Disclaimers.** Rules about privilege and confidentiality match the subject line and attachment names, not the body. Every outbound law-firm email carries a confidentiality disclaimer, so a body match would fire on everything.
- **German Outlook prefixes.** The forward and reply rules recognise `WG:` and `AW:` as well as `Fwd:` and `Re:`.
- **Private clients.** Individual clients who write from a personal address are the main false-positive source for the work-product rules. Each rule's description says how to tune for that.
- **German ID rules.** These are based on the official Sublime DLP rules, with three fixes. The identity-card pattern is grouped properly, the passport rule no longer requires a `.de` sender domain, and the IBAN rule counts IBANs instead of firing on one.

## Rule index

**Leavers**
- `leaver_self_send_name_match`: attachments to a personal address matching the sender's name
- `leaver_work_backup`: backup and "my files" language to personal email
- `leaver_departure_staging`: handover, last-day and resignation material to personal email
- `leaver_client_move_announcement`: telling clients about a move, or taking a mandate along
- `leaver_cv_application_external`: CV, application or Arbeitszeugnis sent externally
- `leaver_auto_forward_freemail`: mail auto-forwarded to a personal address by an inbox rule

**Exfiltration**
- `exfil_bulk_data_freemail`: spreadsheet and database exports over 100 KB, or PST / OST / MBOX archives
- `exfil_large_archive_freemail`: ZIP / 7z / RAR over 5 MB
- `exfil_bulk_document_set_freemail`: three or more documents or saved emails in one message
- `exfil_forwarded_documents_freemail`: forwarded (Fwd / WG) work documents, with personal paperwork excluded
- `exfil_personal_cloud_links_freemail`: WeTransfer, Dropbox, Google Drive, iCloud and similar links
- `exfil_credentials_freemail`: passwords, keys and credential files, with meeting-invite passcodes excluded

**Law-firm content**
- `legal_privileged_material_freemail`: privilege and Berufsgeheimnis markings in subject or filename
- `legal_client_matter_lists_freemail`: client lists, matter lists and billing data
- `legal_precedents_templates_freemail`: Muster, Vorlagen, clause banks and know-how
- `legal_court_pleadings_freemail`: Schriftsätze, Gutachten, Urteile and transaction documents
- `legal_contracts_freemail`: contracts and agreements, with personal contracts excluded
- `legal_court_file_number_freemail`: German court file numbers (Az.) in subject or body

**Microsoft Purview**
- `purview_label_inventory_external`: every labelled message going external (sizing hunt)
- `purview_label_confidential_freemail`: Confidential-type label to personal email
- `purview_label_internal_only_external`: internal-only label leaving the firm
- `purview_label_attachment_freemail`: labelled Office or PDF attachment to personal email

**German identifiers**
- `de_identity_card_number`, `de_passport_number`, `de_tax_id_number`, `de_drivers_license_number`, `de_iban_bulk`

**Misdirected email**
- `misdirected_display_name_external`: external address displayed with a colleague's name
- `misdirected_domain_typo`: recipient domain within two characters of the firm's own
- `misdirected_reply_all_internal_content`: internal discussion on a thread with external parties
- `misdirected_vip_sender_display_name`: the same scenario, scoped to VIP senders

**DMS and shadow IT**
- `dms_document_id_freemail`: DMS document-number stamps in filenames or body
- `dms_document_metadata_freemail`: DMS metadata inside Office documents
- `shadow_it_work_email_signup` (inbound): work address used to sign up for cloud storage or AI tools
