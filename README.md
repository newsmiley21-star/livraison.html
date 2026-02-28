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
