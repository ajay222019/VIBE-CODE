Mood Mirror
A fun, interactive web application that uses your computer's camera to detect your facial expression and displays a relevant meme that matches your mood in real-time.

Live Demo <-- Replace with your GitHub Pages link!
How it Works
The Mood Mirror is built entirely with client-side JavaScript, meaning your camera feed is never sent to a server, ensuring your privacy.

Camera Access: The app requests permission to use your webcam.

Face Detection: Using the powerful face-api.js library (which is built on TensorFlow.js), the application analyzes the video stream to find a face and identify key facial landmarks.

Emotion Recognition: The trained machine learning models in face-api.js classify the facial expression into one of seven categories: happy, sad, angry, surprised, neutral, fearful, or disgusted.

Meme Fetching: Once an emotion is detected, the app calls the Meme API to fetch a random meme from a subreddit that has been curated to match that specific emotion (e.g., r/wholesomememes for "happy").

Display: The fetched meme is then displayed on the screen, creating your "Mood Mirror"!

Features
Real-Time Emotion Detection: See your detected mood update live as your expression changes.

Dynamic Memes: Get a different, relevant meme from a public API every time your mood changes.

Privacy-Focused: All face detection happens directly in your browser. Your camera data is never uploaded.

Responsive Design: The layout works smoothly on both desktop and mobile devices.

No Installation Needed: Runs directly in any modern web browser.

Technologies Used
HTML5: For the basic structure of the web page.

Tailwind CSS: For modern and responsive styling.

JavaScript (ES6+): For all the application logic.

face-api.js: For the core face and expression detection, running on TensorFlow.js.

Meme API: For fetching random memes from Reddit.

How to Run Locally
Clone the repository:

git clone https://github.com/your-username/mood-mirror.git

Navigate to the directory:

cd mood-mirror

Run a local server:
Because the app requires camera access (getUserMedia), which is restricted on file:// protocols for security reasons, you need to serve the index.html file from a local web server. The easiest way to do this is with the Live Server extension in VS Code.

Right-click the index.html file.

Select "Open with Live Server".

This will open the application in your browser, and it will be fully functional.
