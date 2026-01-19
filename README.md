<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PalmCore ERP - Financial Edition</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;500;600;700&display=swap');
        body { font-family: 'Plus Jakarta Sans', sans-serif; background-color: #f8fafc; }
        .glass-panel { background: white; border: 1px solid #e2e8f0; }
        .nav-btn.active { background: #059669 !important; color: white !important; box-shadow: 0 10px 15px -3px rgba(5, 150, 105, 0.2); }
        .modal-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.5); display: none; align-items: center; justify-content: center; z-index: 100000; }
        .page-content { animation: fadeIn 0.3s ease-out; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        
        /* Custom Scrollbar */
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: #f1f1f1; }
        ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 10px; }
        ::-webkit-scrollbar-thumb:hover { background: #94a3b8; }
    </style>
</head>
<body class="text-slate-700">

    <!-- Modal Konfirmasi Hapus -->
    <div id="delete-modal" class="modal-overlay">
        <div class="bg-white p-8 rounded-3xl max-w-sm w-full mx-4 shadow-2xl text-center">
            <div class="text-rose-500 mb-4"><i class="fas fa-exclamation-triangle text-5xl"></i></div>
            <h3 class="text-xl font-black mb-2">Hapus Transaksi?</h3>
            <p class="text-slate-500 text-sm mb-6">Data ini akan dihapus permanen dari penyimpanan lokal browser Anda.</p>
            <div class="flex gap-3">
                <button onclick="closeDeleteModal()" class="flex-1 py-3 bg-slate-100 font-bold rounded-xl hover:bg-slate-200 transition-colors">Batal</button>
                <button id="confirm-delete-btn" class="flex-1 py-3 bg-rose-600 text-white font-bold rounded-xl hover:bg-rose-700 transition-colors">Hapus</button>
            </div>
        </div>
    </div>

    <!-- Layout Utama -->
    <div id="app-body">
        <!-- Sidebar Navigation -->
        <nav class="fixed left-0 top-0 h-full w-20 lg:w-64 bg-white border-r border-slate-200 z-50">
            <div class="p-6 flex flex-col h-full">
                <div class="flex items-center gap-3 mb-10">
                    <div class="bg-emerald-600 p-2 rounded-xl text-white shadow-lg shadow-emerald-200"><i class="fas fa-leaf text-xl"></i></div>
                    <span class="font-bold text-xl hidden lg:block text-emerald-900 tracking-tight">PalmCore<span class="text-emerald-500 font-black">ERP</span></span>
                </div>
                
                <div class="space-y-2 flex-1 overflow-y-auto pr-2">
                    <button id="nav-dashboard" onclick="navTo('dashboard')" class="nav-btn active w-full flex items-center gap-4 p-3 rounded-xl text-slate-500 transition-all hover:bg-slate-50">
                        <i class="fas fa-chart-pie w-5"></i><span class="font-bold hidden lg:block text-sm">Dashboard</span>
                    </button>
                    <button id="nav-modal" onclick="navTo('modal')" class="nav-btn w-full flex items-center gap-4 p-3 rounded-xl text-slate-500 transition-all hover:bg-slate-50">
                        <i class="fas fa-vault w-5"></i><span class="font-bold hidden lg:block text-sm">Input Modal</span>
                    </button>
                    <button id="nav-beli" onclick="navTo('beli')" class="nav-btn w-full flex items-center gap-4 p-3 rounded-xl text-slate-500 transition-all hover:bg-slate-50">
                        <i class="fas fa-balance-scale w-5"></i><span class="font-bold hidden lg:block text-sm">Beli TBS</span>
                    </button>
                    <button id="nav-jual" onclick="navTo('jual')" class="nav-btn w-full flex items-center gap-4 p-3 rounded-xl text-slate-500 transition-all hover:bg-slate-50">
                        <i class="fas fa-truck-moving w-5"></i><span class="font-bold hidden lg:block text-sm">Jual (PKS)</span>
                    </button>
                    <button id="nav-biaya" onclick="navTo('biaya')" class="nav-btn w-full flex items-center gap-4 p-3 rounded-xl text-slate-500 transition-all hover:bg-slate-50">
                        <i class="fas fa-money-bill-wave w-5"></i><span class="font-bold hidden lg:block text-sm">Biaya</span>
                    </button>
                    <button id="nav-laporan" onclick="navTo('laporan')" class="nav-btn w-full flex items-center gap-4 p-3 rounded-xl text-slate-500 transition-all hover:bg-slate-50">
                        <i class="fas fa-file-invoice-dollar w-5"></i><span class="font-bold hidden lg:block text-sm">Laporan & Keuangan</span>
                    </button>
                </div>

                <div class="pt-4 border-t border-slate-100">
                    <button onclick="clearAllData()" class="w-full flex items-center gap-4 p-3 rounded-xl text-rose-500 hover:bg-rose-50 transition-colors">
                        <i class="fas fa-trash-alt w-5"></i><span class="font-bold hidden lg:block text-xs text-left">Reset Semua Data</span>
                    </button>
                </div>
            </div>
        </nav>

        <!-- Main Content Area -->
        <main class="ml-20 lg:ml-64 p-4 lg:p-8 min-h-screen">
            
            <!-- Halaman Dashboard -->
            <div id="page-dashboard" class="page-content">
                <div class="mb-8">
                    <h1 class="text-2xl font-black text-slate-800">Ringkasan Operasional</h1>
                    <p class="text-slate-500 text-sm">Status stok dan kesehatan finansial terkini.</p>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
                    <div class="glass-panel p-6 rounded-3xl border-b-4 border-emerald-500 shadow-sm">
                        <p class="text-[10px] font-bold text-slate-400 uppercase tracking-widest mb-1">Stok Gudang (Kg)</p>
                        <h2 id="stok-val" class="text-3xl font-black text-slate-800">0</h2>
                    </div>
                    <div class="glass-panel p-6 rounded-3xl border-b-4 border-blue-500 shadow-sm">
                        <p class="text-[10px] font-bold text-slate-400 uppercase tracking-widest mb-1">Kas / Tunai Aktif</p>
                        <h2 id="cash-val" class="text-2xl font-black text-blue-600">Rp 0</h2>
                    </div>
                    <div class="glass-panel p-6 rounded-3xl border-b-4 border-amber-500 shadow-sm">
                        <p class="text-[10px] font-bold text-slate-400 uppercase tracking-widest mb-1">Total Investasi Modal</p>
                        <h2 id="modal-total-val" class="text-2xl font-black text-amber-600">Rp 0</h2>
                    </div>
                    <div class="bg-slate-900 p-6 rounded-3xl text-white shadow-xl">
                        <p class="text-[10px] font-bold text-emerald-400 uppercase tracking-widest mb-1">Laba Bersih Berjalan</p>
                        <h2 id="profit-val" class="text-2xl font-black text-emerald-400">Rp 0</h2>
                    </div>
                </div>
                
                <div class="glass-panel p-8 rounded-3xl h-[400px] shadow-sm">
                    <h3 class="font-bold text-slate-800 mb-4 flex items-center gap-2">
                        <i class="fas fa-chart-line text-emerald-500"></i> Tren Volume Transaksi (Kg)
                    </h3>
                    <canvas id="chartView"></canvas>
                </div>
            </div>

            <!-- Halaman Input Modal -->
            <div id="page-modal" class="page-content hidden">
                <div class="max-w-xl mx-auto glass-panel p-8 rounded-[2.5rem] border-t-8 border-amber-500 shadow-xl">
                    <h3 class="font-black text-2xl text-amber-900 mb-6">Input Modal Pemilik</h3>
                    <div class="space-y-6">
                        <div>
                            <label class="block text-[10px] font-black uppercase text-slate-400 mb-2">Nama Pelabur / Sumber</label>
                            <input id="m-sumber" type="text" placeholder="Contoh: Modal Awal Pemilik" class="w-full p-4 bg-slate-50 border rounded-2xl font-bold focus:ring-2 ring-amber-500 outline-none">
                        </div>
                        <div>
                            <label class="block text-[10px] font-black uppercase text-slate-400 mb-2">Nominal Modal (Rp)</label>
                            <input id="m-nominal" type="number" placeholder="0" class="w-full p-4 bg-amber-50 border-2 border-amber-100 rounded-2xl font-bold text-amber-700 focus:ring-2 ring-amber-500 outline-none">
                        </div>
                        <button onclick="simpan('MODAL')" class="w-full py-5 bg-amber-600 hover:bg-amber-700 text-white font-black rounded-2xl transition-all shadow-xl shadow-amber-200">
                            SIMPAN MODAL KE KAS
                        </button>
                    </div>
                </div>
            </div>

            <!-- Halaman Beli TBS -->
            <div id="page-beli" class="page-content hidden">
                <div class="max-w-4xl mx-auto glass-panel p-8 rounded-[2.5rem] border-t-8 border-emerald-500 shadow-xl">
                    <h3 class="font-black text-2xl text-emerald-900 mb-6">Input Pembelian TBS (Masuk)</h3>
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-10">
                        <div class="space-y-5">
                            <input id="b-nama" type="text" placeholder="Nama Pemasok / Petani" class="w-full p-4 bg-slate-50 border rounded-2xl font-bold">
                            <div class="grid grid-cols-2 gap-4">
                                <input id="b-bruto" type="number" oninput="hitungBeli()" placeholder="Brutto (Kg)" class="w-full p-4 bg-slate-50 border rounded-2xl font-bold">
                                <input id="b-tara" type="number" oninput="hitungBeli()" placeholder="Tarra (Kg)" class="w-full p-4 bg-slate-50 border rounded-2xl font-bold text-rose-600">
                            </div>
                            <div class="grid grid-cols-2 gap-4">
                                <div>
                                    <label class="text-[9px] font-black text-rose-400 uppercase ml-2">Potongan %</label>
                                    <input id="b-persen" type="number" oninput="hitungBeli()" value="3" class="w-full p-4 bg-rose-50 border-2 border-rose-100 rounded-2xl font-bold text-rose-700">
                                </div>
                                <div>
                                    <label class="text-[9px] font-black text-emerald-400 uppercase ml-2">Harga Beli /Kg</label>
                                    <input id="b-harga" type="number" oninput="hitungBeli()" placeholder="Rp" class="w-full p-4 bg-emerald-50 border-2 border-emerald-200 rounded-2xl font-bold text-emerald-700">
                                </div>
                            </div>
                        </div>
                        <div class="bg-emerald-900 rounded-[2.5rem] p-10 text-white flex flex-col justify-center text-center shadow-2xl relative overflow-hidden">
                            <div class="absolute top-0 right-0 p-8 opacity-10"><i class="fas fa-file-invoice-dollar text-9xl"></i></div>
                            <p class="text-xs font-bold text-emerald-400 mb-2 uppercase tracking-widest">Total Bayar ke Petani</p>
                            <h2 id="res-total" class="text-4xl font-black mb-4">Rp 0</h2>
                            <p class="text-sm bg-emerald-800 py-2 rounded-full inline-block px-6 mx-auto">Netto: <span id="res-netto" class="font-black">0</span> Kg</p>
                            <button onclick="simpan('BELI')" class="w-full py-5 mt-10 bg-emerald-500 hover:bg-emerald-400 rounded-2xl font-black transition-all shadow-lg shadow-emerald-950/20">
                                PROSES PEMBAYARAN
                            </button>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Halaman Jual PKS -->
            <div id="page-jual" class="page-content hidden">
                <div class="max-w-4xl mx-auto glass-panel p-8 rounded-[2.5rem] border-t-8 border-blue-500 shadow-xl">
                    <h3 class="font-black text-2xl text-blue-900 mb-6">Input Penjualan ke PKS (Keluar)</h3>
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-10">
                        <div class="space-y-5">
                            <input id="j-pabrik" type="text" placeholder="Nama Pabrik (PKS)" class="w-full p-4 bg-slate-50 border rounded-2xl font-bold">
                            <input id="j-netto" type="number" oninput="hitungJual()" placeholder="Netto Pabrik (Kg)" class="w-full p-4 bg-slate-50 border rounded-2xl font-bold">
                            <input id="j-harga" type="number" oninput="hitungJual()" placeholder="Harga Jual /Kg" class="w-full p-4 bg-blue-50 border-2 border-blue-200 rounded-2xl font-bold text-blue-700">
                        </div>
                        <div class="bg-blue-900 rounded-[2.5rem] p-10 text-white flex flex-col justify-center text-center shadow-2xl">
                            <p class="text-xs font-bold text-blue-400 mb-2 uppercase tracking-widest">Total Penerimaan Dana</p>
                            <h2 id="res-jual-total" class="text-4xl font-black">Rp 0</h2>
                            <button onclick="simpan('JUAL')" class="w-full py-5 mt-10 bg-blue-500 hover:bg-blue-400 rounded-2xl font-black transition-all">
                                KONFIRMASI PENJUALAN
                            </button>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Halaman Biaya -->
            <div id="page-biaya" class="page-content hidden">
                <div class="max-w-xl mx-auto glass-panel p-8 rounded-[2.5rem] border-t-8 border-rose-500 shadow-xl">
                    <h3 class="font-black text-2xl text-rose-900 mb-6">Input Biaya Operasional</h3>
                    <div class="space-y-6">
                        <input id="c-ket" type="text" placeholder="Keterangan (Gaji, Listrik, Solar, dll)" class="w-full p-4 bg-slate-50 border rounded-2xl font-bold">
                        <input id="c-nominal" type="number" placeholder="Nominal Biaya (Rp)" class="w-full p-4 bg-rose-50 border-2 border-rose-100 rounded-2xl font-bold text-rose-700">
                        <button onclick="simpan('BIAYA')" class="w-full py-5 bg-rose-600 hover:bg-rose-700 text-white font-black rounded-2xl transition-all shadow-xl shadow-rose-200">
                            CATAT PENGELUARAN
                        </button>
                    </div>
                </div>
            </div>

            <!-- Halaman Laporan & Keuangan -->
            <div id="page-laporan" class="page-content hidden">
                <div class="grid grid-cols-1 lg:grid-cols-2 gap-8 mb-8">
                    <!-- Laporan Laba Rugi -->
                    <div class="glass-panel p-8 rounded-[2rem] shadow-sm">
                        <div class="flex items-center justify-between mb-6">
                            <h3 class="font-black text-xl text-slate-800">Laporan Laba Rugi</h3>
                            <div class="bg-emerald-100 text-emerald-600 p-2 rounded-lg text-xs font-black">P&L</div>
                        </div>
                        <div class="space-y-4 text-sm font-bold">
                            <div class="flex justify-between border-b border-slate-50 pb-2">
                                <span class="text-slate-400">Total Penjualan TBS</span>
                                <span id="pnl-jual" class="text-blue-600">Rp 0</span>
                            </div>
                            <div class="flex justify-between border-b border-slate-50 pb-2">
                                <span class="text-slate-400">Total Pembelian (HPP)</span>
                                <span id="pnl-beli" class="text-rose-500">Rp 0</span>
                            </div>
                            <div class="flex justify-between border-b border-slate-50 pb-2 font-black text-emerald-700 bg-emerald-50 px-2 py-2 rounded-xl">
                                <span>Laba Kotor</span>
                                <span id="pnl-kotor">Rp 0</span>
                            </div>
                            <div class="flex justify-between border-b border-slate-50 pb-2">
                                <span class="text-slate-400">Biaya Operasional</span>
                                <span id="pnl-biaya" class="text-rose-500">Rp 0</span>
                            </div>
                            <div class="flex justify-between pt-4 text-2xl font-black text-slate-900">
                                <span>Laba Bersih</span>
                                <span id="pnl-bersih" class="text-emerald-600">Rp 0</span>
                            </div>
                        </div>
                    </div>

                    <!-- Laporan Perubahan Modal -->
                    <div class="glass-panel p-8 rounded-[2rem] shadow-sm">
                        <div class="flex items-center justify-between mb-6">
                            <h3 class="font-black text-xl text-slate-800">Perubahan Modal</h3>
                            <i class="fas fa-seedling text-amber-500 text-xl"></i>
                        </div>
                        <div class="space-y-4 text-sm font-bold">
                            <div class="flex justify-between border-b border-slate-50 pb-2">
                                <span class="text-slate-400">Modal Awal / Investasi</span>
                                <span id="eq-awal" class="text-amber-600">Rp 0</span>
                            </div>
                            <div class="flex justify-between border-b border-slate-50 pb-2">
                                <span class="text-slate-400">Laba Bersih Berjalan</span>
                                <span id="eq-laba" class="text-emerald-600">Rp 0</span>
                            </div>
                            <div class="flex justify-between pt-8 text-2xl font-black text-slate-900 border-t-2 border-slate-900 mt-4">
                                <span>Modal Akhir</span>
                                <span id="eq-akhir">Rp 0</span>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Tabel Riwayat -->
                <div class="flex flex-col md:flex-row justify-between items-start md:items-center mb-6 gap-4">
                    <h2 class="text-2xl font-black text-slate-800">Riwayat Semua Transaksi</h2>
                    <button onclick="exportExcel()" class="bg-emerald-600 text-white px-8 py-3 rounded-2xl font-bold hover:bg-emerald-700 transition-all flex items-center gap-2 shadow-lg shadow-emerald-100">
                        <i class="fas fa-file-excel"></i> Export Excel
                    </button>
                </div>
                
                <div class="glass-panel rounded-3xl overflow-hidden shadow-sm">
                    <div class="overflow-x-auto">
                        <table class="w-full text-sm text-left">
                            <thead class="bg-slate-50 border-b border-slate-100">
                                <tr>
                                    <th class="p-5 uppercase text-[10px] font-black text-slate-400">Tanggal</th>
                                    <th class="p-5 uppercase text-[10px] font-black text-slate-400">Tipe</th>
                                    <th class="p-5 uppercase text-[10px] font-black text-slate-400">Keterangan</th>
                                    <th class="p-5 uppercase text-[10px] font-black text-slate-400 text-right">Kg</th>
                                    <th class="p-5 uppercase text-[10px] font-black text-slate-400 text-right">Nilai (Rp)</th>
                                    <th class="p-5 uppercase text-[10px] font-black text-slate-400 text-center">Aksi</th>
                                </tr>
                            </thead>
                            <tbody id="table-log" class="divide-y divide-slate-50">
                                <!-- Data rows injected here -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </main>
    </div>

    <!-- Toast Notifikasi -->
    <div id="toast-container" class="fixed bottom-5 right-5 z-[200000] flex flex-col gap-2"></div>

    <script>
        // State Management (LocalStorage)
        let dataStore = JSON.parse(localStorage.getItem('sawit_erp_data')) || [];
        let pendingDeleteIdx = null;
        let mainChart = null;

        function saveData() {
            localStorage.setItem('sawit_erp_data', JSON.stringify(dataStore));
            renderAll();
        }

        // Navigation Controller
        function navTo(id) {
            document.querySelectorAll('.page-content').forEach(p => p.classList.add('hidden'));
            document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('active'));
            document.getElementById('page-' + id).classList.remove('hidden');
            document.getElementById('nav-' + id).classList.add('active');
            
            if(id === 'dashboard') {
                setTimeout(initChart, 50); // Small delay to ensure canvas is visible
            }
        }

        // Calculation Helpers
        function hitungBeli() {
            const bruto = parseFloat(document.getElementById('b-bruto').value) || 0;
            const tara = parseFloat(document.getElementById('b-tara').value) || 0;
            const persen = parseFloat(document.getElementById('b-persen').value) || 0;
            const harga = parseFloat(document.getElementById('b-harga').value) || 0;
            
            const murni = bruto - tara;
            const potongan = Math.round(murni * (persen / 100));
            const netto = murni - potongan;
            const total = netto * harga;
            
            document.getElementById('res-netto').innerText = netto.toLocaleString();
            document.getElementById('res-total').innerText = 'Rp ' + total.toLocaleString();
            return { netto, total };
        }

        function hitungJual() {
            const netto = parseFloat(document.getElementById('j-netto').value) || 0;
            const harga = parseFloat(document.getElementById('j-harga').value) || 0;
            const total = netto * harga;
            
            document.getElementById('res-jual-total').innerText = 'Rp ' + total.toLocaleString();
            return { netto, total };
        }

        // Notification UI
        function showToast(msg, type = 'emerald') {
            const toast = document.createElement('div');
            toast.className = `bg-${type}-600 text-white px-6 py-4 rounded-2xl shadow-2xl font-bold flex items-center gap-3 animate-bounce`;
            toast.innerHTML = `<i class="fas fa-check-circle"></i> ${msg}`;
            document.getElementById('toast-container').appendChild(toast);
            setTimeout(() => {
                toast.style.opacity = '0';
                setTimeout(() => toast.remove(), 500);
            }, 3000);
        }

        // Transaction Handler
        function simpan(tipe) {
            let entry = { tipe, ts: Date.now(), id: Date.now() };
            
            if(tipe === 'BELI') {
                const res = hitungBeli();
                if(res.total <= 0) return alert('Input tidak valid');
                entry.nama = document.getElementById('b-nama').value || 'Pemasok Umum';
                entry.netto = res.netto; 
                entry.total = res.total;
            } else if(tipe === 'JUAL') {
                const res = hitungJual();
                if(res.total <= 0) return alert('Input tidak valid');
                entry.nama = document.getElementById('j-pabrik').value || 'PKS Umum';
                entry.netto = res.netto; 
                entry.total = res.total;
            } else if(tipe === 'BIAYA') {
                const nominal = parseFloat(document.getElementById('c-nominal').value) || 0;
                if(nominal <= 0) return alert('Nominal harus lebih dari 0');
                entry.nama = document.getElementById('c-ket').value || 'Biaya Operasional';
                entry.total = nominal;
                entry.netto = 0;
            } else if(tipe === 'MODAL') {
                const nominal = parseFloat(document.getElementById('m-nominal').value) || 0;
                if(nominal <= 0) return alert('Nominal harus lebih dari 0');
                entry.nama = document.getElementById('m-sumber').value || 'Modal Pemilik';
                entry.total = nominal;
                entry.netto = 0;
            }

            dataStore.unshift(entry);
            saveData();
            showToast(`Transaksi ${tipe} berhasil dicatat`);
            
            // Clear inputs
            document.querySelectorAll('input').forEach(i => {
                if(i.id !== 'b-persen') i.value = '';
            });
            hitungBeli(); hitungJual();
        }

        // Delete Logic
        function openDeleteModal(id) {
            pendingDeleteIdx = id;
            document.getElementById('delete-modal').style.display = 'flex';
        }

        function closeDeleteModal() {
            document.getElementById('delete-modal').style.display = 'none';
        }

        document.getElementById('confirm-delete-btn').onclick = () => {
            dataStore = dataStore.filter(d => d.id !== pendingDeleteIdx);
            saveData();
            closeDeleteModal();
            showToast('Transaksi telah dihapus', 'rose');
        };

        function clearAllData() {
            if(confirm("PERINGATAN: Semua data akan dihapus permanen. Lanjutkan?")) {
                dataStore = [];
                saveData();
                showToast('Database telah direset', 'slate');
            }
        }

        // Data Rendering
        function renderAll() {
            const table = document.getElementById('table-log');
            table.innerHTML = '';
            
            let totalBeli = 0, totalJual = 0, totalBiaya = 0, totalModal = 0, stokKg = 0;
            
            dataStore.forEach(d => {
                // Aggregation
                if(d.tipe==='BELI') { totalBeli += d.total; stokKg += d.netto; }
                if(d.tipe==='JUAL') { totalJual += d.total; stokKg -= d.netto; }
                if(d.tipe==='BIAYA') { totalBiaya += d.total; }
                if(d.tipe==='MODAL') { totalModal += d.total; }

                // Row Color styling
                let typeColor = 'text-slate-600';
                if(d.tipe==='BELI') typeColor = 'text-emerald-600';
                if(d.tipe==='JUAL') typeColor = 'text-blue-600';
                if(d.tipe==='BIAYA') typeColor = 'text-rose-600';
                if(d.tipe==='MODAL') typeColor = 'text-amber-600';

                table.insertAdjacentHTML('beforeend', `
                    <tr class="hover:bg-slate-50 transition-colors">
                        <td class="p-5 text-xs font-bold text-slate-400">${new Date(d.ts).toLocaleDateString('id-ID')}</td>
                        <td class="p-5 font-black ${typeColor}">${d.tipe}</td>
                        <td class="p-5 font-bold">${d.nama}</td>
                        <td class="p-5 text-right font-medium">${d.netto > 0 ? d.netto.toLocaleString() : '-'}</td>
                        <td class="p-5 text-right font-black">Rp ${d.total.toLocaleString()}</td>
                        <td class="p-5 text-center">
                            <button onclick="openDeleteModal(${d.id})" class="text-slate-300 hover:text-rose-600 transition-colors">
                                <i class="fas fa-trash-alt"></i>
                            </button>
                        </td>
                    </tr>
                `);
            });

            // Financial Logic
            const labaKotor = totalJual - totalBeli;
            const labaBersih = labaKotor - totalBiaya;
            const kasAktif = totalModal + totalJual - totalBeli - totalBiaya;
            const modalAkhir = totalModal + labaBersih;

            // UI Updates - Dashboard
            document.getElementById('stok-val').innerText = stokKg.toLocaleString();
            document.getElementById('cash-val').innerText = 'Rp ' + kasAktif.toLocaleString();
            document.getElementById('modal-total-val').innerText = 'Rp ' + totalModal.toLocaleString();
            document.getElementById('profit-val').innerText = 'Rp ' + labaBersih.toLocaleString();

            // UI Updates - Laporan
            document.getElementById('pnl-jual').innerText = 'Rp ' + totalJual.toLocaleString();
            document.getElementById('pnl-beli').innerText = 'Rp ' + totalBeli.toLocaleString();
            document.getElementById('pnl-kotor').innerText = 'Rp ' + labaKotor.toLocaleString();
            document.getElementById('pnl-biaya').innerText = 'Rp ' + totalBiaya.toLocaleString();
            document.getElementById('pnl-bersih').innerText = 'Rp ' + labaBersih.toLocaleString();
            
            document.getElementById('eq-awal').innerText = 'Rp ' + totalModal.toLocaleString();
            document.getElementById('eq-laba').innerText = 'Rp ' + labaBersih.toLocaleString();
            document.getElementById('eq-akhir').innerText = 'Rp ' + modalAkhir.toLocaleString();
        }

        // Charts
        function initChart() {
            const ctx = document.getElementById('chartView').getContext('2d');
            
            // Get last 10 volume records
            const rawData = [...dataStore].filter(d => d.netto > 0).slice(0, 10).reverse();
            
            if(mainChart) mainChart.destroy();
            
            mainChart = new Chart(ctx, { 
                type: 'bar', 
                data: { 
                    labels: rawData.map(d => new Date(d.ts).toLocaleDateString('id-ID', {day:'2-digit', month:'short'})), 
                    datasets: [{
                        label: 'Volume Netto (Kg)', 
                        data: rawData.map(d => d.netto), 
                        backgroundColor: rawData.map(d => d.tipe === 'BELI' ? '#10b981' : '#3b82f6'), 
                        borderRadius: 6,
                        barThickness: 25
                    }] 
                },
                options: { 
                    maintainAspectRatio: false,
                    plugins: { legend: { display: false } },
                    scales: { 
                        y: { beginAtZero: true, grid: { color: '#f1f5f9' } },
                        x: { grid: { display: false } }
                    }
                }
            });
        }

        // Excel Export
        function exportExcel() {
            const exportData = dataStore.map(d => ({ 
                Tanggal: new Date(d.ts).toLocaleDateString('id-ID'), 
                Tipe: d.tipe, 
                Keterangan: d.nama, 
                Netto_Kg: d.netto || 0, 
                Nilai_Rp: d.total 
            }));
            
            const ws = XLSX.utils.json_to_sheet(exportData);
            const wb = XLSX.utils.book_new(); 
            XLSX.utils.book_append_sheet(wb, ws, "Jurnal_ERP");
            XLSX.writeFile(wb, `PalmCore_Report_${Date.now()}.xlsx`);
        }

        // Initialize App
        window.onload = () => { 
            renderAll(); 
            initChart(); 
        }
    </script>
</body>
</html>

