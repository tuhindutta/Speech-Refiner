# 🎙️ Speech-Refiner

![image](https://github.com/user-attachments/assets/82a46653-ac8c-49bd-9c26-27aafba944cc)

## 📌 Overview
Speech Refiner is an Electron-based desktop application that allows users to record their voice or upload `.wav` audio files, and refine the tone of speech using a polite transformation API (hosted separately). The application is built using modern web technologies and packaged for desktop use via Electron.

## 🛠️ [Backend](https://tuhindutta.github.io/Politeness-Engine-API-Backend/)

## 🖥️ Frontend
  - ### 🚀 Features
    - 🎤 In-App Audio Recording via microphone (saved in proper `.wav` format)
    - 📁 File Upload option for `.wav` files
    - 🔁 Playback Support with ability to review recorded or uploaded audio
    - ⚡ One-Click API Call to refine speech
    - 🌙 Modern UI built with Bootstrap 5, supports dark mode and drag & drop
    - 📦 Electron Desktop Packaging (cross-platform-ready)   
  - ### 🧩 Tech Stack
    - Frontend Framework: Vanilla **HTML**, **CSS**, **JavaScript**
    - UI Library: [Bootstrap 5](https://getbootstrap.com/)
    - Recording Engine: `recorder.js` (local patched version - with more explanation below in *troubleshooting section*)
    - Electron: For packaging the app as a desktop .exe
    - External API: Abstracted polite speech transformation engine
  - 🗂️ Directory Structure
    ```graphql
    ├── index.html            # Main UI
    ├── recorder.js           # Local patched WAV recorder script
    ├── favicon.ico           # App icon
    └── main.js               # Electron main process
    ```
  - 🎮 Usage Flow
    1. Record Audio
       - Click the "Record" button to start capturing audio from your microphone.
       - Press "Stop" to end recording. The audio will auto-play and show duration.
    2. Or Upload File
       - Use the file picker or drag-and-drop a `.wav` file into the interface.
    3. Download
       - Optionally download your recording using the built-in playback control.
  - 📦 Packaging for Distribution
    - To run the app, run:
    - To build the `.exe`:
      ```bash
      npx electron .
      ```
      ```bash
      npm install
      npx electron-packager . SpeechRefiner --platform=win32 --arch=x64 --icon=favicon.ico --overwrite
      ```
      > This will create a packaged version of the app using Electron Packager or Electron Forge (as configured).
  - ⚙️ Notes
    - Microphone access must be allowed when prompted.
    - The API endpoint is abstracted and not included in the public repository. (Users can request for the API)
    - Replace the placeholder API URL with the required API when using.
  - 🛡️ Security
    - This version is intended for local/private use:
    - No sensitive data is stored or shared.
    - Content Security Policy (CSP) is not enforced to preserve dev flexibility.
    - The `.exe` is not recommended for public distribution with embedded API keys.

## 📄 Troubleshooting Audio Recording & CSP Integration in Speech Refiner
1. ✅ Local API vs Hosted API Issues
  - Issue: Hosted API (`http://192.168.x.x:5000`) didn’t respond as expected inside Electron.
  - Cause: Hosted API had longer response time (~5 sec) with no visual feedback.
  - Fixes:
      - Added a `Processing...` loader during API call.
      - Verified API response behavior using Postman/browser.
   
2. 🚫 Unsupported Audio Format (WebM instead of WAV)
   - Error:
     ```bash
     File format b'\x1aE\xdf\xa3' not understood. Only 'RIFF' and 'RIFX' supported.
     ```
   - Cause: MediaRecorder API defaulted to WebM; Flask expected `.wav`.
   - Fix: Switched to `recorder.js` which generates proper `.wav` output.

3. 📛 Invalid WAV Header (nAvgBytesPerSec mismatch)
   - Error:
     ```bash
     WAV header is invalid: nAvgBytesPerSec must equal product of nSamplesPerSec and nBlockAlign
     ```
   - Cause: Some versions of Recorder.js generated incorrect headers.
   - Fixes:
       - CDN failed due to MIME issues.
       - Forked versions had broken links.
       - Manually downloaded corrected `recorder.js` from GitHub and loaded it locally.
    
4. 📦 MIME Type Execution Errors
   - Error:
     ```bash
     Refused to execute script from CDN because its MIME type was 'text/plain'
     ```
   - Fix: Used local version of recorder.js:
     ```html
     <script src="recorder.js"></script>
     ```

5. 🛡️ Electron Warning
   - Message:
     ```console
     Insecure Content-Security-Policy: no CSP or unsafe-eval used
     ```
   - Strict CSP Attempt
     ```html
     <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'; connect-src http://192.168.x.x:5000;">
     ```
     - Issue:
       - Inline scripts blocked.
       - Microphone stopped.
       - API hit prematurely without file.
     - Root Cause
       - Electron apps commonly use inline scripts or libraries requiring relaxed policies.
       - Strict CSP blocks `eval`, inline JavaScript, dynamic execution.
     - Solutions Attempted
       - Tried relaxed CSP with:
         ```html
         script-src 'self' 'unsafe-inline'
         ```
       - Inline scripts worked, but reintroduced security risks (e.g., XSS).
     - Final Decision
       - ❌ CSP not applied now due to dev-time constraints.
       - ✅ Plan:
         - Keep .exe private.
         - Share code with API placeholder.
         - Let users build locally and request API key if needed.

> ##### `recorder.js` is downloaded from [here](https://raw.githubusercontent.com/sophister/recorderjs-ex/master/dist/recorder.js).

## 🧭 Recommendations for Future Deployment
- Extract all inline scripts into external files.
- Set a strict and secure CSP header.
- Remove unsafe-inline and unsafe-eval.
- Validate microphone permissions and backend headers for production use.

## 🤝 Contributing
Contributions are welcome! Please fork the repository and submit a pull request for any enhancements or bug fixes. For more details and updates, visit the [GitHub Repository](https://github.com/tuhindutta/Speech-Refiner).
