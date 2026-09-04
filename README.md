# YouTube Video Downloader

A desktop application built with C# and WinForms to practice running external CLI tools asynchronously, reading real-time process output, and handling task cancellation.

---

## 🎥 Demo (Click to watch on YouTube)

<p align="center">
  <a href="https://youtu.be/Q05UTTn6txQ">
    <img src="https://img.youtube.com/vi/Q05UTTn6txQ/maxresdefault.jpg" />
  </a>
</p>

---

## Overview

A Windows application for fetching video details and downloading YouTube videos at selected resolutions. I built this project to practice managing external CLI tools (`yt-dlp` and `ffmpeg`) using the `Process` class, running them asynchronously in the background without freezing the UI, redirecting their standard output to track real-time download progress, managing up to 5 concurrent downloads, and using `CancellationToken` with process tree termination to cleanly stop downloads and clean up temporary files.

---

## Tech Stack

- C#
- .NET Framework
- WinForms
- Guna2UI
- **CLI Tools:** `yt-dlp`, `FFmpeg`
- **Data Storage:** JSON (`Downloads.json`)

---

## Technical Implementation

* **1. Fetching Video Details:**
  * Using the `Process` and `ProcessStartInfo` classes to run `yt-dlp` as an external process and passing arguments to dump metadata without downloading:
    
  * CLI arguments passed to `yt-dlp`:
    ```bash
    --dump-json --no-playlist --skip-download "<URL>"
    ```
  * Parsing the JSON output using `JsonDocument` to extract video details such as title, description, uploader, duration, thumbnail URL, and available video resolutions.
  * Caching the dumped JSON string in memory (`_cachedJson`) and passing it directly into `yt-dlp` through standard input (`--load-info-json - --simulate`) to run an offline download simulation for the selected resolution, calculating its approximate file size:
    
  * CLI arguments passed to `yt-dlp`:
    ```bash
    --load-info-json - --simulate --print "%(filesize,filesize_approx)s" -f "bv*[height<={selectedQuality}]+ba/b[height<={selectedQuality}]/best"
    ```

* **2. Downloading the Video:**
  * Launching `yt-dlp` as a separate background process using `Process` and `ProcessStartInfo` to download the chosen resolution and automatically merge the audio and video streams into an `.mp4` container using `ffmpeg`:
    
  * CLI arguments passed to `yt-dlp`:
    ```bash
    --no-playlist --force-overwrites --merge-output-format mp4 -f "bv*[height<={selectedQuality}]+ba/b[height<={selectedQuality}]/best" -o "<OUTPUT_PATH>" --newline "<URL>"
    ```

* **3. Real-Time Output Parsing & Progress Tracking:**
  * Subscribing to the `Process.OutputDataReceived` event to read line-by-line output from the process standard output (`stdout`) stream on a background thread.
  * Reading progress updates sent by `yt-dlp` to calculate download percentage and update the UI in real time without freezing the screen.

  
* **4. Cancellation & Process Cleanup:**
  * Creating a `CancellationTokenSource` when the user starts a download and passing its token to the download operation.
  * Registering a callback on the token that runs `taskkill /F /T /PID` to immediately stop both `yt-dlp` and any active `ffmpeg` child processes.
  * Deleting incomplete download files left on disk right after cancellation.

    
* **5. Saving Download History:**
  * Writing download records (completed, cancelled, or failed) to `Downloads.json` to preserve history between application restarts.
---

## App Features

- **Video Metadata Preview:** Fetches video details including title, author, duration, thumbnail, and available resolutions upon entering a valid URL.
- **Dynamic File Size Estimation:** Instantly calculates the approximate file size when switching between resolutions.
- **Concurrent Downloads:** Allows downloading multiple videos simultaneously (up to 5 concurrent downloads).
- **Live Progress Tracking:** Displays real-time progress percentages and download statuses (Downloading, Completed, Cancelled, Failed).
- **Download Management:** Right-click options to cancel active downloads, open the file location in Windows Explorer, or remove records from history.
- **Local History:** Saves download history directly to a local JSON file and reloads it every time the application starts.
