<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Drag-and-Drop Resume Builder</title>

  <style>
    :root {
      --bg: #f8f9fa;
      --card-bg: #ffffff;
      --accent: #2b7a78;      /* button color */
      --accent-2: #d9f0f7;    /* skill card background */
      --border: #cccccc;
      --border-soft: #e6e6e6;
      --text: #222;
    }

    * { box-sizing: border-box; }
    html, body { height: 100%; }
    body {
      font-family: Arial, Helvetica, sans-serif;
      margin: 0; padding: 0;
      color: var(--text);
      background-color: var(--bg);
    }

    h1 { text-align: center; margin: 20px 0; }
    h2 { margin: 0 0 10px 0; }
    h3 { margin: 0 0 8px 0; }

    .container {
      display: flex;
      justify-content: space-between;
      align-items: flex-start;
      padding: 20px;
      gap: 20px;
      flex-wrap: wrap;
    }

    .column {
      background: var(--card-bg);
      padding: 15px;
      border-radius: 8px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.08);
      flex: 1 1 420px; /* responsive: min width 420px */
      overflow: visible; /* important so snapshot sees all content */
    }

    /* Left column */
    .skills-list div {
      background-color: var(--accent-2);
      margin: 6px 0;
      padding: 10px;
      border-radius: 4px;
      cursor: grab;
      user-select: none;
    }
    .skills-list div:active { cursor: grabbing; }

    .custom-inputs input {
      width: 100%;
      margin: 6px 0;
      padding: 8px;
      border: 1px solid var(--border);
      border-radius: 4px;
      outline: none;
    }
    .custom-inputs input:focus {
      border-color: #77c0d0;
      box-shadow: 0 0 0 3px rgba(119,192,208,0.25);
    }

    .btn {
      display: inline-block;
      background-color: var(--accent);
      color: #fff;
      padding: 10px 15px;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      margin-top: 10px;
      font-weight: 600;
      letter-spacing: 0.2px;
    }
    .btn[disabled] {
      opacity: 0.6;
      cursor: not-allowed;
    }

    /* Right column (resume) */
    #exportArea { overflow: visible; }
    .resume-section {
      border: 2px dashed var(--border);
      margin-bottom: 15px;
      padding: 10px;
      border-radius: 6px;
      min-height: 54px;
      background: #fff;
    }
    .resume-section h3 {
      margin-top: 0;
      font-size: 1.05rem;
    }
    .resume-item {
      margin: 6px 0;
      padding: 8px 10px;
      background: #f4f7fb;
      border: 1px solid #e1e5ea;
      border-radius: 4px;
    }

    /* Subtle improvements for the exported PDF snapshot only */
    #exportArea.for-pdf, #exportArea.for-pdf * {
      box-shadow: none !important;
      text-shadow: none !important;
    }
    #exportArea.for-pdf .resume-section {
      border-color: var(--border-soft) !important;
    }

    /* Better page breaks if printing via browser as fallback */
    @media print {
      .no-print { display: none !important; }
      .page-break { page-break-before: always; break-before: page; }
    }

    /* Directions panel (optional) */
    .directions {
      background: #fff9db;
      border: 1px solid #ffe58f;
      color: #7a5900;
      padding: 12px 14px;
      margin: 10px 20px 0 20px;
      border-radius: 6px;
    }
  </style>

  <!-- ✅ Correct bundle: provides html2canvas + jsPDF -->
  <script defer src="https://cdn.jsdelivr.net/npm/html2pdf.js@0.10.1/dist/html2pdf.bundle.min.js"></script>
</head>
<body>
  <h1>Drag-and-Drop Resume Builder</h1>

  <div class="directions no-print">
    <strong>Directions:</strong> Press on the skill cards on the left and drag them
    to the Resume categories on the right. You may add skill cards by typing and pressing
    <em>Add Skills</em>. You are not required to use all skill cards. When finished,
    press <em>Print to PDF</em> and save the file, then upload to Edio.
  </div>

  <div class="container">
    <!-- Left column: skill cards + add custom -->
    <div class="column" aria-labelledby="skills-heading">
      <h2 id="skills-heading">Skill Cards</h2>
      <div class="skills-list" id="skillsList">
        <div draggable="true">Good at following directions</div>
        <div draggable="true">Works well with others</div>
        <div draggable="true">Pays attention to detail</div>
        <div draggable="true">Technology enthusiast</div>
        <div draggable="true">Writes neatly and clearly</div>
        <div draggable="true">Microsoft Office skills</div>
        <div draggable="true">Enjoys working out</div>
        <div draggable="true">Enjoys volunteering</div>
        <div draggable="true">Human personality</div>
        <div draggable="true">Likes to read</div>
        <div draggable="true">High school graduate</div>
        <div draggable="true">Honor Roll Student</div>
        <div draggable="true">Volunteered at Community Center</div>
        <div draggable="true">Helped with School Events</div>
      </div>

      <h3>Add Your Own Ideas</h3>
      <div class="custom-inputs">
        <input type="text" placeholder="Type your idea here" />
        <input type="text" placeholder="Type your idea here" />
        <input type="text" placeholder="Type your idea here" />
        <input type="text" placeholder="Type your idea here" />
        <button class="btn" type="button" onclick="addCustomSkills()">Add Skills</button>
      </div>
    </div>

    <!-- Right column: resume export area (Header removed as requested) -->
    <div class="column" id="exportArea" aria-labelledby="resume-heading">
      <h2 id="resume-heading">Your Resume</h2>

      <div class="resume-section" id="skills"><h3>Skills</h3></div>
      <div class="resume-section" id="interests"><h3>Interests</h3></div>
      <div class="resume-section" id="education"><h3>Education</h3></div>
      <div class="resume-section" id="volunteer"><h3>Volunteer Work</h3></div>

      <button class="btn no-print" id="downloadBtn" type="button" onclick="downloadPDF()">
        Print to PDF
      </button>
    </div>
  </div>

  <script>
    // ------- Drag-and-Drop: from left list -> to resume sections -------
    const skillsList = document.getElementById('skillsList');
    const sections = document.querySelectorAll('.resume-section');

    // Start dragging from the skills list
    skillsList.addEventListener('dragstart', (e) => {
      if (e.target && e.target.matches('div[draggable="true"]')) {
        e.dataTransfer.setData('text/plain', e.target.textContent);
      }
    });

    // Allow drop on each resume section
    sections.forEach((section) => {
      section.addEventListener('dragover', (e) => e.preventDefault());
      section.addEventListener('drop', (e) => {
        e.preventDefault();
        const text = e.dataTransfer.getData('text/plain');
        if (!text) return;

        const newItem = document.createElement('div');
        newItem.className = 'resume-item';
        newItem.textContent = text;

        section.appendChild(newItem);
        // Optional: reassure students the item landed
        newItem.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
      });
    });

    // ------- Add custom skills to the left list -------
    function addCustomSkills() {
      const inputs = document.querySelectorAll('.custom-inputs input');
      inputs.forEach((input) => {
        const value = (input.value || '').trim();
        if (value !== '') {
          const newSkill = document.createElement('div');
          newSkill.textContent = value;
          newSkill.setAttribute('draggable', 'true');
          skillsList.appendChild(newSkill);
          input.value = '';
        }
      });
    }

    // ------- Utility: wait for images (if any) inside an element -------
    function waitForImages(root) {
      const imgs = Array.from(root.querySelectorAll('img'));
      if (imgs.length === 0) return Promise.resolve();
      return Promise.all(
        imgs.map((img) => {
          if (img.complete) return Promise.resolve();
          return new Promise((res) => { img.onload = img.onerror = res; });
        })
      );
    }

    // ------- PDF: Snapshot + Multi-page Slicing with graceful fallback -------
    async function downloadPDF() {
      const exportEl = document.getElementById('exportArea');
      const btn = document.getElementById('downloadBtn');

      // Verify libraries provided by the bundle are available
      const hasH2C = typeof window.html2canvas !== 'undefined';
      const hasJsPDF = window.jspdf && window.jspdf.jsPDF;

      // Fallback: if bundle not available (e.g., CDN blocked), open print dialog
      if (!hasH2C || !hasJsPDF) {
        try {
          window.print();
          return;
        } catch {
          alert('PDF tools are blocked. Try reloading or a different network/browser.');
          return;
        }
      }

      // Disable button during generation
      const originalLabel = btn.textContent;
      btn.textContent = 'Generating PDF...';
      btn.disabled = true;

      // Ensure all dynamic content is fully painted
      exportEl.classList.add('for-pdf');
      await waitForImages(exportEl);
      if (document.fonts && document.fonts.ready) {
        try { await document.fonts.ready; } catch {}
      }
      await new Promise((r) => requestAnimationFrame(r));

      try {
        // High-quality canvas snapshot (scaled for clarity, capped for performance)
        const scale = Math.min(2, window.devicePixelRatio || 1.5);
        const canvas = await window.html2canvas(exportEl, {
          scale,
          backgroundColor: '#ffffff',
          useCORS: true,
          allowTaint: false,
          logging: false
        });

        const fullImgData = canvas.toDataURL('image/jpeg', 0.96);

        // Setup jsPDF (A4 portrait)
        const { jsPDF } = window.jspdf;
        const pdf = new jsPDF('p', 'mm', 'a4');

        // PDF sizes in mm
        const pageWidth = pdf.internal.pageSize.getWidth();    // ~210mm
        const pageHeight = pdf.internal.pageSize.getHeight();  // ~297mm
        const margin = 10;                                     // mm
        const usableWidth = pageWidth - margin * 2;
        const usableHeight = pageHeight - margin * 2;

        // Canvas sizes in px
        const imgWidthPx = canvas.width;
        const imgHeightPx = canvas.height;

        // Convert: pixels per mm based on fitting width
        const pxPerMm = imgWidthPx / usableWidth;
        const pageUsableHeightPx = Math.floor(usableHeight * pxPerMm);

        // If the snapshot fits on one page, draw once; else slice
        if (imgHeightPx <= pageUsableHeightPx) {
          const imgHeightMm = imgHeightPx / pxPerMm;
          pdf.addImage(fullImgData, 'JPEG', margin, margin, usableWidth, imgHeightMm);
        } else {
          let sY = 0;
          let pageIndex = 0;

          while (sY < imgHeightPx) {
            const sliceHeightPx = Math.min(pageUsableHeightPx, imgHeightPx - sY);

            // Create a canvas for the current slice
            const pageCanvas = document.createElement('canvas');
            pageCanvas.width = imgWidthPx;
            pageCanvas.height = sliceHeightPx;

            const ctx = pageCanvas.getContext('2d');
            ctx.imageSmoothingEnabled = true;
            ctx.imageSmoothingQuality = 'high';

            // Copy a slice from the full canvas
            ctx.drawImage(
              canvas,
              0, sY, imgWidthPx, sliceHeightPx,  // source rectangle in full canvas
              0, 0, imgWidthPx, sliceHeightPx    // destination rectangle in page canvas
            );

            const pageImgData = pageCanvas.toDataURL('image/jpeg', 0.96);
            const sliceHeightMm = sliceHeightPx / pxPerMm;

            if (pageIndex > 0) pdf.addPage();
