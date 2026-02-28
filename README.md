<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CT241 - Commander du Cash</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        :root { --gabon-vert: #009E60; --gabon-jaune: #FCD116; --gabon-bleu: #3A75C4; }
        .bg-gabon-vert { background-color: var(--gabon-vert); }
        .text-gabon-vert { color: var(--gabon-vert); }
        .border-gabon-jaune { border-color: var(--gabon-jaune); }
    </style>
</head>
<body class="bg-gray-50 font-sans">

    <!-- Header -->
    <header class="bg-white shadow-sm p-4 flex justify-between items-center border-b-4 border-gabon-jaune">
        <div class="flex items-center gap-2">
            <div class="w-10 h-10 bg-gabon-vert rounded-full flex items-center justify-center text-white font-bold">CT</div>
            <h1 class="font-black text-xl tracking-tighter">CT241 <span class="text-gabon-vert">GABON</span></h1>
        </div>
        <button onclick="showSection('suivi')" class="text-sm font-bold text-blue-600 underline">Suivre ma commande</button>
    </header>

    <main class="max-w-md mx-auto p-4">
        
        <!-- SECTION : COMMANDER -->
        <div id="sec-commander" class="section">
            <div class="bg-white rounded-2xl shadow-xl p-6 border border-gray-100">
                <h2 class="text-2xl font-bold mb-2">Besoin de Cash ?</h2>
                <p class="text-gray-500 text-sm mb-6">Faites-vous livrer votre argent partout à Libreville en moins de 30 min.</p>

                <div class="space-y-4">
                    <div>
                        <label class="block text-xs font-bold uppercase text-gray-400 mb-1">Votre Nom</label>
                        <input type="text" id="cNom" class="w-full p-3 bg-gray-50 border rounded-xl focus:ring-2 focus:ring-green-500 outline-none" placeholder="Ex: Jean Mvé">
                    </div>
                    <div>
                        <label class="block text-xs font-bold uppercase text-gray-400 mb-1">Téléphone (WhatsApp)</label>
                        <input type="tel" id="cTel" class="w-full p-3 bg-gray-50 border rounded-xl focus:ring-2 focus:ring-green-500 outline-none" placeholder="077 00 00 00">
                    </div>
                    <div>
                        <label class="block text-xs font-bold uppercase text-gray-400 mb-1">Quartier / Lieu de livraison</label>
                        <input type="text" id="cLieu" class="w-full p-3 bg-gray-50 border rounded-xl focus:ring-2 focus:ring-green-500 outline-none" placeholder="Ex: Akanda, Pharmacie">
                    </div>
                    <div>
                        <label class="block text-xs font-bold uppercase text-gray-400 mb-1">Montant à recevoir (Min 15.000 F)</label>
                        <div class="relative">
                            <input type="number" id="cMontant" oninput="calculerFrais()" class="w-full p-3 bg-gray-50 border rounded-xl focus:ring-2 focus:ring-green-500 outline-none font-bold text-lg" placeholder="0.00">
                            <span class="absolute right-4 top-4 text-gray-400 font-bold">FCFA</span>
                        </div>
                    </div>

                    <div class="bg-green-50 p-4 rounded-xl border border-green-100 mt-4">
                        <div class="flex justify-between text-sm">
                            <span>Frais de service :</span>
                            <span class="font-bold text-green-700">1.190 FCFA</span>
                        </div>
                        <div class="flex justify-between text-lg font-black mt-2 border-t border-green-200 pt-2">
                            <span>Total à payer :</span>
                            <span id="totalDisplay">0 FCFA</span>
                        </div>
                    </div>

                    <button onclick="validerCommande()" id="btnCommander" class="w-full bg-gabon-vert text-white font-black py-4 rounded-xl shadow-lg hover:opacity-90 transition-all transform active:scale-95">
                        COMMANDER MAINTENANT
                    </button>
                </div>
            </div>
            
            <div class="mt-8 grid grid-cols-3 gap-2 text-center">
                <div class="p-2 bg-white rounded-lg shadow-sm">
                    <div class="text-xl">⚡</div>
                    <div class="text-[10px] font-bold uppercase">Rapide</div>
                </div>
                <div class="p-2 bg-white rounded-lg shadow-sm">
                    <div class="text-xl">🛡️</div>
                    <div class="text-[10px] font-bold uppercase">Sécurisé</div>
                </div>
                <div class="p-2 bg-white rounded-lg shadow-sm">
                    <div class="text-xl">📍</div>
                    <div class="text-[10px] font-bold uppercase">Partout</div>
                </div>
            </div>
        </div>

        <!-- SECTION : SUIVI -->
        <div id="sec-suivi" class="section hidden">
            <button onclick="showSection('commander')" class="mb-4 text-sm font-bold text-gray-500">← Retour</button>
            <div class="bg-white rounded-2xl shadow-xl p-6">
                <h2 class="text-xl font-bold mb-4">Suivre ma livraison</h2>
                <input type="tel" id="searchTel" placeholder="Entrez votre numéro..." class="w-full p-3 border rounded-xl mb-4 outline-none focus:ring-2 focus:ring-blue-500">
                <button onclick="rechercherCommande()" class="w-full bg-blue-600 text-white font-bold py-3 rounded-xl mb-6">VÉRIFIER LE STATUT</button>

                <div id="resultatSuivi" class="space-y-4">
                    <!-- Résultat injecté ici -->
                </div>
            </div>
        </div>

    </main>

    <footer class="text-center p-8 text-gray-400 text-xs">
        &copy; 2024 CT241 GABON - Service de Livraison de Cash.
    </footer>

    <!-- Firebase SDK -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.0.1/firebase-app.js";
        import { getDatabase, ref, push, onValue } from "https://www.gstatic.com/firebasejs/11.0.1/firebase-database.js";

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
        let toutesLesMissions = [];

        // Navigation
        window.showSection = (id) => {
            document.querySelectorAll('.section').forEach(s => s.classList.add('hidden'));
            document.getElementById('sec-' + id).classList.remove('hidden');
        };

        // Calcul dynamique
        window.calculerFrais = () => {
            const m = parseFloat(document.getElementById('cMontant').value) || 0;
            if(m > 0) {
                document.getElementById('totalDisplay').innerText = (m + 1190).toLocaleString() + " FCFA";
            } else {
                document.getElementById('totalDisplay').innerText = "0 FCFA";
            }
        };

        // Commander
        window.validerCommande = () => {
            const n = document.getElementById('cNom').value;
            const t = document.getElementById('cTel').value;
            const l = document.getElementById('cLieu').value;
            const m = parseFloat(document.getElementById('cMontant').value);

            if(!n || !t || !l || !m) return alert("Veuillez remplir tous les champs.");
            if(m < 15000) return alert("Le montant minimum est de 15 000 FCFA.");

            const btn = document.getElementById('btnCommander');
            btn.disabled = true;
            btn.innerText = "ENVOI EN COURS...";

            push(ref(db, 'missions'), {
                id: "WEB-"+Date.now().toString().slice(-4),
                nom: n, tel: t, lieu: l, retrait: m,
                comLivreur: 800, profitAdmin: 390,
                etape: 1, livreur: "En attente", 
                date: new Date().toLocaleDateString('fr-FR'),
                timestamp: Date.now(),
                source: "Portail Web"
            }).then(() => {
                alert("Commande reçue ! Un livreur va vous contacter.");
                location.reload();
            });
        };

        // Suivi
        onValue(ref(db, 'missions'), (snapshot) => {
            const data = snapshot.val();
            toutesLesMissions = data ? Object.values(data) : [];
        });

        window.rechercherCommande = () => {
            const tel = document.getElementById('searchTel').value;
            const res = document.getElementById('resultatSuivi');
            res.innerHTML = "";

            const mesCommandes = toutesLesMissions.filter(m => m.tel.includes(tel));

            if(mesCommandes.length === 0) {
                res.innerHTML = "<p class='text-center text-red-500 font-bold'>Aucune commande trouvée pour ce numéro.</p>";
                return;
            }

            mesCommandes.reverse().forEach(m => {
                let statusColor = "bg-yellow-500";
                let statusText = "⏳ Recherche d'un livreur...";
                
                if(m.etape === 2) { 
                    statusColor = "bg-blue-500"; 
                    statusText = "🛵 Livreur en route ! ("+m.livreur+")"; 
                }
                if(m.etape === 3) { 
                    statusColor = "bg-green-500"; 
                    statusText = "✅ Livré avec succès"; 
                }

                res.innerHTML += `
                    <div class="border rounded-xl p-4 bg-gray-50">
                        <div class="flex justify-between items-start mb-2">
                            <span class="text-xs font-bold text-gray-400">Réf: ${m.id}</span>
                            <span class="px-2 py-1 text-[10px] text-white font-bold rounded ${statusColor}">${statusText}</span>
                        </div>
                        <div class="text-sm">
                            <b>${m.retrait.toLocaleString()} FCFA</b> livrés à <b>${m.lieu}</b>
                        </div>
                        <div class="text-[10px] text-gray-400 mt-2">${m.date}</div>
                    </div>
                `;
            });
        };

    </script>
</body>
</html>
