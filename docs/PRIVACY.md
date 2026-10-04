# Via Carraria Privacy & Cookie Policy

*Last updated: September 25, 2026*
*Version: 1.0*

Via Carraria ("we", "our", or "the Platform") provides an open, privacy-respecting spatial knowledge graph exploration and neural vector retrieval engine. This document details how we handle information, our lawful bases for data processing under the European Union General Data Protection Regulation (GDPR, Regulation EU 2016/679) and California Consumer Privacy Act (CCPA), and our taxonomy of web cookies and browser storage.

---

## 1. Data Controller & Architecture

- **Data Controller**: Via Carraria Platform
- **Contact & Inquiries**: `privacy@viacarraria.internal`
- **Data Protection Principle**: Minimalist data ingestion, tenancy-isolated vector search, zero selling of personal data, and automated retention lifecycle enforcement.

---

## 2. Personal Data We Collect & Lawful Basis

| Data Category | Specific Elements | Purpose | Lawful Basis (GDPR Art. 6) |
| :--- | :--- | :--- | :--- |
| **Authentication & Profile** | Email address, username, password hash (PBKDF2/Argon2), preferred language, subscription tier | Account creation, authentication, entitlement checks | **Contract Performance** (Art. 6(1)(b)) |
| **Anonymous Session Tokens** | Cryptographically random UUID session identifier | Enable instant guest research without forced registration | **Legitimate Interest** (Art. 6(1)(f)) |
| **Knowledge Graphs & Sources** | User-created nodes, connecting edges, attached research documents (PDF, Markdown, text), vector embeddings | Graph visualization, semantic search, AI narrative synthesis | **Contract Performance** (Art. 6(1)(b)) |
| **Telemetry & Quality Logs** | Anonymized ad impression counts, viewability duration, rate limit counters | Anti-abuse protection, platform stability, aggregate ad metrics | **Legitimate Interest** (Art. 6(1)(f)) |
| **Advertising Identifiers** | Third-party Google AdSense cookies and identifiers (when consent is granted) | Contextual or personalized advertising to sustain free platform tiers | **Explicit Consent** (Art. 6(1)(a) & ePrivacy Art. 5(3)) |

---

## 3. Cookie & Local Storage Taxonomy

We classify our browser storage into two distinct categories:

### Category A: Strictly Necessary / Essential (Always Active)
These cookies and local storage records are required for technical operation and security. In accordance with Article 5(3) of the EU ePrivacy Directive (Directive 2002/58/EC), strictly necessary cookies do not require prior opt-in consent, but their operation is fully disclosed below.

- **`better-auth.session_token`**:
  - **Type**: HTTP-only, Secure Cookie (`SameSite=Lax`)
  - **Duration**: 90 days (sliding expiration updated every 24h)
  - **Purpose**: Authenticates API calls, secures guest and registered sessions, protects against Cross-Site Request Forgery (CSRF), and enforces rate-limiting tiers.
- **`vc_ad_consent_status` & `vc_ad_consent_record`**:
  - **Type**: Browser `localStorage`
  - **Duration**: Persistent until cleared by user
  - **Purpose**: Retains your privacy consent choice, choice timestamp, and policy version to honor your preferences without prompting on every page refresh.

### Category B: Advertising & Analytics (Optional — Requires Consent)
These cookies are served exclusively when users on Free or Guest tiers choose to support the platform through Google AdSense. **Subscribers on Pro Tier never receive advertising cookies or third-party ad scripts.**

- **Google AdSense (`adsbygoogle`, DoubleClick)**:
  - **Type**: Third-party cookies (`__gads`, `__gpi`, `IDE`)
  - **Provider**: Google LLC (certified IAB TCF 2.2 vendor)
  - **Duration**: Up to 13 months
  - **Purpose**: Frequency capping, aggregated ad reporting, combating fraud, and serving personalized ads based on interests (if "Accept All" is selected) or contextual-only ads without profiling (if "Contextual Only" is selected).

---

## 4. User Rights Under GDPR & CCPA

Under applicable data protection laws, you possess the following rights:

1. **Right to Withdraw Consent (GDPR Art. 7(3))**: You can withdraw or adjust your advertising consent at any time via the *Privacy & Cookie Policy* dialog in the application header or user profile, or via the *AdChoices* icon on any active canvas ad.
2. **Right to Erasure / "To Be Forgotten" (GDPR Art. 17)**: You may permanently delete your account, authentication records, and associated knowledge graphs at any time directly through the *Profile Settings* dialog.
3. **Right to Data Portability (GDPR Art. 20)**: Knowledge graphs can be exported as structured JSON or serialized research narratives.
4. **Right to Restriction of Processing (GDPR Art. 18)**: You can opt out of personalized ad targeting by selecting *Contextual & Essential Only* or *Decline (Demo Mode)*.

---

## 5. Automated Data Retention & Archival

- **Inactive Knowledge Graphs**: Unattached graphs inactive for over 90 days are transitioned to cold storage and scheduled for archival with explicit user notifications.
- **Telemetry Pruning**: Aggregated ad viewability and crawl telemetry pulses are pruned automatically after 90 days.
- **Session Tokens**: Unused session records expire after 90 days and are purged during routine database maintenance sweeps.

---

## 6. How to Manage Your Preferences

You can manage your consent preferences at any time:
- Click **Account & Preferences** in the top navigation bar, then select **Privacy & Cookies**.
- Open **Profile Settings** and select **Privacy & Cookie Policy**.
- Click the **AdChoices** icon on any active canvas ad node.

---

## 7. Safe Contextual Advertising & Predefined Topic Taxonomy

To deliver relevant advertisements without compromising user confidentiality or violating strict privacy mandates:

1. **Zero Raw Document Exposure**: Neither your document text, uploaded PDFs, private notes, graph labels, nor personal queries are ever transmitted to Google or third-party ad networks.
2. **Predefined Safe Taxonomy Whitelist**: During background ingestion, our internal semantic matching models (TEI cross-encoders) map extracted conceptual themes against a curated, publicly vetted list of high-level academic and technical domains (e.g., Artificial Intelligence, Cloud Computing, Cybersecurity, Biotechnology).
3. **Aggregated Broad Slugs Only**: Only the non-sensitive slugs of matched broad categories are used to select contextual sponsor messages or configure Google Ad Manager/GPT channel targeting.
4. **Consent Respect**:
   - Users selecting **Contextual & Essential Only** receive ads matched strictly to these broad taxonomy tags with non-personalized ad parameters (`npa=1`) and no cross-site profiling cookies.
   - Users selecting **Decline (Demo Mode)** have external ad scripts disabled entirely.

