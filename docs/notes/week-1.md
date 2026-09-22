<style>
  /* Make the main page container wider just for this page */
  .md-grid {
    max-width: 95% !important;
  }
</style>

# Week 1 Notes

<div style="text-align: right; margin-bottom: 10px;">
  <a href="../week-1.pdf" download class="md-button md-button--primary">⬇️ Download PDF</a>
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

var url = '../week-1.pdf';
var container = document.getElementById('pdf-container');
var loadingMsg = document.getElementById('loading-msg');

pdfjsLib.getDocument(url).promise.then(function(pdf) {
  loadingMsg.style.display = 'none';
  
  for (let pageNum = 1; pageNum <= pdf.numPages; pageNum++) {
    // Create canvas synchronously to maintain correct order
    let canvas = document.createElement('canvas');
    canvas.id = 'page-' + pageNum;
    container.appendChild(canvas);

    pdf.getPage(pageNum).then(function(page) {
      // Render at a high base scale so it stays sharp when zooming in
      let baseScale = 3.0;
      let viewport = page.getViewport({ scale: baseScale });
      
      // Fix blurriness on high-resolution/retina displays
      let outputScale = window.devicePixelRatio || 1;
      
      canvas.width = Math.floor(viewport.width * outputScale);
      canvas.height = Math.floor(viewport.height * outputScale);
      
      // Maintain proper CSS display size and aspect ratio
      canvas.style.width = "100%";
      canvas.style.maxWidth = Math.floor(page.getViewport({scale: 1.5}).width) + "px";
      canvas.style.height = "auto";
      
      let renderContext = {
        canvasContext: canvas.getContext('2d'),
        transform: [outputScale, 0, 0, outputScale, 0, 0],
        viewport: viewport
      };
      page.render(renderContext);
    });
  }
}).catch(function(err) {
  loadingMsg.innerHTML = 'Error loading PDF. <a href="' + url + '">Download PDF</a>';
});
</script>