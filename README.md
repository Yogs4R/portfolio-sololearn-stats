# 🤖 SoloLearn Stats Automation with n8n & Supabase

![n8n](https://img.shields.io/badge/Workflow-n8n-ff6d5a?style=for-the-badge&logo=n8n)
![Supabase](https://img.shields.io/badge/Database-Supabase-3ecf8e?style=for-the-badge&logo=supabase)
![SoloLearn](https://img.shields.io/badge/Platform-SoloLearn-1F90FF?style=for-the-badge&logo=sololearn&logoColor=white)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Website](https://img.shields.io/badge/Website-My%20Portfolio-blue?style=flat&logo=google-chrome)](https://ridwansuryantara.netlify.app)

**Automated tracking of SoloLearn learning progress including XP, levels, and certificates earned using n8n with data stored in Supabase.**

## 📖 Overview

This repository hosts an n8n workflow designed to track your coding journey on SoloLearn. Unlike standard public APIs, this workflow connects to SoloLearn's internal API (mimicking a browser request) to fetch detailed profile stats and archives the data in Supabase for your personal dashboard.

## 📸 Workflow Preview

![Workflow Overview](./workflow-image-sololearn.png)

## ✨ Features

* **Automated Scheduling:** Runs automatically every 1 hour or on app start.
* **Internal API Integration:** Uses specific HTTP Headers and cURL import logic to access SoloLearn's rich user data.
* **Data Aggregation:** Extracts key metrics using JavaScript:
    * ⚡ **XP & Level:** Tracks your current experience points and user level.
    * 📜 **Certificates:** Counts the total number of completed course certificates.
* **Data Storage:** Processes and updates the corresponding row in **Supabase**.

## 🛠️ Tech Stack

* **Automation:** [n8n](https://n8n.io/) (Self-hosted/Cloud)
* **Database:** [Supabase](https://supabase.com/) (PostgreSQL)
* **Source:** SoloLearn Internal API v2

## 🚀 How to Use

### 1. Import Workflow
* Download the `.json` file from this repository.
* Open your n8n editor.
* Click the menu (top right) and select `Import from File`.

### 2. Configure Nodes

Since this workflow uses an internal API, you need to extract your specific headers (Token) from your browser.

#### ⏰ Schedule (Every 1 Hour)
* Set the **Trigger Interval** (e.g., Every 1 Hour) based on how often you want to track data.

#### 🌐 Get Stats (SoloLearn API)
* **Method:** `GET`
* **URL:** Replace the numeric ID at the end with your **SoloLearn User ID**.
    ```text
    https://api2.sololearn.com/v2/userinfo/v3/profile/YOUR_USER_ID
    ```
    *Example:* `https://api2.sololearn.com/v2/userinfo/v3/profile/34686483`
* **Query Parameters:**
    * `sections`: `1,3,7,8` (This tells the API which data chunks to return).
* **Headers (Crucial):**
    * You must include an **Authorization** header. To get this:
        1. Login to SoloLearn.com in your browser.
        2. Open Developer Tools (F12) -> **Network** tab.
        3. Refresh the profile page.
        4. Find a request to `profile` or `userinfo`.
        5. Copy the `authorization` value (starts with `Bearer ...`).
    * **Add these Headers in n8n:**
        * `authorization`: `Bearer YOUR_LONG_TOKEN_HERE`
        * `user-agent`: `Mozilla/5.0...` (Copy from browser to avoid blocking).
        * `origin`: `https://www.sololearn.com`
        * `referer`: `https://www.sololearn.com/`

#### 🧮 Calculate
* This node parses the complex JSON response.
* It extracts:
    * `xp`: From `userDetails.xp`
    * `level`: From `userDetails.level`
    * `certCount`: Counts the length of the `certificates` array.

#### 💾 Update a row Supabase
* **Operation:** `Update`
* **Table:** `platform_stats` (or your specific table name).
* **Authentication:**
    * Select **Predefined Credential Type** -> **Supabase API**.
    * Select your credentials.
* **Update Condition:**
    * The workflow is set to update a row where `platform` equals `'sololearn'`.
    * *Important:* Ensure you have a row in your database with the column `platform` set to `sololearn` before running this, or switch the operation to "Upsert".

### 3. Activate
* Toggle the workflow to **Active** in the top right corner.

## 📝 Data Structure

The workflow prepares a JSON object similar to this before sending to Supabase:

```json
{
  "xp": 2500,
  "level": 12,
  "certCount": 5,
  "platform": "sololearn"
}
