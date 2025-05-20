# AWS S3 + CloudFront + Aws Certificate Manager + Route 53 + Google Oauth

Here’s a step-by-step guide to set up the following stack:
- index.html hosted on S3
- Served via CloudFront
- HTTPS via AWS Certificate Manager
- Route 53 for DNS
- Google OAuth for authentication

## 🧾 Step-by-Step Setup
### ✅ 1. Prepare index.html File
Create your `index.html` file locally. This will be uploaded to S3.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Grammar Checker</title>
  <script src="https://accounts.google.com/gsi/client" async defer></script>
  <style>
    * {
      box-sizing: border-box;
      font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
    }

    body {
      background: #eef2f7;
      color: #1f2937;
      margin: 0;
      padding: 2rem;
      display: flex;
      justify-content: center;
      align-items: flex-start;
      min-height: 100vh;
    }

    .container {
      background: white;
      border-radius: 20px;
      box-shadow: 0 8px 30px rgba(0, 0, 0, 0.05);
      padding: 2rem;
      width: 100%;
      max-width: 720px;
    }

    h2 {
      margin-top: 0;
      font-size: 2rem;
      color: #111827;
      text-align: center;
    }

    textarea {
      width: 100%;
      padding: 1rem;
      border: 1px solid #d1d5db;
      border-radius: 12px;
      resize: vertical;
      font-size: 1rem;
      margin-top: 1rem;
    }

    button {
      background: #2563eb;
      color: white;
      border: none;
      padding: 0.75rem 1.5rem;
      font-size: 1rem;
      border-radius: 10px;
      cursor: pointer;
      margin-top: 1rem;
      transition: background 0.2s ease;
    }

    button:hover {
      background: #1d4ed8;
    }

    .output {
      margin-top: 2rem;
      background: #f3f4f6;
      border-radius: 12px;
      padding: 1rem;
      line-height: 1.6;
      font-size: 1rem;
    }

    .highlight {
      background: #fde68a;
      border-bottom: 2px dotted #f59e0b;
      padding: 0 4px;
      border-radius: 4px;
      cursor: help;
    }

    .footer {
      margin-top: 2rem;
      font-size: 0.875rem;
      color: #6b7280;
      text-align: center;
    }

    .login-info {
      background: #e0f2fe;
      border: 1px solid #90cdf4;
      padding: 1rem;
      border-radius: 12px;
      margin-bottom: 1rem;
      font-size: 1rem;
      text-align: center;
    }

    .user-info {
      display: flex;
      justify-content: space-between;
      align-items: center;
      background: #f0fdf4;
      border: 1px solid #bbf7d0;
      padding: 1rem;
      border-radius: 12px;
      margin-bottom: 1rem;
    }

    .user-details {
      display: flex;
      flex-direction: column;
    }

    .logout-btn {
      background: #ef4444;
      padding: 0.5rem 1rem;
      font-size: 0.9rem;
      border-radius: 8px;
      color: white;
      border: none;
      cursor: pointer;
    }

    .logout-btn:hover {
      background: #dc2626;
    }

    .google-signin-wrapper {
      display: flex;
      justify-content: center;
      margin-top: 1.5rem;
    }
  </style>
</head>
<body>
  <div class="container">
    <h2>📝 English Grammar Checker</h2>

    <div id="loginContainer">
      <div class="login-info">
        🔐 <strong>You can sign in with your Google account</strong><br>
        to access the Grammar Checker.
      </div>
      <div class="google-signin-wrapper">
        <div id="googleSignInDiv"></div>
      </div>
    </div>

    <div id="userInfo" class="user-info" style="display: none;">
      <div class="user-details">
        <div id="userName" style="font-weight: 600;"></div>
        <div id="userEmail" style="font-size: 0.9rem; color: #6b7280;"></div>
      </div>
      <button class="logout-btn" onclick="logout()">Logout</button>
    </div>
    <br>
    <label for="textInput">Enter a sentence or paragraph:</label>
    <textarea id="textInput" placeholder="Example: She go to school everyday." disabled></textarea>
    <button onclick="checkGrammar()" id="checkBtn" disabled>Check Grammar</button>

    <div class="output" id="output">Your corrected text will appear here.</div>

    <div class="footer">
      Powered by <a href="https://languagetool.org/" target="_blank">LanguageTool</a>
    </div>
  </div>

  <script>
    const GOOGLE_CLIENT_ID = "YOUR_GOOGLE_CLIENT_ID_HERE"; // Replace with your real client ID

    window.onload = function () {
      google.accounts.id.initialize({
        client_id: GOOGLE_CLIENT_ID,
        callback: handleCredentialResponse
      });

      google.accounts.id.renderButton(
        document.getElementById("googleSignInDiv"),
        { theme: "outline", size: "large" }
      );
    };

    function handleCredentialResponse(response) {
      const payload = JSON.parse(atob(response.credential.split('.')[1]));
      document.getElementById("userName").innerText = payload.name;
      document.getElementById("userEmail").innerText = payload.email;

      document.getElementById("loginContainer").style.display = "none";
      document.getElementById("userInfo").style.display = "flex";
      document.getElementById("textInput").disabled = false;
      document.getElementById("checkBtn").disabled = false;
    }

    function logout() {
      google.accounts.id.disableAutoSelect();
      document.getElementById("loginContainer").style.display = "block";
      document.getElementById("userInfo").style.display = "none";
      document.getElementById("textInput").value = "";
      document.getElementById("textInput").disabled = true;
      document.getElementById("checkBtn").disabled = true;
      document.getElementById("output").innerText = "Your corrected text will appear here.";
    }

    async function checkGrammar() {
      const text = document.getElementById("textInput").value;
      const outputDiv = document.getElementById("output");
      outputDiv.innerText = "Checking grammar...";

      const params = new URLSearchParams();
      params.append("text", text);
      params.append("language", "en-US");

      try {
        const res = await fetch("https://api.languagetoolplus.com/v2/check", {
          method: "POST",
          headers: { "Content-Type": "application/x-www-form-urlencoded" },
          body: params.toString()
        });

        const data = await res.json();

        if (data.matches.length === 0) {
          outputDiv.innerHTML = "<b>No grammar issues found!</b>";
          return;
        }

        let result = text;
        data.matches.reverse().forEach(match => {
          const replacement = match.replacements[0]?.value || "";
          if (replacement) {
            const start = match.offset;
            const end = match.offset + match.length;
            result = result.slice(0, start) +
              `<span class="highlight" title="${match.message}">${replacement}</span>` +
              result.slice(end);
          }
        });

        outputDiv.innerHTML = `<b>Suggested Correction:</b><br>${result}`;
      } catch (err) {
        outputDiv.innerText = "Error: " + err.message;
      }
    }
  </script>
</body>
</html>

```

### ✅ 2. Create and Configure S3 Bucket
a. Create S3 Bucket
  - Go to S3 Console
  - Click Create bucket
  - Name: `your-domain-name`
  - Uncheck "Block all public access"
  - Keep static website hosting disabled (use CloudFront instead)

b. Upload index.html
  - Open the bucket
  - Click Upload
  - Upload your index.html

c. Set Object Permissions
  - Go to the uploaded file
  - Click Permissions > Edit
  - let CloudFront handle access (recommended)

### ✅ 3. Create an SSL Certificate (ACM)
a. Go to AWS Certificate Manager
  - Request a certificate
  - Add domain: `your-domain-name` and `www.your-domain-name` (optional)

b. DNS Validation via Route 53
  - ACM will suggest a CNAME
  - Click Create record in Route 53

c. Wait until status is "Issued"

### ✅ 4. Set Up CloudFront Distribution
a. Create a Distribution
  - Origin domain: Select your S3 bucket (select bucket name, not website endpoint)
  - Viewer Protocol Policy: Redirect HTTP to HTTPS
  - Origin Access Control (OAC): Create a new one (recommended)
  - Restrict Bucket Access: Yes (use OAC)
  - Default Root Object: index.html
  - Alternate Domain Name (CNAME): `your-domain-name`
  - Custom SSL Certificate: Choose the one from ACM

b. Update S3 Bucket Policy (automatically done with OAC)

### ✅ 5. Setup Route 53 DNS
a. Go to Route 53 > Hosted Zones
  - Create a Hosted Zone (if not already exists) for `your-domain-name`

b. Create Record
  - Type: A or AAAA
  - Alias: Yes
  - Alias Target: Choose your CloudFront distribution
  - Save

### ✅ 6. Google Oauth Integration
a. Create OAuth 2.0 credentials in Google Cloud Console
  - Go to Google Cloud Console.
  - Create a project or use an existing one.
  - Go to APIs & Services → Credentials → Create Credentials → OAuth 2.0 Client IDs.
  - Application type: Web application
  - Add Authorized JavaScript origins, like:

```bash
your hosted domain (e.g https://example.com)
```

  - Save your Client ID.

b. Update your HTML
  - Replace your actual Google Client ID in place of `YOUR_GOOGLE_CLIENT_ID_HERE`.

### ✅ 7. Test the Setup
  - Open `https://YOUR_DOMAIN_NAME`
  - Confirm:
    - HTTPS is working
    - Content from index.html is loading
    - Google OAuth is protecting access
