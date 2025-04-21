<!DOCTYPE html> <!-- Declares the document type and version of HTML -->
<html lang="en"> <!-- Starts the HTML document and sets the language to English -->

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
  <meta charset="utf-8"> <!-- Sets character encoding to UTF-8 for proper text rendering -->
  <title>Photo Transform</title> <!-- Sets the title of the web page (shown in browser tab) -->
  <meta name="viewport" content="width=device-width, initial-scale=.7"> <!-- Responsive scaling for mobile devices -->
  <meta name="apple-mobile-web-app-capable" content="yes"> <!-- Allows iOS devices to run the app in full-screen -->

  <!-- Include jQuery Mobile CSS and JS libraries for styling and mobile UI support -->
  <link rel="stylesheet" href="https://code.jquery.com/mobile/1.3.2/jquery.mobile-1.3.2.min.css" />
  <script src="https://code.jquery.com/jquery-1.9.1.min.js"></script>
  <script src="https://code.jquery.com/mobile/1.3.2/jquery.mobile-1.3.2.min.js"></script>

  <script>
    var docMod = document.lastModified; // gets last modified date and time of the index.html file
    console.log("This file last modified  " + docMod); // displays last modified date and time in the JavaScript browser console

    // Fix for iPhone photo squish bug
    function detectVerticalSquash(img) {
      var iw = img.naturalWidth, ih = img.naturalHeight; //Get's images natural dimensions
      var canvas = document.createElement('canvas'); // Create temporary canvas
      canvas.width = 2; // Narrow canvas for sampling
      canvas.height = ih;
      var ctx = canvas.getContext('2d');
      ctx.drawImage(img, 0, 0); // Draw the image to the canvas
      var data = ctx.getImageData(0, 0, 1, ih).data; // Get pixel data from a 1-pixel-wide column

      //Binary search for non-transparent pixel from the bottom
      var sy = 0, ey = ih, py = ih;
      while (py > sy) {
        var alpha = data[(py - 1) * 4 + 3]; // Alpha value of current pixel
        if (alpha === 0) {
          ey = py;
        } else {
          sy = py;
        }
        py = (ey + sy) >> 1; // Midpoint
      }
      var ratio = (py / ih);
      return (ratio === 0) ? 1 : ratio; // Return squash ration, default to 1
    }

    // Draws image onto canvas while compensating for iOS vertical squash
    function drawImageIOSFix(ctx, img, sx, sy, sw, sh, dx, dy, dw, dh) {
      var vertSquashRatio = detectVerticalSquash(img);
      ctx.drawImage(img, sx, sy, sw, sh, dx, dy, dw, dh / vertSquashRatio);
    }

    //Displays a bold, large-font result message on the screen
    function showResult(message) {
      const resultDiv = document.getElementById('result');
      resultDiv.textContent = message;
      resultDiv.style.fontSize = '24px';
      resultDiv.style.fontWeight = 'bold';
    }

    // window.onload necessary to keep JavaScripts from running before the app loads entirely
    window.onload = function () {
      canvas = document.getElementById('myCanvas'); // Get the canvas element
      context = canvas.getContext("2d"); //Get 2D drawing context

      var fileInput = document.getElementById('fileInput'); // Image upload input
      var messageDisplayArea = document.getElementById('messageDisplayArea'); //Message output div 
      var uploadedImage = null; // Placeholder for uploaded image

      // Calculates distance between two RGB color values
      function colorDistance(r1, g1, b1, r2, g2, b2) {
        return Math.sqrt((r1 - r2) ** 2 + (g1 - g2) ** 2 + (b1 - b2) ** 2);
      }

      // Returns true if color is close to pink (using rough RGB values)
      function isPink(r, g, b) {
        return colorDistance(r, g, b, 255, 105, 180) < 100;
      }

      // Returns true if color matches yellow within certain thresholds
      function isYellow(r, g, b) {
        return r > 200 && g > 180 && b < 120;
      }
      
      // Analyzes a 20x20 pixel square region centered at (x,y)
      function analyzeClickColor(x, y) {
        var imgData = context.getImageData(x - 10, y - 10, 20, 20);
        var pixels = imgData.data;
        let pinkPixels = 0;
        let yellowPixels = 0;
        
        // Loop over every pixel (each pixel = 4 values: R, G, B, A)
        for (let i = 0; i < pixels.length; i += 4) {
          let r = pixels[i];
          let g = pixels[i + 1];
          let b = pixels[i + 2];
          if (isPink(r, g, b)) pinkPixels++; // Count pink pixels
          else if (isYellow(r, g, b)) yellowPixels++; // Count yellow pixels
        }
        
        // Display diagnostic pixel counts in the message area
        messageDisplayArea.innerHTML += `<br>Analyzed click region: ${pinkPixels} pink pixels, ${yellowPixels} yellow pixels.`;

        // Display test result based on color dominance
        if (pinkPixels > yellowPixels && pinkPixels > 20) {
          showResult("Negative - Color Detected: Pink");
        } else if (yellowPixels > pinkPixels && yellowPixels > 20) {
          showResult("Positive - Color Detected: Yellow");
        } else {
          showResult("No significant color detected.");
        }
      }

      // Add click listener to canvas to analyze color at click location
      canvas.addEventListener("click", function (evt) {
        var rect = canvas.getBoundingClientRect(); // Get canvas position
        var x = evt.clientX - rect.left; // X coordinate relative to canvas
        var y = evt.clientY - rect.top; // Y coordinate relative to canvas
        analyzeClickColor(x, y); // Analyze colors at click
      });

      // Listen for file upload from user
      fileInput.addEventListener('change', function () {
        var file = fileInput.files[0]; // Get selected file
        var imageType = /image.*/; // Regex for image MIME types

        if (file.type.match(imageType)) {
          var reader = new FileReader(); // FileReader to read file contents
          reader.onload = function (e) {
            messageDisplayArea.innerHTML = "You picked an image! Click a tube to analyze its color.";
            var img = new Image(); // Create an image element

            img.onload = function () {
              uploadedImage = img; // Save image globally
              // Draw image with iOS fix applied
              drawImageIOSFix(context, img, 0, 0, img.naturalWidth, img.naturalHeight, 0, 0, canvas.width, canvas.height);
            };
            img.src = reader.result; // Set image source to uploaded file
          };

          reader.readAsDataURL(file); // Start reading file as base64 string
        } else {
          messageDisplayArea.innerHTML = "File not supported!"; // Error message for unsupported file type
        }
      });
    };
  </script>
</head>

<body>
  <!-- jQuery Mobile layout for page -->
  <div data-role="page">
    <div data-role="header">
      <h1>Workblock Hackathon!</h1> <!-- Page heading -->
    </div>

    <div data-role="content">
      <h2>LAMP test results interpreter</h2>
      <div>
      <!-- File input with styled orange background -->
        Imput your image of your tubes and click the liquid of sample you want to interpret:
        <input type="file" id="fileInput" style="background-color: #FF6600;">
      </div>
      
      <!-- Areas for messages and result -->
      <div id="messageDisplayArea"></div>
      <div id="result"></div>

      <!-- Canvas for displaying uploaded image -->
      <canvas id="myCanvas" width="600" height="500" style="border:1px solid #d3d3d3;"></canvas>
      
      <!-- Draw default text on canvas before image upload -->
      <script>
        context.font = '20pt Calibri'; // Set font style
        context.fillStyle = "black"; // Set font color
        context.fillText("Original image will go here", 10, 20); // Placeholder text
      </script>
    </div><!-- /content -->

    <div data-role="footer" data-position="fixed">
      <p>DIY Diagnostics</p> <!-- Footer text -->
    </div><!-- /footer -->
  </div><!-- /page -->
</body>

</html>
