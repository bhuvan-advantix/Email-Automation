
# **Automated Email Workflow – Explained for Beginners**

---

## **1. What This Workflow Does (Plain Language)**

1. You keep a **Google Sheet** with emails to send:

   * Columns: Email, FirstName, Subject, Body, Attachments, Status, SendAt, OptOut
2. The workflow **checks the sheet** for rows ready to send.
3. It **downloads attachments** (if any) from Google Drive.
4. It **sends emails via Gmail**.
5. Updates the Google Sheet with:

   * Status = “Sent”
   * SendAt = time of sending
   * Output = “Sent successfully”
6. Skips rows:

   * Missing email
   * Status not “Pending”
   * Opted out
   * Scheduled for the future

---

## **2. How the Workflow is Triggered**

* **Manual Trigger** → Click “Execute workflow” in n8n to run immediately.
* **Schedule Trigger** → Run automatically at set intervals (e.g., every hour/day).

---

## **3. Step-by-Step Nodes Explanation**

### **Step 1: Get rows from Google Sheet**

**Node:** `Get row(s) in sheet`

* Reads your sheet with all the emails.
* Pulls data like Email, FirstName, Subject, Body, Attachments, etc.

---

### **Step 2: Filter rows and personalize**

**Node:** `Code in JavaScript`

What it does:

* Filters rows to **send only pending emails**.
* Skips:

  * No email
  * Status ≠ Pending
  * OptOut = Yes
  * SendAt in the future
* Replaces `{{FirstName}}` in Subject or Body with the actual name.
* Normalizes CC/BCC fields (empty if missing).
* Keeps attachment paths for sending.

**Result:** Only rows ready to send proceed to next step.

---

### **Step 3: Check for attachments**

**Node:** `If`

* If `Attachments` column is **not empty**, go to `Download file`.
* If empty, skip to `Send a message1`.

---

### **Step 4a: Download attachment**

**Node:** `Download file`

* Downloads the file from **Google Drive** using the URL in the Attachments column.
* Prepares it for Gmail to attach.

---

### **Step 4b: Send email with attachment**

**Node:** `Send a message`

* Sends email to recipient:

  * To = Email column
  * Subject = Subject column
  * Message = Body column
  * CC/BCC = From sheet if available
  * Attachment = File downloaded from Google Drive

---

### **Step 4c: Send email without attachment**

**Node:** `Send a message1`

* Sends email exactly like above but **no attachments**.

---

### **Step 5: Update Google Sheet**

**Node:** `Update row in sheet`

* Marks each row **Sent** in Status column.
* Sets SendAt = current time.
* Output = “Sent successfully”
* Updates by matching `row_number` (so correct row is updated).

---

## **4. Workflow Flow Diagram (Simple)**

```
[Manual Trigger / Schedule Trigger] 
       │
       ▼
 [Get row(s) in sheet] 
       │
       ▼
 [Code in JavaScript → Filter & personalize] 
       │
       ▼
      [If Attachments?]
     ┌───────┴───────┐
     │               │
[Download file]    [Send a message1]
     │               │
     ▼               ▼
[Send a message]   [Update row in sheet]
     │
     ▼
[Update row in sheet]
```

---

## **5. Key Points / Tips**

* **Attachments** must be stored in **Google Drive**; provide URL in the sheet.
* **Personalization** works with `{{FirstName}}` in Subject/Body.
* **Future scheduling** works with `SendAt` column. Emails won’t send before that time.
* **Opt-out** works with `OptOut = Yes`.
* **Gmail account** credentials must have proper OAuth access for n8n.

---

✅ **Outcome:**

* Emails are sent automatically or manually.
* Personalized with recipient name.
* Attachments included if specified.
* Google Sheet is updated to track sent emails.

---

