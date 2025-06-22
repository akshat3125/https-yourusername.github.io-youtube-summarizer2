<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>YouTube Video Summarizer</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 20px;
        }
        
        .container {
            background: white;
            padding: 40px;
            border-radius: 20px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.1);
            max-width: 600px;
            width: 100%;
        }
        
        h1 {
            text-align: center;
            color: #333;
            margin-bottom: 30px;
            font-size: 2.5em;
            background: linear-gradient(45deg, #667eea, #764ba2);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }
        
        .input-group {
            margin-bottom: 20px;
        }
        
        label {
            display: block;
            margin-bottom: 8px;
            font-weight: 600;
            color: #555;
        }
        
        input[type="url"] {
            width: 100%;
            padding: 15px;
            border: 2px solid #e1e5e9;
            border-radius: 10px;
            font-size: 16px;
            transition: all 0.3s ease;
        }
        
        input[type="url"]:focus {
            outline: none;
            border-color: #667eea;
            box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
        }
        
        .btn {
            width: 100%;
            padding: 15px;
            background: linear-gradient(45deg, #667eea, #764ba2);
            color: white;
            border: none;
            border-radius: 10px;
            font-size: 18px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s ease;
            margin-top: 20px;
        }
        
        .btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 20px rgba(102, 126, 234, 0.3);
        }
        
        .btn:disabled {
            opacity: 0.6;
            cursor: not-allowed;
            transform: none;
        }
        
        .loading {
            display: none;
            text-align: center;
            margin-top: 20px;
        }
        
        .spinner {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #667eea;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
            margin: 0 auto 10px;
        }
        
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        
        .result {
            margin-top: 30px;
            padding: 20px;
            background: #f8f9fa;
            border-radius: 10px;
            display: none;
        }
        
        .result h3 {
            color: #333;
            margin-bottom: 15px;
        }
        
        .result pre {
            background: #fff;
            padding: 15px;
            border-radius: 8px;
            overflow-x: auto;
            font-size: 14px;
            border: 1px solid #e1e5e9;
        }
        
        .error {
            background: #fee;
            border-color: #fcc;
            color: #c33;
        }
        
        .success {
            background: #efe;
            border-color: #cfc;
            color: #3c3;
        }
        
        .webhook-config {
            margin-top: 30px;
            padding: 20px;
            background: #f0f8ff;
            border-radius: 10px;
            border-left: 4px solid #667eea;
        }
        
        .webhook-config h3 {
            color: #333;
            margin-bottom: 10px;
        }
        
        .webhook-config input {
            margin-top: 10px;
        }
        
        .example-urls {
            margin-top: 20px;
            padding: 15px;
            background: #fff9e6;
            border-radius: 8px;
            border-left: 4px solid #ffc107;
        }
        
        .example-urls h4 {
            color: #856404;
            margin-bottom: 10px;
        }
        
        .example-urls ul {
            list-style: none;
            padding-left: 0;
        }
        
        .example-urls li {
            margin: 5px 0;
            padding: 5px 10px;
            background: white;
            border-radius: 5px;
            cursor: pointer;
            transition: background 0.2s;
        }
        
        .example-urls li:hover {
            background: #f8f9fa;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🎬 YouTube Summarizer</h1>
        
        <div class="webhook-config">
            <h3>⚙️ Webhook Configuration</h3>
            <label for="webhook-url">N8N Webhook URL:</label>
            <input type="url" id="webhook-url" placeholder="http://localhost:5678/webhook/youtube-summarizer" value="http://localhost:5678/webhook/youtube-summarizer">
            <small style="color: #666; display: block; margin-top: 5px;">
                Make sure your N8N workflow is active and this URL matches your webhook trigger
            </small>
        </div>
        
        <form id="youtube-form">
            <div class="input-group">
                <label for="youtube-url">YouTube Video URL:</label>
                <input type="url" id="youtube-url" placeholder="https://www.youtube.com/watch?v=..." required>
            </div>
            
            <button type="submit" id="submit-btn" class="btn">
                📊 Analyze Video
            </button>
        </form>
        
        <div class="loading" id="loading">
            <div class="spinner"></div>
            <p>Analyzing video... This may take 30-60 seconds</p>
        </div>
        
        <div class="result" id="result">
            <h3>📋 Analysis Result:</h3>
            <pre id="result-content"></pre>
        </div>
        
        <div class="example-urls">
            <h4>📝 Example URLs (click to try):</h4>
            <ul>
                <li onclick="fillUrl('https://www.youtube.com/watch?v=dQw4w9WgXcQ')">
                    🎵 Rick Astley - Never Gonna Give You Up
                </li>
                <li onclick="fillUrl('https://www.youtube.com/watch?v=fJ9rUzIMcZQ')">
                    🎯 Marketing Strategy Video
                </li>
                <li onclick="fillUrl('https://www.youtube.com/watch?v=BxV14h0kFs0')">
                    💡 Productivity Tips
                </li>
            </ul>
        </div>
    </div>

    <script>
        // Get DOM elements
        const form = document.getElementById('youtube-form');
        const urlInput = document.getElementById('youtube-url');
        const webhookInput = document.getElementById('webhook-url');
        const submitBtn = document.getElementById('submit-btn');
        const loading = document.getElementById('loading');
        const result = document.getElementById('result');
        const resultContent = document.getElementById('result-content');

        // Load saved webhook URL
        const savedWebhook = localStorage.getItem('webhook-url');
        if (savedWebhook) {
            webhookInput.value = savedWebhook;
        }

        // Save webhook URL when changed
        webhookInput.addEventListener('change', function() {
            localStorage.setItem('webhook-url', this.value);
        });

        // Fill URL function for example links
        function fillUrl(url) {
            urlInput.value = url;
            urlInput.focus();
        }

        // Form submission handler
        form.addEventListener('submit', async function(e) {
            e.preventDefault();
            
            const youtubeUrl = urlInput.value.trim();
            const webhookUrl = webhookInput.value.trim();
            
            if (!youtubeUrl || !webhookUrl) {
                showResult('Please fill in both YouTube URL and Webhook URL', 'error');
                return;
            }

            // Validate YouTube URL
            if (!isValidYouTubeUrl(youtubeUrl)) {
                showResult('Please enter a valid YouTube URL', 'error');
                return;
            }

            // Show loading state
            setLoading(true);
            
            try {
                // Send data to N8N webhook
                const response = await fetch(webhookUrl, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify({
                        video_url: youtubeUrl,
                        timestamp: new Date().toISOString(),
                        user_agent: navigator.userAgent
                    })
                });

                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }

                const data = await response.text();
                showResult(data, 'success');
                
            } catch (error) {
                console.error('Error:', error);
                showResult(`Error: ${error.message}\n\nTroubleshooting:\n1. Make sure N8N is running (http://localhost:5678)\n2. Check that your webhook URL is correct\n3. Ensure the workflow is activated\n4. Check browser console for CORS errors`, 'error');
            } finally {
                setLoading(false);
            }
        });

        // Helper functions
        function setLoading(isLoading) {
            loading.style.display = isLoading ? 'block' : 'none';
            submitBtn.disabled = isLoading;
            submitBtn.textContent = isLoading ? '🔄 Processing...' : '📊 Analyze Video';
            result.style.display = 'none';
        }

        function showResult(content, type = 'success') {
            result.style.display = 'block';
            result.className = `result ${type}`;
            resultContent.textContent = content;
            
            // Scroll to result
            result.scrollIntoView({ behavior: 'smooth' });
        }

        function isValidYouTubeUrl(url) {
            const youtubeRegex = /^(https?:\/\/)?(www\.)?(youtube\.com\/watch\?v=|youtu\.be\/|youtube\.com\/embed\/|youtube\.com\/v\/)[a-zA-Z0-9_-]{11}/;
            return youtubeRegex.test(url);
        }

        // Add some helpful keyboard shortcuts
        document.addEventListener('keydown', function(e) {
            // Ctrl/Cmd + Enter to submit
            if ((e.ctrlKey || e.metaKey) && e.key === 'Enter') {
                form.dispatchEvent(new Event('submit'));
            }
            
            // Escape to clear result
            if (e.key === 'Escape') {
                result.style.display = 'none';
            }
        });

        // Auto-focus on URL input
        urlInput.focus();
    </script>
</body>
</html>
