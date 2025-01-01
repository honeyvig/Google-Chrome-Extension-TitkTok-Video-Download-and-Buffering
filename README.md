# Google-Chrome-Extension-TitkTok-Video-Download-and-Buffering

### **Tasks Required:**  
1. **Sorting TikTok Videos:**  
   - Access my TikTok profile from your computer.  
   - Install a Google Chrome extension that allows sorting videos by views.  
   - Identify the **7 videos with the fewest views** on the page.  

2. **Downloading Videos:**  
   - Use the website [sssTik.io](https://ssstik.io) to download these 7 videos without the TikTok watermark.  

3. **Scheduling Videos:**  
   - Log in to my Buffer account (login details provided).  
   - Upload the 7 videos to Buffer and schedule them to be posted one per day at **2 AM** on TikTok, Instagram, and YouTube (already linked to Buffer).  

4. **Video Editing (if needed):**  
   - If any video exceeds 1 minute in length, you will need to use your own video editing software to trim it to 1 minute.  
   - Re-upload the edited video to Buffer and proceed with scheduling.  
-------------------
To develop a Chrome extension that automates the process you've outlined, several components need to be addressed. I'll break it down into specific steps for each task required.
Key Components for the Chrome Extension:

    TikTok Profile Scraping - Scraping the user's TikTok profile to list the videos sorted by views.
    Video Download - Using an external website (sssTik.io) to download videos without watermarks.
    Buffer Integration - Automating the login and scheduling of videos via Buffer API.
    Video Editing - Trimming videos that exceed 1 minute using an external library or API (for trimming, this may need to be done separately, as Chrome extensions can't directly perform video editing).

Steps & Code Overview:

We'll need a manifest file for Chrome extension settings and JavaScript files to handle the logic. Here's the structure:
1. Manifest File (manifest.json)

{
  "manifest_version": 3,
  "name": "TikTok Video Scheduler",
  "description": "Automates video sorting, downloading, and scheduling to Buffer.",
  "version": "1.0",
  "permissions": [
    "activeTab",
    "storage",
    "tabs",
    "identity",
    "http://*/*",
    "https://*/*"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "content_scripts": [
    {
      "matches": ["https://www.tiktok.com/@*"],
      "js": ["content.js"]
    }
  ],
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "assets/icon.png",
      "48": "assets/icon.png",
      "128": "assets/icon.png"
    }
  },
  "host_permissions": [
    "https://ssstik.io/*",
    "https://*.buffer.com/*"
  ],
  "icons": {
    "16": "assets/icon.png",
    "48": "assets/icon.png",
    "128": "assets/icon.png"
  }
}

2. Background Script (background.js)

This script can be used to handle the integration with Buffer API (login, upload, and scheduling).

// background.js - Handling buffer upload and scheduling
chrome.runtime.onInstalled.addListener(() => {
  console.log("TikTok Video Scheduler Extension Installed");
});

// Function to authenticate Buffer
const authenticateBuffer = async () => {
  // Use OAuth2.0 to authenticate and get access token
  // Send request to Buffer API to get access token using OAuth
};

// Function to upload video to Buffer
const uploadToBuffer = async (videoUrl, caption) => {
  const accessToken = 'YOUR_ACCESS_TOKEN'; // Replace with dynamic auth
  const response = await fetch("https://api.bufferapp.com/1/updates/create.json", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Authorization": `Bearer ${accessToken}`
    },
    body: JSON.stringify({
      profile_ids: ["profile_id_tiktok", "profile_id_instagram", "profile_id_youtube"],
      text: caption,
      media: { link: videoUrl },
      scheduled_at: new Date().toISOString() // Scheduling
    })
  });
  
  const data = await response.json();
  console.log("Video uploaded:", data);
};

3. Content Script (content.js)

This script will be injected into the TikTok profile page and will scrape the videos. It will sort them based on views and identify the 7 least viewed videos.

// content.js - Extracting and sorting videos from the TikTok profile
const getTikTokVideos = () => {
  let videos = [];
  const videoElements = document.querySelectorAll('div.tiktok-1ibdwbv-DivItemContainer');

  videoElements.forEach((videoElement) => {
    const videoUrl = videoElement.querySelector('a')?.href;
    const views = parseInt(videoElement.querySelector('.tiktok-1r33b61-SpanViews').textContent.replace(/\D/g, ''), 10);
    
    if (videoUrl && views) {
      videos.push({ videoUrl, views });
    }
  });

  // Sort videos by views in ascending order
  videos.sort((a, b) => a.views - b.views);

  // Return the first 7 videos with the fewest views
  return videos.slice(0, 7);
};

// Trigger the sorting and show the result in the console
const videosToDownload = getTikTokVideos();
console.log("7 Least Viewed Videos: ", videosToDownload);

4. Downloading Videos (sssTik.io Integration)

To download videos from TikTok using sssTik.io, we need to send a request to their site. Note that scraping or API access will need to conform to their terms of service.

// Example: Using sssTik.io to download a video
const downloadVideo = async (videoUrl) => {
  const apiUrl = `https://ssstik.io/api/download?url=${encodeURIComponent(videoUrl)}`;
  
  try {
    const response = await fetch(apiUrl);
    const data = await response.json();
    
    if (data.url) {
      const videoDownloadUrl = data.url;
      console.log("Download URL: ", videoDownloadUrl);
      return videoDownloadUrl; // Use this URL to download the video
    }
  } catch (error) {
    console.error("Error downloading video:", error);
  }
};

// Download the videos
videosToDownload.forEach(video => {
  downloadVideo(video.videoUrl);
});

5. Handling Video Editing (Optional)

For video editing, specifically trimming videos that exceed 1 minute, you’ll need to either integrate an API or use a separate application, as Chrome extensions cannot directly edit video files.

You can integrate an external service or provide instructions to users for manually trimming videos using tools like ffmpeg or Adobe Premiere.
6. Popup UI (popup.html & popup.js)

A basic popup to interact with the extension and trigger the functions.

popup.html

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>TikTok Scheduler</title>
  <style>
    body { font-family: Arial, sans-serif; padding: 10px; }
    button { padding: 10px; margin-top: 10px; cursor: pointer; background-color: #4CAF50; color: white; border: none; }
    button:hover { background-color: #45a049; }
  </style>
</head>
<body>
  <h3>TikTok Video Scheduler</h3>
  <button id="sortAndSchedule">Sort & Schedule Videos</button>
  <script src="popup.js"></script>
</body>
</html>

popup.js

// popup.js - Triggering functions from popup UI
document.getElementById('sortAndSchedule').addEventListener('click', () => {
  chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
    const tab = tabs[0];
    chrome.scripting.executeScript({
      target: { tabId: tab.id },
      func: getTikTokVideos
    }, (result) => {
      const videos = result[0].result;
      videos.forEach(video => {
        downloadVideo(video.videoUrl);
        // You can upload to Buffer after downloading videos
        uploadToBuffer(video.videoUrl, "Scheduled TikTok Video");
      });
    });
  });
});

7. Buffer API Authorization

Use OAuth2 to integrate Buffer's API and authenticate your users. You can utilize the chrome.identity API to handle authentication.

// OAuth2 Buffer Authentication using chrome.identity API
chrome.identity.getAuthToken({ interactive: true }, function(token) {
  // Use token to make authenticated requests to Buffer API
});

Conclusion

    This Chrome extension will scrape TikTok profiles, download the least viewed videos, and schedule them to Buffer.
    For video editing, you need external tools (e.g., ffmpeg or Adobe Premiere) since the extension cannot handle video editing directly.
    Be sure to respect the terms of service of the websites involved (TikTok, sssTik.io, and Buffer).

Testing and Debugging:

Ensure you test each part separately (sorting videos, downloading, uploading to Buffer) and handle errors or issues (e.g., rate limiting, network failures) gracefully in the production version.
