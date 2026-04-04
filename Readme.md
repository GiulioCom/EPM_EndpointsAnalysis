## Executive Summary

The **EPM Endpoints Analysis Tool** is a high-performance, zero-footprint utility designed for CyberArk Endpoint Privilege Management (EPM) administrators. It provides a specialized environment to reconcile two critical report types: **MyComputer** and **Endpoints**. 

Unlike bulky spreadsheet processors, this tool executes entirely within the client's browser using vanilla JavaScript. It is architected to handle large CSV datasets with minimal memory overhead, providing instant insights into agent version readiness for migrations, identifying synchronization gaps between views, and flagging duplicate hardware or agent identifiers.

---

# EPM Endpoints Analysis & Reconciliation Tool

A lightweight, secure, and efficient analytical dashboard for CyberArk EPM report auditing.

## 🚀 Key Features

* **Dual-Report Integration:** Seamlessly switch between and compare "Endpoints" and "MyComputer" CSV exports.
* **Version Readiness Heuristics:** Automatically categorizes agent versions. Versions $\ge$ **25.12** are flagged as "Ready" (Green), while older versions are marked for update (Red).
* **Advanced Duplicate Detection:** Identifies collisions and redundancies based on three distinct vectors:
    * Computer Name
    * Agent ID
    * New Agent ID
* **Deep Reconciliation View:** A dedicated "Comparison View" that highlights "Ghost" endpoints (present in one report but missing in the other) and calculates version deltas.
* **Performance-Optimized UI:** Utilizes `IntersectionObserver` for "Lazy Rendering" of table rows, ensuring the UI remains responsive even when processing thousands of records.

---

## 🛠 Usage Instructions

1.  **Export Data:** Export the "Endpoints" and "MyComputer" reports from your CyberArk EPM console as **CSV** files.
2.  **Load Endpoints:** In the **Endpoints** tab, click **UPLOAD CSV** and select your Endpoints report.
3.  **Load MyComputer:** Switch to the **MyComputer** tab, click **UPLOAD CSV**, and select your MyComputer report.
4.  **Analyze:**
    * Check the **Stats Cards** for duplicate counts.
    * Review the **Versions** table to see the percentage of the fleet requiring updates.
    * Navigate to **Comparison View** to identify discrepancies between the two datasets.

---

## 🧠 Technical Deep Dive

### 1. Performance & Memory Management
In the tradition of old-school systems programming, this tool avoids "framework bloat."
* **Vanilla Parser:** Instead of importing heavy libraries like PapaParse, we use a custom, linear CSV parser that maps headers dynamically to minimize CPU cycles.
* **DOM Recycling:** The implementation uses a document fragment pattern for table rendering. For large datasets, it employs a **Sentinel-based Lazy Loader**. This prevents the browser from choking on thousands of DOM elements by only rendering chunks of 100 rows as the user scrolls.

### 2. Security & Data Privacy
As a Senior Architect, I prioritize the "Air-Gap" philosophy for sensitive infrastructure data:
* **Zero Data Egress:** This tool is a static HTML file. **No data is uploaded to a server.** All parsing, filtering, and reconciliation happen within the local browser's memory (RAM).
* **Input Sanitization:** Every cell is processed as a string literal to prevent basic XSS (Cross-Site Scripting) if a CSV contains malicious payloads.
* **Exclusion Logic:** The tool explicitly handles and ignores null/zero GUIDs (`00000000-0000...`) to prevent false positive duplicate flagging for uninitialized agents.

### 3. Comparison Logic
The "Comparison View" performs a **Full Outer Join** equivalent across three primary keys (Computer, Agent ID, New Agent ID). If a record in report A lacks a match in report B across *any* of these identifiers, it is flagged with a visual "Mismatch" warning (yellow cells), allowing administrators to identify sync issues within the EPM database itself.

---

## 📊 Version Compatibility Matrix
The tool applies the following logic for migration readiness:

| Version Range | Status | UI Color |
| :--- | :--- | :--- |
| $\ge$ 25.12 | **Endpoints Ready** | Success (Green) |
| $<$ 25.12 | **Update Required** | Danger (Red) |
| Unknown / N/A | **Review Needed** | Gray |
