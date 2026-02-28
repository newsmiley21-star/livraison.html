<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>CT241 - Commander du Cash</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Leaflet pour la carte GPS -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <style>
        :root { --gabon-vert: #009E60; --gabon-jaune: #FCD116; --gabon-bleu: #3A75C4; }
        .bg-gabon-vert { background-color: var(--gabon-vert); }
        .text-gabon-vert { color: var(--gabon-vert); }
        .border-gabon-jaune { border-color: var(--gabon-jaune); }
        #map { height: 280px; width: 100%; border-radius: 16px; margin-top: 12px; display: none; z-index: 10; }
        .pulse { animation: pulse-animation 2s infinite; }
        @keyframes pulse-animation { 0% { opacity: 1; } 50% { opacity: 0.4; } 100% { opacity: 1; } }
        input::-webkit-outer-spin-button, input::-webkit-inner-spin-button { -webkit-appearance: none; margin: 0; }
    </style>
</head>
<body class="bg-gray-50 text-gray-900 font-sans">

    <!-- BARRE DE NAVIGATION -->
    <header class="bg-white sticky top-0 z-50 shadow-sm border-b-4 border-gabon-jaune p-4 flex justify-between items-center">
        <div class="flex items-center gap-2" onclick="location.reload()">
            <div class="w-10 h-10 bg-gabon-vert rounded-xl flex items-center justify-center text-white font-black shadow-sm">CT</div>
            <div>
                <h1 class="font-black text-xl leading-none">CT241</h1>
                <span class="text-[10px] font-bold text-gabon-vert uppercase tracking-widest">Gabon Cash</span>
            </div>
        </div>
        <button onclick="showSection('suivi')" class="bg-blue-50 text-blue-700 px-4 py-2 rounded-full text-xs font-black uppercase tracking-tighter">Suivi</button>
    </header>

    <main class="max-w-md mx-auto p-4 pb-20">
        
        <!-- SECTION 1 : PASSER COMMANDE -->
        <div id="sec-commander" class="section">
            <div class="bg-white rounded-3xl shadow-xl p-6 border border-gray-100">
                <h2 class="text-2xl font-black mb-1">Besoin de Cash ?</h2>
                <p class="text-gray-500 text-xs mb-6 uppercase font-bold tracking-tight">Livraison sécurisée à Libreville</p>

                <div class="space-y-4">
                    <div>
                        <label class="block text-[10px] font-black text-gray-400 uppercase mb-1 ml-1">Nom complet</label>
                        <input type="text" id="cNom" class="w-full p-4 bg-gray-50 border-2 border-transparent focus:border-green-500 rounded-2xl outline-none transition-all" placeholder="Ex: Marc Obiang">
                    </div>
                    
                    <div class="grid grid-cols-2 gap-3">
                        <div>
                            <label class="block text-[10px] font-black text-gray-400 uppercase mb-1 ml-1">Téléphone</label>
                            <input type="tel" id="cTel" class="w-full p-4 bg-gray-50 border-2 border-transparent focus:border-green-500 rounded-2xl outline-none" placeholder="07...">
                        </div>
                        <div>
                            <label class="block text-[10px] font-black text-gray-400 uppercase mb-1 ml-1">Quartier</label>
                            <input type="text" id="cLieu" class="w-full p-4 bg-gray-50 border-2 border-transparent focus:border-green-500 rounded-2xl outline-none" placeholder="Ex: Nzeng-Ayong">
                        </div>
                    </div>

                    <div>
                        <label class="block text-[10px] font-black text-gray-400 uppercase mb-1 ml-1">Montant souhaité</label>
                        <div class="relative">
                            <input type="number" id="cMontant" oninput="calculerFrais()" class="w-full p-5 bg-gray-50 border-2 border-transparent focus:border-green-500 rounded-2xl outline-none font-black text-2xl text-green-700" placeholder="0">
                            <span class="absolute right-4 top-5 text-gray-400 font-bold">FCFA</span>
                        </div>
                    </div>

                    <!-- RÉSUMÉ FINANCIER -->
                    <div class="bg-blue-600 rounded-2xl p-5 text-white shadow-lg">
                        <div class="flex justify-between text-[10px] font-bold opacity-80 mb-1 uppercase">
                            <span>Frais de service (190 + 1000)</span>
                            <span>1.190 F</span>
                        </div>
                        <div class="flex justify-between items-end">
                            <span class="font-bold text-sm">TOTAL À PAYER :</span>
                            <span id="totalDisplay" class="font-black text-2xl leading-none">0 F</span>
                        </div>
                    </div>

                    <button onclick="validerCommande()" id="btnCommander" class="w-full bg-gabon-vert text-white font-black py-5 rounded-2xl shadow-xl active:scale-95 transition-transform text-lg mt-4">
                        COMMANDER MAINTENANT
                    </button>
                </div>
            </div>
            
            <p class="text-center text-[10px] text-gray-400 mt-6 font-medium">Service disponible de 08h à 20h — Libreville & Akanda</p>
        </div>

        <!-- SECTION 2 : SUIVI GPS -->
        <div id="sec-suivi" class="section hidden">
            <button onclick="showSection('commander')" class="mb-4 text-xs font-black text-gray-400 uppercase flex items-center gap-2">
                <span class="text-lg">←</span> Retour
            </button>
            
            <div class="bg-white rounded-3xl shadow-xl p-6 mb-6">
                <h2 class="text-xl font-black mb-4">Suivre mon Cash</h2>
                <div class="flex gap-2 mb-2">
                    <input type="tel" id="searchTel" placeholder="Votre numéro de téléphone" class="flex-1 p-4 bg-gray-50 border rounded-2xl outline-none focus:border-blue-500 font-bold">
                    <button onclick="rechercherCommande()" class="bg-blue-600 text-white px-6 rounded-2xl font-black uppercase text-xs">OK</button>
                </div>
                <p class="text-[9px] text-gray-400 font-bold ml-1 uppercase">Entrez le numéro utilisé lors de la commande</p>
            </div>

            <div id="resultatSuivi" class="space-y-4">
                <!-- Les détails de la commande s'affichent ici -->
            </div>

            <!-- CONTENEUR CARTE GPS -->
            <div id="mapContainer" class="hidden mt-6">
                <div class="flex items-center justify-between mb-2 px-1">
                    <span class="text-[10px] font-black text-blue-600 uppercase flex items-center gap-1">
                        <span class="w-2 h-2 bg-red-500 rounded-full pulse"></span> Position du livreur
                    </span>
                    <span id="livreurName" class="text-[10px] font-black text-gray-500">MOUKAGNI (MOTO)</span>
                </div>
                <div id="map"></div>
                <p class="text-center text-[9px] text-gray-400 mt-3 italic">La carte se met à jour à chaque étape de validation du livreur.</p>
            </div>
        </div>
    </main>

    <!-- SCRIPTS -->
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
        let toutesMissions = [];
        let mapInstance = null;
        let markerInstance = null;

        // Navigation entre sections
        window.showSection = (id) => {
            document.querySelectorAll('.section').forEach(s => s.classList.add('hidden'));
            document.getElementById('sec-' + id).classList.remove('hidden');
            window.scrollTo(0,0);
        };

        // Calcul dynamique
        window.calculerFrais = () => {
            const m = parseFloat(document.getElementById('cMontant').value) || 0;
            document.getElementById('totalDisplay').innerText = m > 0 ? (m + 1190).toLocaleString() + " F" : "0 F";
        };

        // Enregistrement de la commande
        window.validerCommande = async () => {
            const n = document.getElementById('cNom').value;
            const t = document.getElementById('cTel').value;
            const l = document.getElementById('cLieu').value;
            const m = parseFloat(document.getElementById('cMontant').value);

            if(!n || !t || !l || !m) return alert("Veuillez remplir tous les champs.");
            if(m < 15000) return alert("Le montant minimum est de 15 000 FCFA.");

            const btn = document.getElementById('btnCommander');
            btn.disabled = true;
            btn.innerText = "ENVOI...";

            try {
                const nouvelleRef = push(ref(db, 'missions'));
                // Assurer la compatibilité avec l'app comptable en envoyant des nombres et les bons noms de champs
                await set(nouvelleRef, {
                    id: "CT" + Date.now().toString().slice(-4),
                    nom: n, 
                    tel: t, 
                    lieu: l, 
                    retrait: Number(m),
                    frais: 1190,
                    comLivreur: 800, 
                    profitAdmin: 390,
                    etape: 1, 
                    livreur: "En attente", 
                    date: new Date().toLocaleDateString('fr-FR'),
                    timestamp: Date.now(),
                    source: "Web Client"
                });
                alert("Commande validée ! Un livreur va vous appeler.");
                document.getElementById('searchTel').value = t;
                showSection('suivi');
                rechercherCommande();
            } catch (e) {
                alert("Erreur de connexion.");
                btn.disabled = false;
                btn.innerText = "RÉESSAYER";
            }
        };

        // Écoute des données
        onValue(ref(db, 'missions'), (snapshot) => {
            const data = snapshot.val();
            toutesMissions = data ? Object.values(data) : [];
            if(!document.getElementById('sec-suivi').classList.contains('hidden')) {
                rechercherCommande();
            }
        });

        // Recherche et Tracking
        window.rechercherCommande = () => {
            const tel = document.getElementById('searchTel').value.trim();
            const res = document.getElementById('resultatSuivi');
            const mapCont = document.getElementById('mapContainer');
            if(!tel) return;

            const mesCommandes = toutesMissions.filter(m => m.tel && m.tel.includes(tel));
            res.innerHTML = "";

            if(mesCommandes.length === 0) {
                res.innerHTML = "<div class='text-center py-10 text-gray-400 font-bold uppercase text-xs bg-white rounded-3xl'>Aucune commande trouvée</div>";
                mapCont.classList.add('hidden');
                return;
            }

            const m = mesCommandes.sort((a,b) => b.timestamp - a.timestamp)[0];
            
            let badge = "bg-yellow-400 text-yellow-900";
            let statusLabel = "⏳ RECHERCHE LIVREUR";
            
            if(m.etape === 2) {
                badge = "bg-blue-600 text-white pulse";
                statusLabel = "🛵 LIVREUR EN ROUTE";
                if(m.gps) {
                    document.getElementById('livreurName').innerText = m.livreur;
                    chargerCarte(m.gps);
                }
            } else if(m.etape === 3) {
                badge = "bg-green-600 text-white";
                statusLabel = "✅ LIVRAISON TERMINÉE";
                mapCont.classList.add('hidden');
            }

            res.innerHTML = `
                <div class="bg-white rounded-3xl p-6 shadow-md border-2 border-gray-50">
                    <div class="flex justify-between items-start mb-4">
                        <div>
                            <span class="text-[10px] font-black text-gray-300 uppercase tracking-widest">Référence #${m.id}</span>
                            <h3 class="text-2xl font-black text-gray-800">${m.retrait.toLocaleString()} F</h3>
                        </div>
                        <span class="text-[9px] font-black px-3 py-1.5 rounded-full uppercase ${badge}">${statusLabel}</span>
                    </div>
                    <div class="space-y-2 border-t pt-4 border-gray-50">
                        <div class="text-xs font-bold text-gray-500 uppercase flex items-center gap-2">📍 Destination : <span class="text-gray-800">${m.lieu}</span></div>
                        <div class="text-xs font-bold text-gray-500 uppercase flex items-center gap-2">👤 Client : <span class="text-gray-800">${m.nom}</span></div>
                        <div class="text-xs font-bold text-gray-500 uppercase flex items-center gap-2">🛵 Livreur : <span class="text-blue-600">${m.livreur}</span></div>
                    </div>
                </div>`;
        };

        function chargerCarte(gpsStr) {
            const mapCont = document.getElementById('mapContainer');
            const mapDiv = document.getElementById('map');
            mapCont.classList.remove('hidden');
            mapDiv.style.display = 'block';

            const coords = gpsStr.split(',').map(c => parseFloat(c));
            
            setTimeout(() => {
                if (!mapInstance) {
                    mapInstance = L.map('map', { zoomControl: false }).setView(coords, 16);
                    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(mapInstance);
                    markerInstance = L.marker(coords).addTo(mapInstance)
                        .bindPopup("<b style='font-size:11px'>CT241 LIVRAISON</b>")
                        .openPopup();
                } else {
                    mapInstance.setView(coords, 16);
                    markerInstance.setLatLng(coords);
                }
            }, 100);
        }
    </script>
</body>
</html>
