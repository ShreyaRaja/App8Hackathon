# App8Hackathon
<!DOCTYPE html>
<html lang="en">

<!-- The following code has been developed by students and/or researchers of the Freshman Research Initiative, DIY Diagnostics Stream at The University of Texas at Austin.  This code is shared for demonstration purposes and should not be considered a product -- it is for entertainment purposes only.  Any user of this code does so at their own risk. Members of the DIY Stream, FRI, and The University of Texas system are not liable for anything related to this code.
 
  THIS CODE SHOULD NOT BE USED TO DIAGNOSE ANY KIND OF MEDICAL CONDITION.
 
  Further Information:
  http://cns.utexas.edu/fri
 
  Research Educator:
  Timothy E. Riedel
  triedel@utexas.edu
 
  Authors in chronological order of contribution:
  Author 1: Timothy E. Riedel
  Author 2: Workblock 1
  
  References:
  http://docs.webplatform.org/wiki/concepts/programming/drawing_images_onto_canvas#Loading_the_image_programmatically
  http://www.html5rocks.com/en/tutorials/file/dndfiles/
  http://www.w3.org/TR/FileAPI/
  http://mobilehtml5.org/
  http://stackoverflow.com/questions/11929099/html5-canvas-drawimage-ratio-bug-ios
  
  Brief Description of Goal of Code:
  
 
  Known Issues:
  blah, blah, blah, blah
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
  console.log("This file last modified  " + docMod); // displays last modified date and time in the javascipt browser console 

/* detectVerticalSquash & drawImageIOSFix = are functions that are critical due to a flaw in the iPhone camera software that squishes any photos displayed on an HTML5 canvas element. https://github.com/stomita/ios-imagefile-megapixel
*/
 function detectVerticalSquash(img) {
  var iw = img.naturalWidth, ih = img.naturalHeight;
  var canvas = document.createElement('canvas');
  canvas.width = 2;
  canvas.height = ih;
  var ctx = canvas.getContext('2d');
  ctx.drawImage(img, 0, 0);
  var data = ctx.getImageData(0, 0, 1, ih).data;
  var sy = 0;
  var ey = ih;
  var py = ih;
  while (py > sy) {
   var alpha = data[(py - 1) * 4 + 3];
   if (alpha === 0) {
    ey = py;
   } 
   else {
    sy = py;
   }
   py = (ey + sy) >> 1;
  }
  var ratio = (py / ih);
  return (ratio===0)?1:ratio;
 }

 function drawImageIOSFix(ctx, img, sx, sy, sw, sh, dx, dy, dw, dh) {
  var vertSquashRatio = detectVerticalSquash(img);
  ctx.drawImage(img, sx, sy, sw, sh, dx, dy, dw, dh / vertSquashRatio);
 }

Function showResult(message) { 
const resultDiv = document.getElementById('result');
  resultDiv.textContent = message;
  resultDiv.style.fontSize = '24px';
  resultDiv.style.fontWeight = 'bold';
}

/* window.onload necessary to keep javascripts from running before the app gets a chance to load entirely. */
 window.onload = function() {
  var fileInput = document.getElementById('fileInput');
  var messageDisplayArea = document.getElementById('messageDisplayArea');
  /* fileInput.addEventListener looks for any change in the <input> tag indicating that the user has selected a file (picture). */
  fileInput.addEventListener('change', function(e) {
    var file = fileInput.files[0];
    var imageType = /image.*/;
    /* if (file.type.match(imageType)) checks that the selected file is indeed an image. */
    if (file.type.match(imageType)) {
      /* new FileReader(); and reader.onload start some kind of file reading process in javaScript. I do not understand why this is necessary although it may simply slow things down to let the image load. */
      var reader = new FileReader();
      reader.onload = function(e) {
        /* messageDisplayArea.innerHTML not needed but allows you to send a message at this point in the program. */
        messageDisplayArea.innerHTML = "You picked an image!";
        /* new Image() and img.onload start some kind of new image creation process in javaScript. I do not understand why this is necessary although it may simply slow things down to let the image load. */
        var img = new Image();
        img.onload = function() {
          /*This finally draws the image onto the first drawing canvas. The specific canvas is referred to by "context". This is established down in the HTML part of the file just under the first <canvas> tag. If we weren't worried about fixing an iPhone bug we would use context.drawImage(image, 0, 0, img.naturalWidth, img.naturalHeight, 0, 0, 600, 500) instead of drawImageIOSFix */
        
          drawImageIOSFix(context,img, 0, 0, img.naturalWidth, img.naturalHeight, 0, 0, 600, 500);

 // ====================== BEGIN TUBE DETECTION ======================

// Get pixel data from the canvas
var imgData = context.getImageData(0, 0, canvas.width, canvas.height);
var pixels = imgData.data;

// Estimated width of one test tube (adjust if needed)
var tubeWidth = 40;
// Y-position across the image where tubes sit (middle of the canvas)
var rowY = 250;

// Color detection helpers
function isPink(r, g, b) {
  return r > 200 && g < 100 && b > 150;
}

function isYellow(r, g, b) {
  return r > 200 && g > 200 && b < 100;
}

let pinkCount = 0;
let yellowCount = 0;

// Loop across the image, assuming tubes are spaced left to right
for (let x = 0; x < canvas.width; x += tubeWidth) {
  let pinkPixels = 0;
  let yellowPixels = 0;

  // Scan a small vertical slice at this horizontal position
  for (let dx = 0; dx < tubeWidth; dx++) {
    for (let dy = -10; dy < 10; dy++) {
      let px = x + dx;
      let py = rowY + dy;
      let index = (py * canvas.width + px) * 4;
      let r = pixels[index];
      let g = pixels[index + 1];
      let b = pixels[index + 2];

      if (isPink(r, g, b)) pinkPixels++;
      else if (isYellow(r, g, b)) yellowPixels++;
    }
  }

  // If enough pixels in the area are pink/yellow, count this "tube"
  if (pinkPixels > 50) pinkCount++;
  else if (yellowPixels > 50) yellowCount++;
}

// Show result on the page
messageDisplayArea.innerHTML += `<br>Detected ${pinkCount} pink tube(s) and ${yellowCount} yellow tube(s).`;

// ====================== END TUBE DETECTION ======================

<div id=”result” ></div>
showResult(‘Positive - Color Dectected: Pink’); 
showResult(‘Negative - Color Dectected: Yellow’); 

          /*  +++++++++++++++ BEGIN IMAGE PROCESSING +++++++++++++++ */
          /*  ++++++++++++++++++++++++++++++++++++++++++++++++++++++ */

          /* PUT ANY CHANGES TO THE JAVASCRIPT HERE  */
          
          
          
          
        } // end of img.onload
        
        img.src = reader.result;
      } //  end of reader.onload
      
      reader.readAsDataURL(file);
    } // end of the if statement checking that file is an image
    else
    {
      // this is what happens if the original file selected is not an image file
      messageDisplayArea.innerHTML = "File not supported!"
    } // end of the else statement if file in NOT an image
  }); // end of the fileInput.addEventListener function
 } // end of the window.onload function
 </script>
 </head>

 <body>
  <div data-role="page">
    <div data-role="header">
    <h1>Workblock Hackathon!</h1></div><!-- /header -->
    <div data-role="content">	
      <h2>Push button to take picture.</h2>
      <div>
        Press orange button to create or choose image file:
        <input type="file" id="fileInput" STYLE="background-color: #FF6600;">
      </div>
      <div id="messageDisplayArea"></div>
      
      <canvas id="myCanvas" width=600 height=500 style="border:1px solid #d3d3d3;"></canvas>
      <script> /* This script initiates the cavas and puts the text on the canvas "myCanvas" */
        var canvas = document.getElementById('myCanvas');
        var context = canvas.getContext("2d");
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
