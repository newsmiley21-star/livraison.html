<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>CT241 - Logistique & Performance</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <script src="https://unpkg.com/html5-qrcode"></script>
    
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.1.0/firebase-app.js";
        import { getAuth, signInWithEmailAndPassword, onAuthStateChanged, signOut } from "https://www.gstatic.com/firebasejs/11.1.0/firebase-auth.js";
        import { getFirestore, collection, doc, setDoc, updateDoc, deleteDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/11.1.0/firebase-firestore.js";

        const firebaseConfig = {
            apiKey: "AIzaSyAdNSFmL45rSo9SxJJkvUPWeext0f7RX_Q",
            authDomain: "ct241-service-de-livraison.firebaseapp.com",
            projectId: "ct241-service-de-livraison",
            storageBucket: "ct241-service-de-livraison.firebasestorage.app",
            messagingSenderId: "297254676010",
            appId: "1:297254676010:web:01e3765686c8d478618553"
        };

        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        const appId = 'ct241-service-de-livraison';

        let currentUser = null;
        let userRole = null;
        let allMissions = [];
        let html5QrCode = null;
        let currentMissionForPhoto = null;

        // --- AUTH ---
        window.handleLogin = async (e) => {
            e.preventDefault();
            try { await signInWithEmailAndPassword(auth, document.getElementById('loginEmail').value.trim().toLowerCase(), document.getElementById('loginPass').value); } 
            catch { document.getElementById('loginError').classList.remove('hidden'); }
        };

        window.handleLogout = async () => { await signOut(auth); location.reload(); };

        onAuthStateChanged(auth, (user) => {
            if (user) {
                currentUser = user;
                if (assignRole(user.email)) {
                    document.getElementById('authSection').classList.add('hidden');
                    document.getElementById('appContent').classList.remove('hidden');
                    startListeners();
                    prepareNextId();
                } else { signOut(auth); }
            } else {
                document.getElementById('authSection').classList.remove('hidden');
                document.getElementById('appContent').classList.add('hidden');
            }
        });

        const assignRole = (email) => {
            const e = email.toLowerCase();
            if (e.includes('admin')) userRole = 'admin';
            else if (e.includes('relais')) userRole = 'relais';
            else if (e.includes('dispatch')) userRole = 'dispatch';
            else if (e.includes('livreur')) userRole = 'livreur';
            else return false;
            
            document.querySelectorAll('.role-section').forEach(s => s.classList.add('hidden'));
            if (userRole === 'admin') { document.querySelectorAll('.role-section').forEach(s => s.classList.remove('hidden')); document.getElementById('adminControls').classList.remove('hidden'); }
            else if (userRole === 'relais') { document.getElementById('rubrique1').classList.remove('hidden'); document.getElementById('rubrique4').classList.remove('hidden'); }
            else if (userRole === 'dispatch') { document.getElementById('rubrique2').classList.remove('hidden'); document.getElementById('rubrique4').classList.remove('hidden'); }
            else if (userRole === 'livreur') { document.getElementById('rubrique3').classList.remove('hidden'); document.getElementById('statLivreurSection').classList.remove('hidden'); }
            return true;
        };

        // --- GESTION MISSIONS ---
        const startListeners = () => {
            onSnapshot(collection(db, 'artifacts', appId, 'public', 'data', 'missions'), (snap) => {
                allMissions = snap.docs.map(doc => doc.data());
                renderUI();
                renderStats();
            });
        };

        const prepareNextId = () => {
            window.nextId = "2026-" + Math.floor(Math.random() * 900000 + 100000);
            const el = document.getElementById('displayNextId');
            if(el) el.innerText = window.nextId;
        };

        window.genererMission = async () => {
            const fields = {
                en: document.getElementById('expNom').value,
                dn: document.getElementById('destNom').value,
                dq: document.getElementById('dq').value,
                dt: document.getElementById('dt').value,
                p: parseFloat(document.getElementById('fraisLivraison').value) || 0
            };
            if (!fields.en || !fields.dt || !fields.p) return showToast("Informations manquantes", "error");
            
            const validationCode = Math.floor(1000 + Math.random() * 9000).toString();
            try {
                const now = Date.now();
                await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'missions', window.nextId), {
                    ...fields, id: window.nextId, s: 0, ca: now, cad: new Date(now).toISOString().split('T')[0], ce: currentUser.email, code: validationCode
                });
                showToast("Mission enregistrée", "success");
                prepareNextId();
            } catch (e) { showToast("Erreur", "error"); }
        };

        window.verifierCode = async () => {
            const saisie = document.getElementById('codeSaisie').value;
            const mission = allMissions.find(m => m.id === currentMissionForPhoto);
            if (!mission) return;

            if (saisie === mission.code) {
                await updateDoc(doc(db, 'artifacts', appId, 'public', 'data', 'missions', currentMissionForPhoto), {
                    s: 2, le: currentUser.email, lk: new Date().toISOString()
                });
                document.getElementById('codeModal').classList.add('hidden');
                document.getElementById('codeSaisie').value = "";
                showToast("Mission validée", "success");
            } else { alert("Code incorrect"); }
        };

        window.triggerPhoto = (id) => {
            currentMissionForPhoto = id;
            document.getElementById('codeModal').classList.remove('hidden');
        };

        // --- SCANNER / UI ---
        window.openQrScanner = () => {
            document.getElementById('qrScannerModal').classList.remove('hidden');
            html5QrCode = new Html5Qrcode("reader");
            html5QrCode.start({ facingMode: "environment" }, { fps: 10, qrbox: 250 }, (text) => {
                const missionId = text.replace('CT241-', '');
                const m = allMissions.find(x => x.id === missionId);
                if(m && m.s === 1) { 
                    html5QrCode.stop(); 
                    document.getElementById('qrScannerModal').classList.add('hidden');
                    triggerPhoto(missionId); 
                }
            });
        };

        window.renderUI = () => {
            const contDisp = document.getElementById('containerDispatch');
            const contLiv = document.getElementById('containerLivreur');
            const contArch = document.getElementById('archiveBody');
            
            if(contDisp) contDisp.innerHTML = "";
            if(contLiv) contLiv.innerHTML = "";
            if(contArch) contArch.innerHTML = "";
            
            allMissions.sort((a,b) => b.ca - a.ca).forEach(m => {
                const isAdmin = userRole === 'admin';
                if (m.s === 0 && (isAdmin || userRole === 'dispatch')) {
                     contDisp.innerHTML += `<div class="p-5 bg-white rounded-3xl mb-4">#${m.id} - ${m.dn} <button onclick="publierMission('${m.id}')" class="bg-blue-600 p-2 text-white text-[10px]">Assigner</button></div>`;
                } else if (m.s === 1 && (isAdmin || userRole === 'livreur')) {
                     contLiv.innerHTML += `<div class="p-6 bg-white border-l-8 border-amber-400 rounded-[2rem] mb-5">${m.dn} <button onclick="openQrScanner()" class="bg-black text-white p-2">Scanner</button></div>`;
                } else if (m.s === 2 && (isAdmin || userRole === 'dispatch' || userRole === 'relais')) {
                     contArch.innerHTML += `<tr class="text-[10px] text-white border-b"><td>${m.id}</td><td>${m.dn}</td><td>${isAdmin ? m.code : '***'}</td></tr>`;
                }
            });
        };

        window.showToast = (m, t) => {
            const el = document.getElementById('toast');
            el.innerText = m; el.className = `fixed bottom-10 left-1/2 -translate-x-1/2 px-8 py-4 rounded-3xl text-white font-black text-xs z-[600] ${t==='success'?'bg-emerald-600':'bg-red-600'}`;
            el.classList.remove('hidden'); setTimeout(() => el.classList.add('hidden'), 3000);
        };
    </script>
</head>
<body class="antialiased">
    <div id="toast" class="hidden"></div>
    
    <section id="authSection" class="fixed inset-0 bg-[#0f172a] flex items-center justify-center p-8 z-[1000]">
        <form onsubmit="handleLogin(event)" class="w-full max-w-sm bg-white rounded-[3.5rem] p-10 space-y-4">
            <input type="email" id="loginEmail" required class="w-full p-4 bg-slate-100 rounded-xl" placeholder="E-mail">
            <input type="password" id="loginPass" required class="w-full p-4 bg-slate-100 rounded-xl" placeholder="Mot de passe">
            <button type="submit" class="w-full bg-black text-white py-4 rounded-xl">Connexion</button>
        </form>
    </section>

    <main id="appContent" class="hidden min-h-screen p-6">
        <section id="adminControls" class="hidden bg-slate-900 p-8 rounded-[3rem] text-white">Gestion Admin</section>
        <section id="rubrique1" class="role-section hidden">
            <input type="text" id="expNom" placeholder="Expéditeur" class="w-full p-4 border rounded-xl">
            <input type="text" id="destNom" placeholder="Destinataire" class="w-full p-4 border rounded-xl">
            <input type="text" id="dq" placeholder="Quartier" class="w-full p-4 border rounded-xl">
            <input type="tel" id="dt" placeholder="Tél" class="w-full p-4 border rounded-xl">
            <input type="number" id="fraisLivraison" placeholder="Frais" class="w-full p-4 border rounded-xl">
            <button onclick="genererMission()" class="w-full bg-green-600 text-white p-4 rounded-xl">Enregistrer</button>
        </section>
        <section id="rubrique2" class="role-section hidden"><div id="containerDispatch"></div></section>
        <section id="rubrique3" class="role-section hidden"><div id="containerLivreur"></div></section>
        <section id="rubrique4" class="role-section hidden"><table><tbody id="archiveBody"></tbody></table></section>
    </main>

    <div id="codeModal" class="fixed inset-0 bg-black/90 z-[1000] hidden flex flex-col items-center justify-center">
        <input type="text" id="codeSaisie" maxlength="4" class="p-6 text-3xl text-center rounded-2xl" placeholder="0000">
        <button onclick="verifierCode()" class="bg-emerald-600 text-white p-4 mt-4 rounded-xl">Valider</button>
        <button onclick="document.getElementById('codeModal').classList.add('hidden')" class="text-white mt-4">Annuler</button>
    </div>

    <div id="qrScannerModal" class="fixed inset-0 bg-black/95 z-[900] hidden flex flex-col items-center justify-center p-8">
        <div id="reader" class="w-full max-w-sm bg-slate-800 h-64"></div>
        <button onclick="document.getElementById('qrScannerModal').classList.add('hidden'); if(html5QrCode) html5QrCode.stop();" class="mt-4 text-white">Fermer</button>
    </div>
</body>
</html>
