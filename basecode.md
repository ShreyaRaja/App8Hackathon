# App8Hackathon
Plan:

<!DOCTYPE html>
<html lang="en">

<!-- 
  Developed by the DIY Diagnostics Stream (FRI, UT Austin) for educational purposes.
  NOT FOR MEDICAL USE.
-->

<head>
  <meta charset="utf-8">
  <title>Tube Color Detection</title>
  <meta name="viewport" content="width=device-width, initial-scale=0.7">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <link rel="stylesheet" href="https://code.jquery.com/mobile/1.3.2/jquery.mobile-1.3.2.min.css" />
  <script src="https://code.jquery.com/jquery-1.9.1.min.js"></script>
  <script src="https://code.jquery.com/mobile/1.3.2/jquery.mobile-1.3.2.min.js"></script>

  <style>
    #result {
      font-size: 24px;
      font-weight: bold;
      margin-top: 10px;
    }
  </style>

  <script>
    // Fix iPhone squish bug when displaying images on canvas
    function detectVerticalSquash(img) {
      const ih = img.naturalHeight;
      const canvas = document.createElement('canvas');
      canvas.width = 2;
      canvas.height = ih;
      const ctx = canvas.getContext('2d');
      ctx.drawImage(img, 0, 0);
      const data = ctx.getImageData(0, 0, 1, ih).data;

      let sy = 0, ey = ih, py = ih;
      while (py > sy) {
        const alpha = data[(py - 1) * 4 + 3];
        if (alpha === 0) {
          ey = py;
        } else {
          sy = py;
        }
        py = (ey + sy) >> 1;
      }
      const ratio = (py / ih);
      return (ratio === 0) ? 1 : ratio;
    }

    function drawImageIOSFix(ctx, img, sx, sy, sw, sh, dx, dy, dw, dh) {
      const vertSquashRatio = detectVerticalSquash(img);
      ctx.drawImage(img, sx, sy, sw, sh, dx, dy, dw, dh / vertSquashRatio);
    }

    // Display result in the "result" div
    function showResult(message) {
      const resultDiv = document.getElementById('result');
      resultDiv.textContent = message;
    }

    // Wait for the page to finish loading
    window.onload = function () {
      const fileInput = document.getElementById('fileInput');
      const messageDisplayArea = document.getElementById('messageDisplayArea');

      fileInput.addEventListener('change', function () {
        const file = fileInput.files[0];
        const imageType = /image.*/;

        if (file.type.match(imageType)) {
          const reader = new FileReader();

          reader.onload = function () {
            messageDisplayArea.innerHTML = "You picked an image!";
            const img = new Image();

            img.onload = function () {
              // Draw image on canvas
              drawImageIOSFix(context, img, 0, 0, img.naturalWidth, img.naturalHeight, 0, 0, 600, 500);

              // ========== BEGIN TUBE COLOR DETECTION ==========
              const imgData = context.getImageData(0, 0, canvas.width, canvas.height);
              const pixels = imgData.data;

              const tubeWidth = 40; // Adjust if tubes are wider/narrower
              const rowY = 250; // Vertical slice across tube centers
              let pinkCount = 0;
              let yellowCount = 0;

              function isPink(r, g, b) {
                return r > 200 && g < 100 && b > 150;
              }

              function isYellow(r, g, b) {
                return r > 200 && g > 200 && b < 100;
              }

              for (let x = 0; x < canvas.width; x += tubeWidth) {
                let pinkPixels = 0;
                let yellowPixels = 0;

                for (let dx = 0; dx < tubeWidth; dx++) {
                  for (let dy = -10; dy < 10; dy++) {
                    const px = x + dx;
                    const py = rowY + dy;
                    const index = (py * canvas.width + px) * 4;
                    const r = pixels[index];
                    const g = pixels[index + 1];
                    const b = pixels[index + 2];

                    if (isPink(r, g, b)) pinkPixels++;
                    else if (isYellow(r, g, b)) yellowPixels++;
                  }
                }

                // If enough pixels are colored, count it as a tube
                if (pinkPixels > 50) pinkCount++;
                else if (yellowPixels > 50) yellowCount++;
              }

              // Output the result
              const resultMessage = `Detected ${pinkCount} pink tube(s) and ${yellowCount} yellow tube(s).`;
              messageDisplayArea.innerHTML += `<br>${resultMessage}`;

              // Final result output
              if (pinkCount > 0) {
                showResult("Positive - Color Detected: Pink");
              } else if (yellowCount > 0) {
                showResult("Negative - Color Detected: Yellow");
              } else {
                showResult("No color detected.");
              }
              // ========== END TUBE COLOR DETECTION ==========

            }; // end of img.onload

            img.src = reader.result;
          }; // end of reader.onload

          reader.readAsDataURL(file);
        } else {
          messageDisplayArea.innerHTML = "File not supported!";
        }
      }); // end of file input listener
    }; // end of window.onload
  </script>
</head>

<body>
  <div data-role="page">
    <div data-role="header">
      <h1>Workblock Hackathon!</h1>
    </div>

    <div data-role="content">
      <h2>Push button to take picture.</h2>

      <div>
        Press orange button to create or choose image file:
        <input type="file" id="fileInput" style="background-color: #FF6600;">
      </div>

      <div id="messageDisplayArea"></div>

      <canvas id="myCanvas" width="600" height="500" style="border:1px solid #d3d3d3;"></canvas>

      <script>
        // Draw initial placeholder text
        const canvas = document.getElementById('myCanvas');
        const context = canvas.getContext("2d");
        context.font = '20pt Calibri';
        context.fillStyle = "black";
        context.fillText("Original image will go here", 10, 20);
      </script>

      <!-- Where the result (positive/negative) is shown -->
      <div id="result"></div>
    </div>

    <div data-role="footer" data-position="fixed">
      <p>DIY Diagnostics</p>
    </div>
  </div>
</body>

</html>

