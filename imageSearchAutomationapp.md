Product Image Search Automation

Summary
### Project Description

I am looking for an experienced automation specialist to build a complete and reliable workflow for product image discovery with an approval and retry mechanism.

I have a list of products where each row contains only the product name.
The goal is to automatically search for a suitable product image, save the result, and allow manual approval or re-request before final acceptance.

---

### Project Goal

The main objective is to automate product image searching while keeping human control over the final decision.

The system should:

* Search for a product image automatically
* Save the image result
* Allow approval or retry until the correct image is found

---

### Workflow Requirements

The workflow should follow this logic:

1. Input a product name (from Google Sheets).
2. Automatically search the internet for a suitable product image.
3. Select an image that meets the following criteria:

   * Product-only image (no people)
   * Clean or white background
   * High quality and clear resolution
   * No watermark, text, or promotional overlays
4. Return ONLY a direct image URL (must end with .jpg or .png).
5. Save the image URL into Google Sheets with a status of "pending".
6. Allow manual review with two possible actions:

   * **Approve** → the image is accepted and the workflow stops.
   * **Retry** → the system searches again and replaces the image.
7. Track the number of attempts and optionally stop after a defined limit.

---

### Google Sheets Structure

The workflow should use or create the following fields:

* product_name
* image_url
* status (pending / approved / retry)
* attempts
* notes (optional)

---

### Technical Requirements

* The solution must be built using Python, Crawlee, React, FastAPI.
* The workflow must be stable, reusable, and easy to maintain.
* Image quality and correctness are more important than speed.

---

### Deliverables

* CSV file with product name and image url
* Clear explanation of the workflow logic
* Instructions on how to approve, retry, and manage results
* Optional recommendations for future expansion

---
