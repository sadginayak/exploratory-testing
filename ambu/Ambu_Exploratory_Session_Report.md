# Exploratory Testing Session Report
*SBTM-Style Session — Ambu Product Demo / Sign-Up Workflow*

## 1. Session Metadata

| Field | Details |
|---|---|
| **Tester** | Sadgi Nayak (independent / non-Ambu-employee tester) |
| **Session Duration** | 90 minutes |
| **Date** | 07 September 2026 |
| **Application Under Test** | Ambu.com — Product live-demo / sign-up workflow (Endoscopy > Pulmonology > Ambu aScope 5 Broncho / aScope 4a Broncho) |
| **Environments Used** | Default browser; Chrome, Microsoft Edge (cross-browser check) |
| **Session Type** | Session-Based Test Management (SBTM) — single charter, exploratory |

## 2. Charter

**Mission**
Explore the Ambu product live-demo / sign-up workflow.

**Areas to Explore**
- Product discovery and navigation
- Demo / sign-up forms
- Forms discrepancy and validation
- Country and product availability
- Submission and email confirmation

**Oracles**
- UI behaviour
- Field validation
- Product availability information
- Confirmation messages / emails

## 3. Navigation Path Observed

- Go to Endoscopy > Pulmonology (default selected).
- Select Products > Ambu aScope 5 Broncho > Learn more.
- Product page: "Contact us" > "Contact us to Arrange a demo" > form to set up a trial.
- Further down the same page: "Go to product" opens a new window.
- New page provides "Try now" and "Get Quote".
- "Try now" > "Sign up here" opens a second, separate sign-up form.
- The two forms are similar but not identical (see Section 4).

## 4. Categorized Findings

| Category | Observation / Finding | Severity |
|---|---|---|
| **Form Discrepancy** | Contact Us and Sign Up forms overlap in purpose but differ in fields: Contact Us has a "live demo at my convenience" checkbox and no clinical-interest field; Sign Up has clinical-interest but no checkbox. | Low / UX |
| **Field Validation** | Mandatory fields (Hospital, Job Role, Clinical Interest) correctly block submission when empty. However, once populated, garbage values pass unchecked: numeric names ("12"), single-character hospital ("S"), special-character job title ("#"), 35-character names. Validation is presence-only, not format/content-aware. | Medium |
| **Field Validation - Email** | Malformed email format only triggers a browser-level "must be valid" prompt for obviously invalid text. | Low |
| **Submission & Confirmation (Critical)** | After the first successful submission, all further submissions from the same email address — across different countries (Germany, Denmark) and different products (aScope 5 Broncho, aScope 4a Broncho) — show a success "Thank you" screen but no confirmation email is ever sent (spam folder checked). The UI gives no indication that the email failed to send. | High |
| **Root Cause Hypothesis** | Likely cause: the tester's email was unsubscribed after the first confirmation email, and the mailing platform appears to apply a global suppression flag per email address, silently blocking all future sends regardless of country or product. Not yet confirmed with a fresh, never-unsubscribed email. | High (unconfirmed) |
| **Data Persistence** | Switching to a different product (aScope 4a Broncho) retained previously entered form data instead of resetting the Sign Up form. Risk of a user unknowingly submitting an enquiry for the wrong product or with stale data. | Medium |
| **Confirmation Email Content** | The one confirmation email that was received was generic and did not reference which product (aScope 5 Broncho) the enquiry was about, reducing traceability for both user and sales follow-up. | Low / UX |
| **Country & Product Availability** | The Sign Up country dropdown lists all countries, but "Find your local representative" shows a restricted list. Maldives is absent from the representative list (confirmed unavailable), yet is still selectable and submittable on the Sign Up form with no warning. | Medium |
| **Navigation** | Back button after the "Thank you" screen correctly returns to the Try Demo page with no resubmission or error — no defect found here. | Info |


## 6. Blindspots / Recommended Follow-Up

- Confirmatory test with a fresh, never-unsubscribed email across product/country to isolate suppression-list vs. systemic dedup bug.
- Field boundary / maximum-length limits on name, hospital, and job-title fields (actual cap not yet found).
- Injection and Unicode/emoji input in free-text fields (Hospital, Job Title, Clinical Interest).
- Data-context persistence when "Go to product" opens a new window/tab.
- Email content deep-dive: sender authenticity, links, and whether an unavailable-country submission would generate a different email template if delivery succeeded.

## 7. High-priority observation / suspected defect — requires confirmation with a fresh email address.

After a user's first successful enquiry, every subsequent enquiry from the same email address, regardless of country or product, displays a success "Thank you" screen but sends no confirmation email, with no warning to the user. Suspected cause is an email-unsubscribe / suppression flag being applied. This is a false-positive success state and is recommended as the priority defect for engineering follow-up, pending one confirmatory test with a fresh email address.
