<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mood Mirror</title>
    <!-- Tailwind CSS for styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- face-api.js for face and expression detection -->
    <script src="https://cdn.jsdelivr.net/npm/face-api.js@0.22.2/dist/face-api.min.js"></script>
    <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        /* Add a subtle glow effect to the video and meme */
        .glow {
            box-shadow: 0 0 15px rgba(76, 29, 149, 0.5), 0 0 25px rgba(167, 139, 250, 0.4);
        }
        /* Style for the status text to make it pop */
        #status {
            text-shadow: 0px 1px 5px rgba(0,0,0,0.2);
        }
        /* Transition for the meme image for smooth loading */
        #memeImage {
            transition: opacity 0.5s ease-in-out;
        }
    </style>
</head>
<body class="bg-gray-900 text-white flex flex-col items-center justify-center min-h-screen p-4">

    <div class="w-full max-w-5xl mx-auto text-center">
        <h1 class="text-4xl md:text-5xl font-bold text-violet-400 mb-2">Mood Mirror</h1>
        <p class="text-gray-300 mb-6">Let's see a meme for that mood!</p>

        <!-- Main container for video and meme -->
        <div class="flex flex-col md:flex-row items-center justify-center gap-8">
            
            <!-- Video Feed Container -->
            <div class="relative w-full max-w-md bg-gray-800 rounded-lg overflow-hidden border-2 border-violet-500 glow">
                <video id="video" width="640" height="480" autoplay muted playsinline class="w-full h-auto"></video>
                <div id="loader" class="absolute inset-0 bg-black bg-opacity-75 flex items-center justify-center">
                    <div class="text-center">
                        <svg class="animate-spin h-8 w-8 text-violet-400 mx-auto mb-3" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                        </svg>
                        <p id="loader-text">Starting Camera...</p>
                    </div>
                </div>
            </div>

            <!-- Meme Display Container -->
            <div id="memeContainer" class="w-full max-w-md h-auto bg-gray-800 rounded-lg flex items-center justify-center p-4 border-2 border-violet-500 glow min-h-[300px] md:min-h-[480px]">
                <img id="memeImage" src="https://placehold.co/600x400/1f2937/a78bfa?text=Waiting+for+your+face..." alt="Meme corresponding to mood" class="rounded-md max-w-full h-auto">
            </div>
        </div>
        
        <!-- Status and Control -->
        <div class="mt-6">
            <p id="status" class="text-xl font-semibold text-violet-300 h-8">Initializing...</p>
            <button id="startButton" class="mt-4 px-6 py-3 bg-violet-600 text-white font-bold rounded-lg hover:bg-violet-700 transition-all duration-300 shadow-lg">
                Start Mood Mirror
            </button>
        </div>
    </div>

    <script>
        const video = document.getElementById('video');
        const startButton = document.getElementById('startButton');
        const statusText = document.getElementById('status');
        const memeImage = document.getElementById('memeImage');
        const loader = document.getElementById('loader');
        const loaderText = document.getElementById('loader-text');
        
        let detectionInterval;

        // Hide the video and loader initially
        video.style.display = 'none';
        loader.style.display = 'none';

        startButton.addEventListener('click', () => {
            startButton.style.display = 'none';
            video.style.display = 'block';
            loader.style.display = 'flex';
            startVideoAndDetection();
        });

        async function startVideoAndDetection() {
            try {
                // 1. Start the webcam stream
                statusText.textContent = 'Accessing your camera...';
                const stream = await navigator.mediaDevices.getUserMedia({ video: {} });
                video.srcObject = stream;

                video.addEventListener('play', async () => {
                    // 2. Load the face-api models from the correct CDN path
                    loaderText.textContent = 'Loading AI models... (this may take a moment)';
                    // Using the more stable NPM-based CDN path for the models
                    const MODEL_URL = 'https://cdn.jsdelivr.net/gh/justadudewhohacks/face-api.js@0.22.2/weights';
                    await Promise.all([
                        faceapi.nets.tinyFaceDetector.loadFromUri(MODEL_URL),
                        faceapi.nets.faceLandmark68Net.loadFromUri(MODEL_URL),
                        faceapi.nets.faceRecognitionNet.loadFromUri(MODEL_URL),
                        faceapi.nets.faceExpressionNet.loadFromUri(MODEL_URL)
                    ]);
                    
                    loader.style.display = 'none';
                    statusText.textContent = 'Looking for a face...';

                    // 3. Start the detection loop
                    detectionInterval = setInterval(async () => {
                        const detections = await faceapi.detectAllFaces(video, new faceapi.TinyFaceDetectorOptions()).withFaceLandmarks().withFaceExpressions();
                        
                        if (detections.length > 0) {
                            // Get the primary detected emotion
                            const expressions = detections[0].expressions;
                            const primaryEmotion = Object.keys(expressions).reduce((a, b) => expressions[a] > expressions[b] ? a : b);
                            
                            statusText.textContent = `I see you're feeling... ${primaryEmotion}!`;
                            updateMeme(primaryEmotion);
                        } else {
                            statusText.textContent = 'Hmm, can\'t see your face clearly.';
                        }
                    }, 1000); // Increased interval to 1s to be friendlier to the API
                });

            } catch (error) {
                console.error("Error starting video or detection:", error);
                statusText.textContent = 'Error: Could not access camera or load models.';
                loaderText.textContent = 'Permission denied or error. Please refresh and allow camera access.';
                loader.style.display = 'flex';
                video.style.display = 'none';
                startButton.style.display = 'block';
            }
        }

        let lastEmotion = '';
        let isFetchingMeme = false; // Flag to prevent multiple API calls at once

        // This function now fetches a meme from a public API
        async function updateMeme(emotion) {
            // Only fetch if the emotion is new and we are not already fetching
            if (emotion && emotion !== lastEmotion && !isFetchingMeme) {
                isFetchingMeme = true;
                lastEmotion = emotion;

                // --- IMPROVED SUBREDDIT MAPPING ---
                // This list is curated to find more relevant memes for each emotion.
                const subredditMap = {
                    happy: 'wholesomememes',
                    sad: 'sadposting',
                    angry: 'angrymemes',
                    surprised: 'memes', // 'surprised' memes are common in general meme subreddits
                    neutral: 'memes',
                    fearful: 'dankmemes', // Using a broad but popular meme source as specific ones are rare
                    disgusted: 'mildlyinfuriating' // This often has content that evokes disgust in a funny way
                };

                const subreddit = subredditMap[emotion] || 'memes'; // Default to r/memes
                const apiUrl = `https://meme-api.com/gimme/${subreddit}`;

                // Show a loading state
                memeImage.style.opacity = '0.5';

                try {
                    const response = await fetch(apiUrl);
                    if (!response.ok) {
                        throw new Error(`API request failed with status ${response.status}`);
                    }
                    const data = await response.json();

                    // The API sometimes returns non-image posts, so we check for a valid image URL
                    if (data.url && data.url.match(/\.(jpeg|jpg|gif|png)$/)) {
                        memeImage.src = data.url;
                        memeImage.alt = data.title;
                    } else {
                        // If not an image, try again with the same emotion on the next detection cycle
                        console.log("Post was not an image, will retry on next detection.");
                        lastEmotion = ''; // Allow re-fetching for the same emotion
                    }
                } catch (error) {
                    console.error("Could not fetch meme:", error);
                    // Fallback to a simple placeholder if API fails
                    memeImage.src = `https://placehold.co/600x400/1f2937/a78bfa?text=Error+finding+meme+:(`;
                    memeImage.alt = 'Error fetching meme';
                } finally {
                    memeImage.style.opacity = '1';
                    isFetchingMeme = false; // Reset the flag
                }
            }
        }
    </script>
</body>
</html>
