This app open image `project_directory/static/image.jpg` and user draw bounding boxes. The result is save in a `bounding_boxes.json` file. This app is made with a help of chatGPT 40, with multiple prompts.

```bash
project_directory
    /static
        image.jpg
    /templates
        index.html
    app.py
```

index.html:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Draw Multiple Bounding Boxes</title>
    <style>
        canvas {
            border: 1px solid black;
            display: block;
        }
    </style>
</head>
<body>
    <h1>Draw Multiple Bounding Boxes on the Image</h1>
    
    <!-- Canvas where the image and bounding box will be drawn -->
    <canvas id="imageCanvas"></canvas>

    <!-- Download button for bounding boxes -->
    <button id="downloadButton" style="display:none;">Download Bounding Box Data</button>

    <script>
        const imageCanvas = document.getElementById('imageCanvas');
        const ctx = imageCanvas.getContext('2d');
        const img = new Image();
        img.src = "{{ url_for('static', filename='image.jpg') }}";  // Load image

        let startX, startY, isDrawing = false;
        const boundingBoxes = [];  // Array to store multiple bounding boxes

        // When the image is loaded, set the canvas size to the image size and draw the image
        img.onload = function() {
            imageCanvas.width = img.width;
            imageCanvas.height = img.height;
            ctx.drawImage(img, 0, 0);
        };

        // Function to redraw all bounding boxes
        function redrawCanvas() {
            ctx.clearRect(0, 0, imageCanvas.width, imageCanvas.height);
            ctx.drawImage(img, 0, 0);

            // Redraw all bounding boxes
            boundingBoxes.forEach(box => {
                ctx.beginPath();
                ctx.rect(box.startX, box.startY, box.width, box.height);
                ctx.strokeStyle = 'red';
                ctx.lineWidth = 2;
                ctx.stroke();
            });
        }

        // Mouse down event to start drawing the bounding box
        imageCanvas.addEventListener('mousedown', function(e) {
            startX = e.offsetX;
            startY = e.offsetY;
            isDrawing = true;
        });

        // Mouse move event to draw the bounding box dynamically as the user moves the mouse
        imageCanvas.addEventListener('mousemove', function(e) {
            if (isDrawing) {
                const currentX = e.offsetX;
                const currentY = e.offsetY;
                const width = currentX - startX;
                const height = currentY - startY;

                // Redraw the canvas and display the new bounding box being drawn
                redrawCanvas();
                ctx.beginPath();
                ctx.rect(startX, startY, width, height);
                ctx.strokeStyle = 'blue';  // Use a different color while drawing
                ctx.lineWidth = 2;
                ctx.stroke();
            }
        });

        // Mouse up event to finalize the bounding box drawing
        imageCanvas.addEventListener('mouseup', function(e) {
            isDrawing = false;

            // Capture the final bounding box dimensions
            const currentX = e.offsetX;
            const currentY = e.offsetY;
            const width = currentX - startX;
            const height = currentY - startY;

            // Save the bounding box data locally
            boundingBoxes.push({ startX, startY, width, height });

            // Send the bounding box data to the server via AJAX
            sendBoundingBoxData(startX, startY, width, height);

            // Redraw the canvas with the new bounding box included
            redrawCanvas();
        });

        // Function to send bounding box data to the server
        function sendBoundingBoxData(x, y, width, height) {
            const data = {
                startX: x,
                startY: y,
                width: width,
                height: height
            };

            fetch('/save_bounding_box', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(data)
            })
            .then(response => response.json())
            .then(data => {
                console.log('Bounding box saved:', data);
                // Show the download button after the bounding box is saved
                document.getElementById('downloadButton').style.display = 'block';
            })
            .catch((error) => {
                console.error('Error:', error);
            });
        }

        // Add event listener to download button
        document.getElementById('downloadButton').addEventListener('click', function() {
            window.location.href = '/download_bounding_boxes';
        });

    </script>
</body>
</html>
```

app.py
```python
from flask import Flask, request, jsonify, render_template, send_file
import os
import json

app = Flask(__name__)

# Path to save the bounding box data
BOUNDING_BOX_FILE = 'bounding_boxes.json'

@app.route('/')
def home():
    return render_template('index.html')

# Route to handle saving the bounding box data
@app.route('/save_bounding_box', methods=['POST'])
def save_bounding_box():
    data = request.get_json()  # Get the JSON data sent from the frontend

    # Bounding box data structure
    bounding_box_data = {
        "startX": data['startX'],
        "startY": data['startY'],
        "width": data['width'],
        "height": data['height']
    }

    # Load existing bounding boxes if they exist, otherwise create an empty list
    if os.path.exists(BOUNDING_BOX_FILE):
        with open(BOUNDING_BOX_FILE, 'r') as f:
            boxes = json.load(f)
    else:
        boxes = []

    # Add the new bounding box to the list and save to file
    boxes.append(bounding_box_data)
    with open(BOUNDING_BOX_FILE, 'w') as f:
        json.dump(boxes, f)

    print(f"Bounding Box saved: {bounding_box_data}")

    # Respond back to the frontend with success
    return jsonify({"message": "Bounding box saved successfully"}), 200

# Route to download the bounding box data as a file
@app.route('/download_bounding_boxes', methods=['GET'])
def download_bounding_boxes():
    if os.path.exists(BOUNDING_BOX_FILE):
        return send_file(BOUNDING_BOX_FILE, as_attachment=True)
    else:
        return jsonify({"message": "No bounding box data available"}), 404

if __name__ == '__main__':
    app.run(debug=True)
```

Requirement: 

```bash
pip install Flask
```

Run the app by `python app.py`. 

Result is `bounding_boxes.json`. E.g.:
`[{"startX": 551, "startY": 66, "width": 384, "height": 543}, {"startX": 37, "startY": 106, "width": 515, "height": 523}]`
