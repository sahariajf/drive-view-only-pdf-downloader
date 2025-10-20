# 🚀 Drive View Only PDF Downloader

Easily **download high-quality PDFs from “View-Only” Google Drive files** or any webpage showing pages as images — without installing anything.  
Just copy-paste or use a bookmarklet — and your PDF will be ready in seconds!

---

## 🧠 Before You Start

Please follow these important steps carefully:

✅ **Step 1:** Open your **Google Drive "View Only" document** (like a scanned book, notes, or preview file).  
✅ **Step 2:** **Scroll down and load all pages** you want included — make sure every page image is fully visible.  
✅ **Step 3:** Open your browser **Developer Console**:
- Windows → `Ctrl + Shift + J`
- Mac → `Cmd + Option + J`
✅ **Step 4:** Copy and paste the full code (below) into the console and press **Enter**.  
✅ **Step 5:** When prompted, type your desired page numbers (e.g. `1,3,5-8`).  
✅ **Step 6:** Wait a few seconds — your **high-quality PDF** will download automatically!

---

## 💻 Copy-Paste Version

> 📋 Double-click the box below → Copy → Paste into browser console.

```javascript
(function () {
  console.log("Loading script ...");

  // Prompt user for page numbers (1,3-5 etc.)
  let input = prompt("Enter page numbers to download (e.g., 1,3,5-7):", "1");
  if (!input) {
    console.log("No pages entered. Exiting...");
    return;
  }

  // Parse user input (1-based → 0-based)
  function parsePages(inputStr) {
    let pages = [];
    let parts = inputStr.split(",");
    parts.forEach((part) => {
      if (part.includes("-")) {
        let [start, end] = part.split("-").map((n) => parseInt(n.trim(), 10));
        for (let i = start; i <= end; i++) pages.push(i);
      } else {
        pages.push(parseInt(part.trim(), 10));
      }
    });
    return pages.filter((n) => !isNaN(n) && n > 0).map((n) => n - 1);
  }

  const selectedPages = parsePages(input);
  console.log("Selected pages:", selectedPages.map((x) => x + 1));

  let script = document.createElement("script");
  script.onload = function () {
    const { jsPDF } = window.jspdf;
    let pdf = null;
    let imgElements = document.getElementsByTagName("img");
    let validImgs = [];
    let initPDF = true;

    console.log("Scanning content ...");
    for (let i = 0; i < imgElements.length; i++) {
      let img = imgElements[i];
      // ✅ Allow blob or googleusercontent images
      if (!img.src.startsWith("blob:") && !img.src.includes("googleusercontent")) {
        continue;
      }
      validImgs.push(img);
    }

    console.log(`${validImgs.length} total pages found.`);

    // Filter only selected ones
    let imgsToProcess = selectedPages.map((idx) => validImgs[idx]).filter((img) => img);
    if (imgsToProcess.length === 0) {
      console.log("No valid pages found for the given numbers.");
      return;
    }

    console.log(`Processing ${imgsToProcess.length} selected pages...`);

    for (let i = 0; i < imgsToProcess.length; i++) {
      let img = imgsToProcess[i];
      let canvasElement = document.createElement("canvas");
      let con = canvasElement.getContext("2d");

      // ✅ High-quality 2× rendering
      let scale = 2;
      canvasElement.width = img.naturalWidth * scale;
      canvasElement.height = img.naturalHeight * scale;
      con.scale(scale, scale);
      con.drawImage(img, 0, 0, img.naturalWidth, img.naturalHeight);

      // ✅ Export highest-quality PNG
      let imgData = canvasElement.toDataURL("image/png", 1.0);

      let orientation = img.naturalWidth > img.naturalHeight ? "l" : "p";
      let pageWidth = img.naturalWidth;
      let pageHeight = img.naturalHeight;

      if (initPDF) {
        pdf = new jsPDF({
          orientation: orientation,
          unit: "px",
          format: [pageWidth, pageHeight],
          compress: false,
        });
        initPDF = false;
      }

      pdf.addImage(imgData, "PNG", 0, 0, pageWidth, pageHeight, "", "FAST");
      if (i !== imgsToProcess.length - 1) pdf.addPage();

      const percent = Math.floor(((i + 1) / imgsToProcess.length) * 100);
      console.log(`Progress: ${percent}%`);
    }

    let title =
      document.querySelector('meta[itemprop="name"]')?.content || "download";
    if (!title.endsWith(".pdf")) title += ".pdf";

    console.log("Downloading high-quality PDF ...");
    pdf.save(title, { returnPromise: true, compress: false }).then(() => {
      document.body.removeChild(script);
      console.log("✅ High-quality PDF downloaded successfully!");
    });
  };

  // ✅ Load jsPDF library securely
  let scriptURL = "https://unpkg.com/jspdf@latest/dist/jspdf.umd.min.js";
  let trustedURL;
  if (window.trustedTypes && trustedTypes.createPolicy) {
    const policy = trustedTypes.createPolicy("myPolicy", {
      createScriptURL: (input) => input,
    });
    trustedURL = policy.createScriptURL(scriptURL);
  } else {
    trustedURL = scriptURL;
  }

  script.src = trustedURL;
  document.body.appendChild(script);
})();
```
---

## ⚡ Features

✨ **2× High-Resolution Output** — captures and renders images at double quality  
✨ **Supports Google Drive “View Only” Files** — works directly on previewed images  
✨ **Custom Page Selection** — choose exactly which pages to include (e.g., `1,3,5-7`)  
✨ **Automatic Orientation Detection** — handles portrait and landscape pages  
✨ **Completely Client-Side** — runs fully in your browser, no uploads or tracking  
✨ **Lightweight** — under 10KB, no installation or dependencies  
✨ **Cross-Browser** — tested on Chrome, Edge, Brave, and Firefox  

---
---

## ⚠️ Important Notes

🚫 Make sure **all pages are fully loaded** before running the script (scroll to the bottom).  
🚫 Do **not switch tabs** or reload while it’s generating.  
🚫 Works best for **image-based pages**, not text-rendered PDFs.  
✅ For long documents, process in small batches (e.g., pages 1–50, then 51–100).  
✅ Works offline once the images are loaded in memory.

---

## 💡 Example Usage

1. Open your Google Drive view-only document.  
2. Scroll to load all desired pages completely.  
3. Open **Developer Console** (`Ctrl+Shift+J` or `Cmd+Option+J`).  
4. Paste the code and hit **Enter**.  
5. Input your page range (e.g., `1,3,5-8`).  
6. Wait a few seconds — ✅ your PDF will download automatically!

---

## 🪪 License

**MIT License**

This script is open-source and free for everyone.  
You are allowed to use, modify, and distribute it for both personal and commercial projects.  
Attribution to the author is appreciated but not required.

---

## ✨ Credits

Created with ❤️ by **Sahariaj**  
© 2025 — *Drive View Only PDF Downloader*

---

## 🏷️ Repo Badges

![Static Badge](https://img.shields.io/badge/Script-Copy%20%26%20Paste%20Friendly-blue)
![Static Badge](https://img.shields.io/badge/Client%20Side-100%25-green)
![Static Badge](https://img.shields.io/badge/License-MIT-orange)
![Static Badge](https://img.shields.io/badge/Made%20with-%E2%9D%A4-red)

---

## ⭐ Support

If this project helped you, please **⭐ Star this repository** to show your support!  
Pull requests, ideas, and improvements are always welcome — let’s make it even better 🚀

