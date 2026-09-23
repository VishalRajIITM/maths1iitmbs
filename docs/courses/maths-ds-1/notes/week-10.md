<style>
  /* Make the main page container wider just for this page */
  .md-grid {
    max-width: 95% !important;
  }
</style>

# Week 10 Notes

<div style="text-align: right; margin-bottom: 10px;">
  <a href="../week-10.pdf" download class="md-button md-button--primary">⬇️ Download PDF</a>
</div>

<div id="loading-msg">Loading PDF... Please wait.</div>
<div id="pdf-container"></div>

<style>
#pdf-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
}
#pdf-container canvas {
  max-width: 100%;
  margin-bottom: 20px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.15); /* elegant shadow */
  background-color: white;
}
#loading-msg {
  text-align: center;
  padding: 40px;
  font-size: 1.2em;
  color: #555;
}
</style>

<script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.16.105/pdf.min.js"></script>
<script>
pdfjsLib.GlobalWorkerOptions.workerSrc = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.16.105/pdf.worker.min.js';

var url = '../week-10.pdf';
var container = document.getElementById('pdf-container');
var loadingMsg = document.getElementById('loading-msg');
var pdfDoc = null;

// Render at extra resolution beyond the CSS display size so zooming in
// (pinch or browser zoom) stays sharp instead of upscaling a 1:1 raster.
// Lazy-loading (below) is what keeps this affordable: only a couple of
// pages are ever rasterized at once, so this can afford real headroom.
var displayScale = 1.5;
var zoomHeadroom = 2;
var renderScale = displayScale * Math.min(Math.max(window.devicePixelRatio || 1, 1) * zoomHeadroom, 3);

// Only rasterize a page once it's about to scroll into view, instead of
// rendering the whole document up front.
var observer = new IntersectionObserver(function(entries) {
  entries.forEach(function(entry) {
    if (entry.isIntersecting) {
      renderPage(entry.target);
      observer.unobserve(entry.target);
    }
  });
}, { rootMargin: '400px 0px' });

function renderPage(canvas) {
  var pageNum = parseInt(canvas.dataset.pageNum, 10);
  pdfDoc.getPage(pageNum).then(function(page) {
    var viewport = page.getViewport({ scale: renderScale });
    canvas.width = Math.floor(viewport.width);
    canvas.height = Math.floor(viewport.height);
    page.render({
      canvasContext: canvas.getContext('2d'),
      viewport: viewport
    });
  });
}

pdfjsLib.getDocument(url).promise.then(function(pdf) {
  pdfDoc = pdf;

  var pagePromises = [];
  for (let pageNum = 1; pageNum <= pdf.numPages; pageNum++) {
    pagePromises.push(pdf.getPage(pageNum));
  }

  Promise.all(pagePromises).then(function(pages) {
    loadingMsg.style.display = 'none';

    pages.forEach(function(page, idx) {
      let pageNum = idx + 1;
      let viewport = page.getViewport({ scale: displayScale });

      let canvas = document.createElement('canvas');
      canvas.id = 'page-' + pageNum;
      canvas.dataset.pageNum = pageNum;
      canvas.style.width = "100%";
      canvas.style.maxWidth = Math.floor(viewport.width) + "px";
      canvas.style.aspectRatio = viewport.width + ' / ' + viewport.height;
      canvas.style.height = "auto";
      canvas.style.backgroundColor = "#fff";

      container.appendChild(canvas);
      observer.observe(canvas);
    });
  });
}).catch(function(err) {
  loadingMsg.innerHTML = 'Error loading PDF. <a href="' + url + '">Download PDF</a>';
});
</script>