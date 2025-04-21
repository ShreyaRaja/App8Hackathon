<!DOCTYPE html>
<html lang="en">

<!-- The following code has been developed by students and/or researchers of the Freshman Research Initiative, DIY Diagnostics Stream at The University of Texas at Austin. This code is shared for demonstration purposes and should not be considered a product -- it is for entertainment purposes only. Any user of this code does so at their own risk. Members of the DIY Stream, FRI, and The University of Texas system are not liable for anything related to this code.

  THIS CODE SHOULD NOT BE USED TO DIAGNOSE ANY KIND OF MEDICAL CONDITION.

  Further Information:
  http://cns.utexas.edu/fri

  Research Educator:
  Timothy E. Riedel
  triedel@utexas.edu

  Authors in chronological order of contribution:
  Author 1: Timothy E. Riedel
  Author 2: Workblock 1
-->

<head>
  <meta charset="utf-8">
  <title>Photo Transform</title>
  <meta name="viewport" content="width=device-width, initial-scale=.7">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <link rel="stylesheet" href="https://code.jquery.com/mobile/1.3.2/jquery.mobile-1.3.2.min.css" />
  <script src="https://code.jquery.com/jquery-1.9.1.min.js"></script>
  <script src="https://code.jquery.com/mobile/1.3.2/jquery.mobile-1.3.2.min.js"></script>

  <script>
    var docMod = document.lastModified; // gets last modified date and time of the index.html file
    console.log("This file last modified  " + docMod); // displays last modified date and time in the JavaScript browser console

    // Fix for iPhone photo squish bug
    function detectVerticalSquash(img) {
      var iw = img.naturalWidth, ih = img.naturalHeight;
      var canvas = document.createElement('canvas');
      canvas.width = 2;
      canvas.height = ih;
      var ctx = canvas.getContext('2d');
      ctx.drawImage(img, 0, 0);
      var data = ctx.getImageData(0, 0, 1, ih).data;
      var sy = 0, ey = ih, py = ih;
      while (py > sy) {
        var alpha = data[(py - 1) * 4 + 3];
        if (alpha === 0) {
          ey = py;
        } else {
          sy = py;
        }
        py = (ey + sy) >> 1;
      }
      var ratio = (py / ih);
      return (ratio === 0) ? 1 : ratio;
    }

    function drawImageIOSFix(ctx, img, sx, sy, sw, sh, dx, dy, dw, dh) {
      var vertSquashRatio = detectVerticalSquash(img);
      ctx.drawImage(img, sx, sy, sw, sh, dx, dy, dw, dh / vertSquashRatio);
    }

    function showResult(message) {
      const resultDiv = document.getElementById('result');
      resultDiv.textContent = message;
      resultDiv.style.fontSize = '24px';
      resultDiv.style.fontWeight = 'bold';
    }

    // window.onload necessary to keep JavaScripts from running before the app loads entirely
    window.onload = function () {
      canvas = document.getElementById('myCanvas');
      context = canvas.getContext("2d");

      var fileInput = document.getElementById('fileInput');
      var messageDisplayArea = document.getElementById('messageDisplayArea');
      var uploadedImage = null;

      function colorDistance(r1, g1, b1, r2, g2, b2) {
        return Math.sqrt((r1 - r2) ** 2 + (g1 - g2) ** 2 + (b1 - b2) ** 2);
      }

      function isPink(r, g, b) {
        return colorDistance(r, g, b, 255, 105, 180) < 100;
      }

      function isYellow(r, g, b) {
        return colorDistance(r, g, b, 255, 255, 0) < 100;
      }

      function analyzeClickColor(x, y) {
        var imgData = context.getImageData(x - 10, y - 10, 20, 20);
        var pixels = imgData.data;
        let pinkPixels = 0;
        let yellowPixels = 0;

        for (let i = 0; i < pixels.length; i += 4) {
          let r = pixels[i];
          let g = pixels[i + 1];
          let b = pixels[i + 2];
          if (isPink(r, g, b)) pinkPixels++;
          else if (isYellow(r, g, b)) yellowPixels++;
        }

        messageDisplayArea.innerHTML += `<br>Analyzed click region: ${pinkPixels} pink pixels, ${yellowPixels} yellow pixels.`;

        if (pinkPixels > yellowPixels && pinkPixels > 20) {
          showResult("Positive - Color Detected: Pink");
        } else if (yellowPixels > pinkPixels && yellowPixels > 20) {
          showResult("Negative - Color Detected: Yellow");
        } else {
          showResult("No significant color detected.");
        }
      }

      canvas.addEventListener("click", function (evt) {
        var rect = canvas.getBoundingClientRect();
        var x = evt.clientX - rect.left;
        var y = evt.clientY - rect.top;
        analyzeClickColor(x, y);
      });

      fileInput.addEventListener('change', function () {
        var file = fileInput.files[0];
        var imageType = /image.*/;

        if (file.type.match(imageType)) {
          var reader = new FileReader();
          reader.onload = function (e) {
            messageDisplayArea.innerHTML = "You picked an image! Click a tube to analyze its color.";
            var img = new Image();

            img.onload = function () {
              uploadedImage = img;
              drawImageIOSFix(context, img, 0, 0, img.naturalWidth, img.naturalHeight, 0, 0, canvas.width, canvas.height);
            };
            img.src = reader.result;
          };

          reader.readAsDataURL(file);
        } else {
          messageDisplayArea.innerHTML = "File not supported!";
        }
      });
    };
  </script>
</head>

<body>
  <div data-role="page">
    <div data-role="header">
      <h1>Workblock Hackathon!</h1>
    </div><!-- /header -->

    <div data-role="content">
      <h2>Push button to take picture.</h2>
      <div>
        Press orange button to create or choose image file:
        <input type="file" id="fileInput" style="background-color: #FF6600;">
      </div>
      <div id="messageDisplayArea"></div>
      <div id="result"></div>

      <canvas id="myCanvas" width="600" height="500" style="border:1px solid #d3d3d3;"></canvas>
      <script>
        context.font = '20pt Calibri';
        context.fillStyle = "black";
        context.fillText("Original image will go here", 10, 20);
      </script>
    </div><!-- /content -->

    <div data-role="footer" data-position="fixed">
      <p>DIY Diagnostics</p>
    </div><!-- /footer -->
  </div><!-- /page -->
</body>

</html>
