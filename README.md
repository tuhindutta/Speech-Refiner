# Speech-Refiner

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

> ##### `recorder.js` is downloaded from [here](https://raw.githubusercontent.com/sophister/recorderjs-ex/master/dist/recorder.js).

5. 🛡️ Electron Warning
   - Message:
     ```console
     Insecure Content-Security-Policy: no CSP or unsafe-eval used
     ```
   - Strict CSP Attempt
     ```html
     <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'; connect-src http://192.168.1.5:5000;">
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

## 🧭 Recommendations for Future Deployment
- Extract all inline scripts into external files.
- Set a strict and secure CSP header.
- Remove unsafe-inline and unsafe-eval.
- Validate microphone permissions and backend headers for production use.
