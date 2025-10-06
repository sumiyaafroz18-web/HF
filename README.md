import zipfile
import os

# Folder setup for full MHF PWA
folder_name = '/mnt/data/MHF_full_app'
os.makedirs(folder_name, exist_ok=True)

# index.html (full version with Firebase + placeholders)
index_html = """<!DOCTYPE html>
<html lang="bn">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>MHF - Full Version</title>
<link rel="stylesheet" href="style.css">
<link rel="manifest" href="manifest.json">
<script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="min-h-screen bg-gray-100">

<header class="sticky top-0 bg-white shadow-md z-10 p-4 flex justify-between items-center">
<h1 class="text-2xl font-bold text-blue-600">MHF</h1>
<div id="auth-status" class="text-sm text-gray-500 flex items-center">
<span class="mr-2">User ID:</span>
<span id="user-display-id" class="font-mono text-xs bg-gray-100 p-1 rounded-full text-blue-600">Loading...</span>
</div>
</header>

<main class="max-w-4xl mx-auto p-4 space-y-6">

<!-- Post Creation -->
<div class="bg-white p-4 rounded-xl shadow-lg">
<textarea id="post-input" rows="3" placeholder="মনোভাব প্রকাশ করুন..." class="w-full p-3 border border-gray-300 rounded-lg mb-3"></textarea>
<button id="post-btn" class="bg-blue-600 text-white px-6 py-2 rounded-full">পোস্ট করুন</button>
<div id="post-msg" class="mt-2 text-sm text-green-600"></div>
</div>

<!-- Post Feed -->
<div id="posts-container" class="space-y-4">
<p class="text-center text-gray-500 mt-8">পোস্ট লোড হচ্ছে...</p>
</div>

<!-- Courses Panel -->
<div class="bg-white p-4 rounded-xl shadow-lg border border-green-200">
<h2 class="text-xl font-bold text-green-600 mb-3">কোর্স বিক্রির সুযোগ</h2>
<p class="text-gray-600 mb-3">প্রতিষ্ঠানরা কোর্স আপলোড এবং বিক্রি করতে পারবে।</p>
<button class="w-full bg-green-500 text-white px-4 py-2 rounded-lg font-semibold">কোর্স দেখুন (টাকা লেন্দেন)</button>
</div>

</main>

<!-- Firebase SDKs -->
<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
import { getFirestore, collection, addDoc, serverTimestamp, query, orderBy, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

// Firebase config placeholder (replace with your own)
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
let userId = null;

signInAnonymously(auth).catch(console.error);
onAuthStateChanged(auth, user => {
  if(user){userId=user.uid;document.getElementById('user-display-id').textContent=userId.substring(0,8)+'...';initializePostListener();}else{document.getElementById('user-display-id').textContent='Error';}
});

// Post creation logic
document.getElementById('post-btn').addEventListener('click', async ()=>{
const input=document.getElementById('post-input');
if(input.value.trim().length<1){alert('পোস্টের জন্য কিছু লিখুন');return;}
const postsCollection = collection(db,'posts');
await addDoc(postsCollection,{userId,userInput:input.value,timestamp:serverTimestamp()});
input.value='';
});

// Firestore listener for feed
function initializePostListener(){
const postsCollection = collection(db,'posts');
const q = query(postsCollection, orderBy('timestamp','desc'));
onSnapshot(q,snapshot=>{
const container=document.getElementById('posts-container');container.innerHTML='';
snapshot.forEach(doc=>{
const data=doc.data();
const div=document.createElement('div');
div.className='bg-white p-4 rounded-lg shadow';
div.innerHTML='<p class="text-gray-700">'+data.userInput+'</p><p class="text-xs text-gray-500 mt-2">'+(data.timestamp?new Date(data.timestamp.toDate()).toLocaleString('bn-BD'):'এখনই')+'</p>';
container.appendChild(div);
});
});
}

// Simple video watermark/security logic
window.addEventListener('contextmenu',e=>{if(e.target.tagName==='VIDEO'){e.preventDefault();alert('ভিডিওতে স্ক্রিনশট/ডাউনলোড ব্লক করা হয়েছে।');}});

</script>

<script>
if('serviceWorker' in navigator){navigator.serviceWorker.register('./sw.js').then(()=>console.log('SW Registered')).catch(console.error);}
</script>

</body>
</html>
"""

# style.css
style_css = """body {font-family: 'Inter', sans-serif; background-color:#f0f2f5;}
textarea {font-size:14px;}
@media (max-width:640px){.max-w-4xl{max-width:100%;}}"""

# manifest.json
manifest_json = """{
"name": "MHF",
"short_name": "MHF",
"start_url": "./index.html",
"display": "standalone",
"background_color": "#f0f2f5",
"theme_color": "#0d6efd",
"icons":[{"src":"icon.png","sizes":"192x192","type":"image/png"},{"src":"icon.png","sizes":"512x512","type":"image/png"}]
}"""

# sw.js
sw_js = """const CACHE_NAME='MHF-full-cache-v1';const urlsToCache=['./index.html','./style.css','./manifest.json'];self.addEventListener('install',e=>{e.waitUntil(caches.open(CACHE_NAME).then(c=>c.addAll(urlsToCache)))});self.addEventListener('fetch',e=>{e.respondWith(caches.match(e.request).then(r=>r||fetch(e.request)))});"""

# Dummy icon
icon_path = os.path.join(folder_name,'icon.png')
with open(icon_path,'wb') as f:
    f.write(b'\x89PNG\r\n\x1a\n')

# Write files
files = {'index.html': index_html,'style.css': style_css,'manifest.json': manifest_json,'sw.js': sw_js}
for filename, content in files.items():
    with open(os.path.join(folder_name,filename),'w',encoding='utf-8') as f:
        f.write(content)

# Create ZIP
zip_path='/mnt/data/MHF_full_app.zip'
with zipfile.ZipFile(zip_path,'w') as zipf:
    for root,_,filenames in os.walk(folder_name):
        for file in filenames:
            zipf.write(os.path.join(root,file), arcname=file)

zip_path# HF
