<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>CT241 - Commander du Cash</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <style>
        :root { --gabon-vert: #009E60; --gabon-jaune: #FCD116; --gabon-bleu: #3A75C4; }
        .bg-gabon-vert { background-color: var(--gabon-vert); }
        .text-gabon-vert { color: var(--gabon-vert); }
        .border-gabon-jaune { border-color: var(--gabon-jaune); }
        #map { height: 280px; width: 100%; border-radius: 16px; margin-top: 12px; display: none; z-index: 10; border: 2px solid #e5e7eb; }
        .pulse { animation: pulse-animation 2s infinite; }
        @keyframes pulse-animation { 0% { opacity: 1; } 50% { opacity: 0.4; } 100% { opacity: 1; } }
        input::-webkit-outer-spin-button, input::-webkit-inner-spin-button { -webkit-appearance: none; margin: 0; }
        .section { animation: fadeIn 0.3s ease-out; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
    </style>
</head>
<body class="bg-gray-50 text-gray-900 font-sans">

    <!-- HEADER -->
    <header class="bg-white sticky top-0 z-50 shadow-sm border-b-4 border-gabon-jaune p-4 flex justify-between items-center">
        <div class="flex items-center gap-2" onclick="location.reload()" style="cursor: pointer;">
            <div class="w-10 h-10 bg-gabon-vert rounded-xl flex items-center justify-center text-white font-black shadow-sm">CT</div>
            <div>
                <h1 class="font-black text-xl leading-none tracking-tighter text-gray-800">CT241</h1>
                <span class="text-[10px] font-bold text-gabon-vert uppercase tracking-widest">Gabon Cash</span>
            </div>
        </div>
        <button onclick="showSection('suivi')" class="bg-blue-600 text-white px-5 py-2 rounded-full text-[11px] font-black uppercase shadow-md active:scale-90 transition-transform">Suivre</button>
    </header>

    <main class="max-w-md mx-auto p-4 pb-24">
        
        <!-- SECTION 1 : PASSER COMMANDE -->
        <div id="sec-commander" class="section">
            <div class="bg-white rounded-[2rem] shadow-2xl p-6 border border-gray-100 overflow-hidden relative">
                <div class="absolute top-0 right-0 w-32 h-32 bg-gabon-vert opacity-5 rounded-full -mr-16 -mt-16"></div>
                
                <h2 class="text-2xl font-black mb-1 text-gray-800">Besoin de Cash ?</h2>
                <p class="text-gray-400 text-[10px] mb-8 uppercase font-bold tracking-widest">Libreville • Akanda • Owendo</p>

                <div class="space-y-5">
                    <div class="relative">
                        <label class="text-[10px] font-black text-gray-400 uppercase mb-1 block ml-1">Nom complet</label>
                        <input type="text" id="cNom" class="w-full p-4 bg-gray-50 border-2 border-gray-100 focus:border-green-500 rounded-2xl outline-none transition-all font-bold" placeholder="Ex: Jean-Marc">
                    </div>
                    
                    <div class="grid grid-cols-2 gap-4">
                        <div class="relative">
                            <label class="text-[10px] font-black text-gray-400 uppercase mb-1 block ml-1">Téléphone</label>
                            <input type="tel" id="cTel" class="w-full p-4 bg-gray-50 border-2 border-gray-100 focus:border-green-500 rounded-2xl outline-none font-bold" placeholder="077...">
                        </div>
                        <div class="relative">
                            <label class="text-[10px] font-black text-gray-400 uppercase mb-1 block ml-1">Quartier</label>
                            <input type="text" id="cLieu" class="w-full p-4 bg-gray-50 border-2 border-gray-100 focus:border-green-500 rounded-2xl outline-none font-bold" placeholder="Lieu dit">
                        </div>
                    </div>

                    <div class="relative">
                        <label class="text-[10px] font-black text-gray-400 uppercase mb-1 block ml-1">Montant à recevoir</label>
                        <div class="relative group">
                            <input type="number" id="cMontant" oninput="calculerFrais()" class="w-full p-6 bg-gray-100 border-2 border-transparent focus:bg-white focus:border-green-500 rounded-3xl outline-none font-black text-3xl text-green-700 transition-all shadow-inner" placeholder="0">
                            <span class="absolute right-6 top-7 text-gray-400 font-black">FCFA</span>
                        </div>
                    </div>

                    <!-- BLOC RÉSUMÉ -->
                    <div class="bg-gray-900 rounded-[1.5rem] p-6 text-white shadow-xl transform hover:scale-[1.02] transition-transform">
                        <div class="flex justify-between text-[10px] font-bold text-gray-400 mb-3 uppercase tracking-tighter border-b border-gray-800 pb-2">
                            <span>Service (190) + Livraison (1000)</span>
                            <span class="text-blue-400">+ 1.190 F</span>
                        </div>
                        <div class="flex justify-between items-end">
                            <span class="font-bold text-xs text-gray-400 uppercase">Total Final</span>
                            <span id="totalDisplay" class="font-black text-3xl leading-none text-white tracking-tighter">0 F</span>
                        </div>
                    </div>

                    <button onclick="validerCommande()" id="btnCommander" class="w-full bg-gabon-vert text-white font-black py-5 rounded-2xl shadow-xl active:scale-95 hover:brightness-110 transition-all text-xl mt-4 uppercase tracking-tighter">
                        Valider la Commande
                    </button>
                </div>
            </div>
            
            <div class="mt-8 flex justify-center gap-6 opacity-30 grayscale hover:grayscale-0 transition-all">
                [Logo Airtel Money] [Logo Moov Money] [Logo CT241]
            </div>
        </div>

        <!-- SECTION 2 : SUIVI GPS -->
        <div id="sec-suivi" class="section hidden">
            <div class="flex items-center justify-between mb-6">
                <button onclick="showSection('commander')" class="bg-white p-3 rounded-full shadow-md text-gray-600 active:scale-90 transition-transform">
                    <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M15 19l-7-7 7-7"></path></svg>
                </button>
                <h2 class="text-lg font-black text-gray-800 uppercase tracking-tighter">Mon Tracking</h2>
                <div class="w-10"></div>
            </div>
            
            <div class="bg-white rounded-[2rem] shadow-xl p-6 mb-4 border border-blue-50">
                <p class="text-[10px] font-black text-blue-600 uppercase mb-3 ml-1 tracking-widest">Vérifier mon statut</p>
                <div class="flex gap-2">
                    <input type="tel" id="searchTel" placeholder="Numéro 07..." class="flex-1 p-4 bg-gray-50 border-2 border-gray-100 rounded-2xl outline-none focus:border-blue-500 font-black text-lg">
                    <button onclick="rechercherCommande()" class="bg-blue-600 text-white px-6 rounded-2xl font-black text-sm active:scale-90 transition-all shadow-lg">OK</button>
                </div>
            </div>

            <div id="resultatSuivi" class="space-y-4"></div>

            <!-- CARTE GPS -->
            <div id="mapContainer" class="hidden mt-6 section">
                <div class="bg-white rounded-[2rem] p-4 shadow-xl border border-red-50 overflow-hidden">
                    <div class="flex items-center justify-between mb-3 px-2">
                        <span class="text-[10px] font-black text-red-600 uppercase flex items-center gap-1.5">
                            <span class="w-2.5 h-2.5 bg-red-500 rounded-full pulse"></span> Live GPS
                        </span>
                        <span id="livreurName" class="text-[10px] font-black text-gray-500 bg-gray-100 px-3 py-1 rounded-full uppercase tracking-tighter">Chargement...</span>
                    </div>
                    <div id="map"></div>
                </div>
            </div>
        </div>
    </main>

    <!-- FOOTER NAV -->
    <nav class="fixed bottom-0 left-0 right-0 bg-white/80 backdrop-blur-md border-t border-gray-100 p-4 flex justify-around items-center z-50">
        <button onclick="showSection('commander')" class="flex flex-col items-center gap-1 text-gabon-vert">
            <svg class="w-6 h-6" fill="currentColor" viewBox="0 0 24 24"><path d="M12 2L2 12h3v8h6v-6h2v6h6v-8h3L12 2z"/></svg>
            <span class="text-[10px] font-bold">Accueil</span>
        </button>
        <button onclick="showSection('suivi')" class="flex flex-col items-center gap-1 text-gray-400">
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z"/><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 11a3 3 0 11-6 0 3 3 0 016 0z"/></svg>
            <span class="text-[10px] font-bold">Suivi</span>
        </button>
    </nav>

    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-app.js";
        import { getDatabase, ref, push, onValue, set } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-database.js";

        const firebaseConfig = {
            apiKey: "AIzaSyAPCKRy9NTo4X8nn8YpxAbPtX8SlKj-7sQ",
            authDomain: "cashtransfert-21.firebaseapp.com",
            databaseURL: "https://cashtransfert-21-default-rtdb.firebaseio.com",
            projectId: "cashtransfert-21",
            storageBucket: "cashtransfert-21.firebasestorage.app",
            messagingSenderId: "564831743134",
            appId: "1:564831743134:web:c22a1f53707f0f1dd9df8f"
        };

        const app = initializeApp(firebaseConfig);
        const db = getDatabase(app);
        let missions = [];
        let map = null;
        let marker = null;

        window.showSection = (id) => {
            document.querySelectorAll('.section').forEach(s => s.classList.add('hidden'));
            document.getElementById('sec-' + id).classList.remove('hidden');
            window.scrollTo(0,0);
        };

        window.calculerFrais = () => {
            const m = parseFloat(document.getElementById('cMontant').value) || 0;
            document.getElementById('totalDisplay').innerText = m > 0 ? (m + 1190).toLocaleString('fr-FR') + " F" : "0 F";
        };

        window.validerCommande = async () => {
            const n = document.getElementById('cNom').value;
            const t = document.getElementById('cTel').value;
            const l = document.getElementById('cLieu').value;
            const m = parseFloat(document.getElementById('cMontant').value);

            if(!n || !t || !l || !m) return alert("⚠️ Veuillez remplir le formulaire.");
            if(m < 15000) return alert("❌ Montant minimum : 15 000 FCFA");

            const btn = document.getElementById('btnCommander');
            btn.disabled = true; btn.innerText = "CONNEXION...";

            try {
                const newRef = push(ref(db, 'missions'));
                // Champs harmonisés pour l'app COMPTABLE
                const data = {
                    id: "CT" + Math.floor(1000 + Math.random() * 9000),
                    nom: n.trim(), 
                    tel: t.trim(), 
                    lieu: l.trim(), 
                    retrait: Number(m),
                    frais: 1190,
                    total: Number(m) + 1190,
                    comLivreur: 800,
                    profitAdmin: 390,
                    benefice: 390, // Clé comptable
                    commission: 1190, // Clé comptable
                    etape: 1, 
                    livreur: "En attente", 
                    date: new Date().toLocaleDateString('fr-FR'),
                    timestamp: Date.now(),
                    source: "Web Portail"
                };
                await set(newRef, data);
                alert("🚀 Commande transmise ! Un livreur va vous appeler.");
                document.getElementById('searchTel').value = t;
                showSection('suivi');
                rechercherCommande();
            } catch (e) {
                alert("Erreur réseau"); btn.disabled = false;
            }
        };

        onValue(ref(db, 'missions'), (s) => {
            missions = s.val() ? Object.values(s.val()) : [];
            if(!document.getElementById('sec-suivi').classList.contains('hidden')) rechercherCommande();
        });

        window.rechercherCommande = () => {
            const tel = document.getElementById('searchTel').value.trim();
            const res = document.getElementById('resultatSuivi');
            const mapC = document.getElementById('mapContainer');
            if(!tel) return;

            const myMissions = missions.filter(m => m.tel && m.tel.includes(tel));
            res.innerHTML = "";

            if(myMissions.length === 0) {
                res.innerHTML = "<div class='p-8 text-center text-gray-400 font-bold bg-white rounded-3xl uppercase text-xs'>Aucune commande trouvée</div>";
                mapC.classList.add('hidden'); return;
            }

            const m = myMissions.sort((a,b) => b.timestamp - a.timestamp)[0];
            let badge = "bg-yellow-400 text-yellow-900";
            let status = "🔍 RECHERCHE EN COURS";

            if(m.etape === 2) {
                badge = "bg-blue-600 text-white pulse";
                status = "🛵 LIVREUR EN ROUTE";
                if(m.gps) {
                    document.getElementById('livreurName').innerText = m.livreur;
                    initMap(m.gps);
                }
            } else if(m.etape === 3) {
                badge = "bg-green-600 text-white";
                status = "✅ LIVRAISON TERMINÉE";
                mapC.classList.add('hidden');
            }

            res.innerHTML = `
                <div class="bg-white rounded-[2rem] p-6 shadow-xl border-2 border-gray-50 relative overflow-hidden">
                    <div class="absolute top-0 right-0 w-24 h-24 bg-blue-500 opacity-[0.03] rounded-full -mr-12 -mt-12"></div>
                    <div class="flex justify-between items-start mb-6">
                        <div>
                            <p class="text-[9px] font-black text-gray-300 uppercase tracking-widest mb-1">Réf #${m.id}</p>
                            <h3 class="text-3xl font-black text-gray-800 tracking-tighter">${m.retrait.toLocaleString()} <span class="text-xs">F</span></h3>
                        </div>
                        <span class="text-[9px] font-black px-3 py-1.5 rounded-full uppercase ${badge}">${status}</span>
                    </div>
                    <div class="space-y-3 border-t border-gray-50 pt-4">
                        <div class="flex items-center gap-3">
                            <div class="w-8 h-8 bg-gray-100 rounded-full flex items-center justify-center text-gray-400 italic font-black text-[10px]">L</div>
                            <div class="text-[10px] font-bold text-gray-400 uppercase tracking-tight">Destination : <span class="text-gray-800">${m.lieu}</span></div>
                        </div>
                        <div class="flex items-center gap-3">
                            <div class="w-8 h-8 bg-blue-100 rounded-full flex items-center justify-center text-blue-400 font-black text-[10px]">M</div>
                            <div class="text-[10px] font-bold text-gray-400 uppercase tracking-tight">Livreur : <span class="text-blue-600">${m.livreur}</span></div>
                        </div>
                    </div>
                </div>`;
        };

        function initMap(coordsStr) {
            document.getElementById('mapContainer').classList.remove('hidden');
            document.getElementById('map').style.display = 'block';
            const coords = coordsStr.split(',').map(Number);
            setTimeout(() => {
                if(!map) {
                    map = L.map('map', { zoomControl: false }).setView(coords, 16);
                    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);
                    marker = L.marker(coords).addTo(map);
                } else {
                    map.setView(coords, 16);
                    marker.setLatLng(coords);
                }
            }, 300);
        }
    </script>
</body>
</html>
