
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>RJ SHOP - Dashboard Produksi & Gudang</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- React & ReactDOM -->
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <!-- Babel for JSX -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <!-- Chart.js -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;500;600;700;800&display=swap');
        
        body {
            font-family: 'Plus Jakarta Sans', sans-serif;
        }

        .custom-scrollbar::-webkit-scrollbar {
            width: 6px;
        }
        .custom-scrollbar::-webkit-scrollbar-track {
            background: transparent;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb {
            background: #e2e8f0;
            border-radius: 10px;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb:hover {
            background: #cbd5e1;
        }

        .fade-in {
            animation: fadeIn 0.3s ease-in-out;
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }

        .notification-pulse {
            animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
        }

        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: .5; }
        }
    </style>
</head>
<body class="bg-slate-50">
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useMemo, useRef } = React;
        
        // --- Icons Mapping for Lucide ---
        const Icon = ({ name, size = 20, className = "" }) => {
            const LucideIcon = lucide.icons[name];
            if (!LucideIcon) return null;
            return <i data-lucide={name} className={className} style={{ width: size, height: size }}></i>;
        };

        // --- Mock Data ---
        const INITIAL_STATS = [
            { label: 'Total Produk', value: '1,284', icon: 'Package', color: 'text-blue-600', bg: 'bg-blue-100' },
            { label: 'Produksi Aktif', value: '42 Lot', icon: 'Scissors', color: 'text-amber-600', bg: 'bg-amber-100' },
            { label: 'Stok Bahan', value: '850m', icon: 'Warehouse', color: 'text-emerald-600', bg: 'bg-emerald-100' },
            { label: 'Omset Bulan Ini', value: 'Rp 145jt', icon: 'DollarSign', color: 'text-indigo-600', bg: 'bg-indigo-100' },
        ];

        const INITIAL_PRODUCTION = [
            { id: 'SPK-001', item: 'T-Shirt Cotton Combed 30s', qty: 500, stage: 'Jahit', progress: 65, status: 'On Progress', deadline: '2023-12-25' },
            { id: 'SPK-002', item: 'Hoodie Oversize Heavyweight', qty: 200, stage: 'Sablon', progress: 30, status: 'Delayed', deadline: '2023-12-22' },
            { id: 'SPK-003', item: 'Chino Pants Slim Fit', qty: 350, stage: 'Packing', progress: 95, status: 'Urgent', deadline: '2023-12-20' },
            { id: 'SPK-004', item: 'Kemeja Flanel Premium', qty: 150, stage: 'Cutting', progress: 10, status: 'On Progress', deadline: '2023-12-28' },
        ];

        const INITIAL_STOCK = [
            { id: 1, name: 'Kain Cotton Combed 30s Black', type: 'Bahan Baku', qty: 120, unit: 'Roll', status: 'Safe' },
            { id: 2, name: 'Benang Polyester Putih', type: 'Tambahan', qty: 15, unit: 'Box', status: 'Low' },
            { id: 3, name: 'Label Satin Brand A', type: 'Aksesoris', qty: 50, unit: 'Pack', status: 'Out of Stock' },
        ];

        // --- Components ---
        const SidebarItem = ({ icon, label, active, onClick }) => (
            <button 
                onClick={onClick}
                className={`w-full flex items-center space-x-3 px-4 py-3 rounded-xl transition-all duration-200 ${
                    active 
                        ? 'bg-blue-600 text-white shadow-lg shadow-blue-200' 
                        : 'text-slate-500 hover:bg-slate-100 hover:text-slate-800'
                }`}
            >
                <Icon name={icon} size={20} />
                <span className="font-medium text-sm">{label}</span>
            </button>
        );

        const Card = ({ children, title, extra }) => (
            <div className="bg-white rounded-2xl p-6 shadow-sm border border-slate-100 h-full">
                {(title || extra) && (
                    <div className="flex justify-between items-center mb-6">
                        {title && <h3 className="font-bold text-slate-800 text-lg">{title}</h3>}
                        {extra}
                    </div>
                )}
                {children}
            </div>
        );

        const Badge = ({ status }) => {
            const styles = {
                'On Progress': 'bg-blue-100 text-blue-700',
                'Delayed': 'bg-red-100 text-red-700',
                'Urgent': 'bg-orange-100 text-orange-700',
                'Safe': 'bg-emerald-100 text-emerald-700',
                'Low': 'bg-amber-100 text-amber-700',
                'Out of Stock': 'bg-red-100 text-red-700',
            };
            return (
                <span className={`px-3 py-1 rounded-full text-[10px] font-bold uppercase tracking-wider ${styles[status] || 'bg-slate-100 text-slate-600'}`}>
                    {status}
                </span>
            );
        };

        const App = () => {
            const [activeTab, setActiveTab] = useState('Dashboard');
            const [isSidebarOpen, setSidebarOpen] = useState(false);
            const [isLoggedIn, setIsLoggedIn] = useState(false);
            const [notifications, setNotifications] = useState([
                { id: 1, text: "Stok Benang sisa sedikit!", type: "alert", time: "5 mnt lalu" },
                { id: 2, text: "SPK-003 masuk tahap packing", type: "info", time: "10 mnt lalu" }
            ]);
            const [showNotifPanel, setShowNotifPanel] = useState(false);
            const [showSettingsPanel, setShowSettingsPanel] = useState(false);

            // Refs for charts to prevent re-initialization issues
            const lineChartRef = useRef(null);
            const doughnutChartRef = useRef(null);

            useEffect(() => {
                lucide.createIcons();
            }, [activeTab, isLoggedIn, isSidebarOpen, showNotifPanel, showSettingsPanel]);

            useEffect(() => {
                if (isLoggedIn && activeTab === 'Dashboard') {
                    const ctxLine = document.getElementById('lineChart');
                    const ctxDoughnut = document.getElementById('doughnutChart');
                    
                    if(ctxLine) {
                        if(lineChartRef.current) lineChartRef.current.destroy();
                        lineChartRef.current = new Chart(ctxLine, {
                            type: 'line',
                            data: {
                                labels: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun'],
                                datasets: [{
                                    label: 'Produksi',
                                    data: [1200, 1900, 1500, 2100, 2400, 1800],
                                    borderColor: '#2563eb',
                                    backgroundColor: 'rgba(37, 99, 235, 0.1)',
                                    fill: true,
                                    tension: 0.4
                                }]
                            },
                            options: { maintainAspectRatio: false, plugins: { legend: { display: false } } }
                        });
                    }

                    if(ctxDoughnut) {
                        if(doughnutChartRef.current) doughnutChartRef.current.destroy();
                        doughnutChartRef.current = new Chart(ctxDoughnut, {
                            type: 'doughnut',
                            data: {
                                labels: ['T-Shirt', 'Hoodie', 'Pants', 'Shirt'],
                                datasets: [{
                                    data: [45, 25, 20, 10],
                                    backgroundColor: ['#2563eb', '#f59e0b', '#10b981', '#6366f1'],
                                    borderWidth: 0,
                                }]
                            },
                            options: { maintainAspectRatio: false }
                        });
                    }
                }
            }, [isLoggedIn, activeTab]);

            const handleTabChange = (tab) => {
                setActiveTab(tab);
                setSidebarOpen(false);
            };

            if (!isLoggedIn) {
                return (
                    <div className="min-h-screen bg-slate-50 flex items-center justify-center p-4">
                        <div className="w-full max-w-md bg-white rounded-3xl shadow-xl overflow-hidden fade-in">
                            <div className="p-8">
                                <div className="flex justify-center mb-6">
                                    <div className="w-16 h-16 bg-blue-600 rounded-2xl flex items-center justify-center shadow-lg transform rotate-3">
                                        <Icon name="ShoppingBag" size={32} className="text-white" />
                                    </div>
                                </div>
                                <h1 className="text-2xl font-bold text-center text-slate-800 mb-2">RJ SHOP Management</h1>
                                <p className="text-center text-slate-500 mb-8">Kelola produksi & gudang secara realtime</p>
                                
                                <form className="space-y-4" onSubmit={(e) => { e.preventDefault(); setIsLoggedIn(true); }}>
                                    <div>
                                        <label className="block text-sm font-medium text-slate-700 mb-1">Email / Username</label>
                                        <input type="text" className="w-full px-4 py-3 rounded-xl border border-slate-200 focus:ring-2 focus:ring-blue-500 outline-none" placeholder="admin@rjshop.id" defaultValue="admin@rjshop.id" />
                                    </div>
                                    <div>
                                        <label className="block text-sm font-medium text-slate-700 mb-1">Password</label>
                                        <input type="password" className="w-full px-4 py-3 rounded-xl border border-slate-200 focus:ring-2 focus:ring-blue-500 outline-none" placeholder="••••••••" defaultValue="password" />
                                    </div>
                                    <button className="w-full bg-blue-600 text-white py-3 rounded-xl font-bold hover:bg-blue-700 transition-colors shadow-lg shadow-blue-100">Login Sekarang</button>
                                </form>
                            </div>
                        </div>
                    </div>
                );
            }

            return (
                <div className="min-h-screen flex bg-slate-50 text-slate-800">
                    {/* Sidebar Desktop & Mobile Overlay */}
                    <aside className={`fixed lg:static inset-y-0 left-0 z-50 w-72 bg-white border-r border-slate-100 transform transition-transform duration-300 lg:translate-x-0 ${isSidebarOpen ? 'translate-x-0' : '-translate-x-full'}`}>
                        <div className="h-full flex flex-col p-6">
                            <div className="flex items-center justify-between mb-10 px-2">
                                <div className="flex items-center space-x-3">
                                    <div className="w-10 h-10 bg-blue-600 rounded-xl flex items-center justify-center text-white">
                                        <Icon name="ShoppingBag" size={20} />
                                    </div>
                                    <span className="text-xl font-black tracking-tight text-slate-800 uppercase">RJ<span className="text-blue-600"> SHOP</span></span>
                                </div>
                                <button onClick={() => setSidebarOpen(false)} className="lg:hidden text-slate-400">
                                    <Icon name="X" size={24} />
                                </button>
                            </div>

                            <nav className="flex-1 space-y-2 overflow-y-auto pr-2 custom-scrollbar">
                                <p className="text-[10px] font-bold text-slate-400 uppercase tracking-widest px-4 mb-2">Main Menu</p>
                                <SidebarItem icon="LayoutDashboard" label="Dashboard" active={activeTab === 'Dashboard'} onClick={() => handleTabChange('Dashboard')} />
                                <SidebarItem icon="Scissors" label="Produksi" active={activeTab === 'Produksi'} onClick={() => handleTabChange('Produksi')} />
                                <SidebarItem icon="Warehouse" label="Gudang" active={activeTab === 'Gudang'} onClick={() => handleTabChange('Gudang')} />
                                <SidebarItem icon="Package" label="Master Produk" active={activeTab === 'Produk'} onClick={() => handleTabChange('Produk')} />
                                <SidebarItem icon="ShoppingCart" label="Penjualan" active={activeTab === 'Penjualan'} onClick={() => handleTabChange('Penjualan')} />
                                
                                <p className="text-[10px] font-bold text-slate-400 uppercase tracking-widest px-4 mt-6 mb-2">Operational</p>
                                <SidebarItem icon="DollarSign" label="Keuangan" active={activeTab === 'Keuangan'} onClick={() => handleTabChange('Keuangan')} />
                                <SidebarItem icon="Users" label="Karyawan" active={activeTab === 'Karyawan'} onClick={() => handleTabChange('Karyawan')} />
                                <SidebarItem icon="BarChart3" label="Laporan" active={activeTab === 'Laporan'} onClick={() => handleTabChange('Laporan')} />
                                <SidebarItem icon="Settings" label="Pengaturan" active={activeTab === 'Pengaturan'} onClick={() => handleTabChange('Pengaturan')} />
                            </nav>

                            <div className="mt-auto pt-6 border-t border-slate-100">
                                <div className="flex items-center space-x-3 px-4 py-2 bg-slate-50 rounded-2xl">
                                    <div className="w-10 h-10 rounded-full bg-blue-600 flex items-center justify-center text-white font-bold">R</div>
                                    <div className="flex-1 overflow-hidden text-xs">
                                        <p className="font-bold text-slate-800 truncate">Admin RJ</p>
                                        <p className="text-slate-400 truncate">Owner Account</p>
                                    </div>
                                    <button onClick={() => setIsLoggedIn(false)} className="text-slate-400 hover:text-red-500 transition-colors">
                                        <Icon name="LogOut" size={18} />
                                    </button>
                                </div>
                            </div>
                        </div>
                    </aside>

                    {/* Main Content */}
                    <main className="flex-1 flex flex-col min-w-0 h-screen overflow-hidden relative">
                        {/* Header */}
                        <header className="bg-white border-b border-slate-100 px-6 py-4 flex items-center justify-between sticky top-0 z-40">
                            <div className="flex items-center space-x-4">
                                <button onClick={() => setSidebarOpen(true)} className="lg:hidden p-2 hover:bg-slate-100 rounded-lg">
                                    <Icon name="Menu" size={24} />
                                </button>
                                <div className="hidden md:flex items-center bg-slate-50 border border-slate-100 rounded-xl px-4 py-2 w-80">
                                    <Icon name="Search" size={18} className="text-slate-400" />
                                    <input type="text" placeholder="Cari data realtime..." className="bg-transparent border-none outline-none ml-2 w-full text-sm" />
                                </div>
                            </div>
                            <div className="flex items-center space-x-2">
                                <div className="relative">
                                    <button 
                                        onClick={() => {setShowNotifPanel(!showNotifPanel); setShowSettingsPanel(false);}} 
                                        className={`p-2 text-slate-500 hover:bg-slate-100 rounded-xl transition-all ${showNotifPanel ? 'bg-blue-50 text-blue-600' : ''}`}
                                    >
                                        <Icon name="Bell" size={20} />
                                        <span className="absolute top-2 right-2 w-2 h-2 bg-red-500 rounded-full border-2 border-white notification-pulse"></span>
                                    </button>
                                    {/* Notif Panel */}
                                    {showNotifPanel && (
                                        <div className="absolute right-0 mt-3 w-80 bg-white rounded-2xl shadow-2xl border border-slate-100 z-50 fade-in overflow-hidden">
                                            <div className="p-4 border-b border-slate-50 bg-slate-50/50 flex justify-between items-center">
                                                <h4 className="font-bold text-sm">Notifikasi Terkini</h4>
                                                <span className="text-[10px] text-blue-600 font-bold bg-blue-100 px-2 py-0.5 rounded-full">2 Baru</span>
                                            </div>
                                            <div className="max-h-64 overflow-y-auto custom-scrollbar">
                                                {notifications.map(n => (
                                                    <div key={n.id} className="p-4 border-b border-slate-50 hover:bg-slate-50 cursor-pointer transition-colors">
                                                        <p className="text-xs font-semibold text-slate-800">{n.text}</p>
                                                        <p className="text-[10px] text-slate-400 mt-1">{n.time}</p>
                                                    </div>
                                                ))}
                                            </div>
                                            <button className="w-full py-3 text-xs font-bold text-blue-600 hover:bg-blue-50">Tandai semua telah dibaca</button>
                                        </div>
                                    )}
                                </div>
                                <div className="relative">
                                    <button 
                                        onClick={() => {setShowSettingsPanel(!showSettingsPanel); setShowNotifPanel(false);}} 
                                        className={`p-2 text-slate-500 hover:bg-slate-100 rounded-xl transition-all ${showSettingsPanel ? 'bg-blue-50 text-blue-600' : ''}`}
                                    >
                                        <Icon name="Settings" size={20} />
                                    </button>
                                    {/* Settings Panel */}
                                    {showSettingsPanel && (
                                        <div className="absolute right-0 mt-3 w-56 bg-white rounded-2xl shadow-2xl border border-slate-100 z-50 fade-in overflow-hidden">
                                            <div className="p-4 border-b border-slate-50">
                                                <h4 className="font-bold text-sm">Aksi Cepat</h4>
                                            </div>
                                            <div className="p-2">
                                                <button className="w-full text-left px-4 py-2 text-xs font-medium hover:bg-slate-50 rounded-lg flex items-center space-x-2">
                                                    <Icon name="User" size={14}/> <span>Profil Saya</span>
                                                </button>
                                                <button className="w-full text-left px-4 py-2 text-xs font-medium hover:bg-slate-50 rounded-lg flex items-center space-x-2">
                                                    <Icon name="Lock" size={14}/> <span>Keamanan</span>
                                                </button>
                                                <button className="w-full text-left px-4 py-2 text-xs font-medium text-red-500 hover:bg-red-50 rounded-lg flex items-center space-x-2 mt-1">
                                                    <Icon name="LogOut" size={14}/> <span>Keluar</span>
                                                </button>
                                            </div>
                                        </div>
                                    )}
                                </div>
                            </div>
                        </header>

                        <div className="flex-1 overflow-y-auto p-4 md:p-8 custom-scrollbar">
                            <div className="max-w-7xl mx-auto space-y-8 fade-in">
                                <div className="flex flex-col md:flex-row md:items-center justify-between gap-4">
                                    <div>
                                        <h2 className="text-2xl font-bold text-slate-800 tracking-tight">Overview {activeTab}</h2>
                                        <p className="text-slate-500 text-sm">Monitoring performa RJ SHOP hari ini.</p>
                                    </div>
                                    <div className="flex items-center space-x-3">
                                        <button className="flex items-center space-x-2 bg-blue-600 text-white px-5 py-2.5 rounded-xl text-sm font-bold shadow-lg hover:bg-blue-700 transition-all">
                                            <Icon name="Plus" size={18} />
                                            <span>Tambah Data</span>
                                        </button>
                                    </div>
                                </div>

                                {activeTab === 'Dashboard' ? (
                                    <>
                                        <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6">
                                            {INITIAL_STATS.map((stat, i) => (
                                                <div key={i} className="bg-white p-6 rounded-2xl border border-slate-100 shadow-sm hover:shadow-md transition-all group cursor-pointer">
                                                    <div className="flex items-center justify-between mb-4">
                                                        <div className={`${stat.bg} ${stat.color} p-3 rounded-xl`}><Icon name={stat.icon} size={24} /></div>
                                                        <span className="text-emerald-500 text-[10px] font-black bg-emerald-50 px-2 py-1 rounded-lg">+12.5%</span>
                                                    </div>
                                                    <p className="text-slate-500 text-xs font-bold uppercase tracking-wider">{stat.label}</p>
                                                    <h4 className="text-2xl font-black text-slate-800">{stat.value}</h4>
                                                </div>
                                            ))}
                                        </div>

                                        <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
                                            <div className="lg:col-span-2">
                                                <Card title="Tren Produksi Mingguan">
                                                    <div className="h-64"><canvas id="lineChart"></canvas></div>
                                                </Card>
                                            </div>
                                            <div>
                                                <Card title="Penjualan Produk">
                                                    <div className="h-64"><canvas id="doughnutChart"></canvas></div>
                                                </Card>
                                            </div>
                                        </div>

                                        <div className="grid grid-cols-1 lg:grid-cols-2 gap-6 pb-12">
                                            <Card title="Status Produksi Realtime">
                                                <div className="overflow-x-auto">
                                                    <table className="w-full text-left">
                                                        <thead>
                                                            <tr className="border-b border-slate-50 text-slate-400 text-[10px] uppercase font-black">
                                                                <th className="pb-3">SPK / Item</th>
                                                                <th className="pb-3">Status</th>
                                                                <th className="pb-3">Progress</th>
                                                            </tr>
                                                        </thead>
                                                        <tbody className="divide-y divide-slate-50">
                                                            {INITIAL_PRODUCTION.map((prod) => (
                                                                <tr key={prod.id} className="text-sm hover:bg-slate-50 transition-colors">
                                                                    <td className="py-4">
                                                                        <p className="font-bold text-slate-800">{prod.id}</p>
                                                                        <p className="text-xs text-slate-400 truncate max-w-[150px]">{prod.item}</p>
                                                                    </td>
                                                                    <td className="py-4"><Badge status={prod.status} /></td>
                                                                    <td className="py-4">
                                                                        <div className="w-24 bg-slate-100 h-1.5 rounded-full overflow-hidden">
                                                                            <div className="bg-blue-600 h-full transition-all duration-1000" style={{ width: `${prod.progress}%` }}></div>
                                                                        </div>
                                                                        <span className="text-[10px] font-bold text-slate-400 mt-1 inline-block">{prod.progress}%</span>
                                                                    </td>
                                                                </tr>
                                                            ))}
                                                        </tbody>
                                                    </table>
                                                </div>
                                            </Card>
                                            <Card title="Status Stok Gudang">
                                                <div className="space-y-4">
                                                    {INITIAL_STOCK.map((item) => (
                                                        <div key={item.id} className="flex items-center p-3 rounded-xl border border-slate-50 hover:border-blue-100 transition-all cursor-pointer">
                                                            <div className="w-10 h-10 bg-slate-50 rounded-lg flex items-center justify-center text-slate-400 mr-3 border border-slate-100"><Icon name="Box" size={18} /></div>
                                                            <div className="flex-1 min-w-0">
                                                                <p className="text-sm font-bold text-slate-800 truncate">{item.name}</p>
                                                                <p className="text-[10px] text-slate-400 font-medium">Sisa {item.qty} {item.unit} • {item.type}</p>
                                                            </div>
                                                            <Badge status={item.status} />
                                                        </div>
                                                    ))}
                                                </div>
                                            </Card>
                                        </div>
                                    </>
                                ) : (
                                    <div className="h-96 flex flex-col items-center justify-center bg-white rounded-3xl border border-dashed border-slate-200 fade-in">
                                        <div className="w-20 h-20 bg-blue-50 rounded-full flex items-center justify-center text-blue-600 mb-4 shadow-inner">
                                            <Icon name={
                                                activeTab === 'Produksi' ? 'Scissors' : 
                                                activeTab === 'Gudang' ? 'Warehouse' : 
                                                activeTab === 'Produk' ? 'Package' : 
                                                activeTab === 'Penjualan' ? 'ShoppingCart' : 
                                                activeTab === 'Keuangan' ? 'DollarSign' : 'Settings'
                                            } size={40} />
                                        </div>
                                        <h3 className="text-xl font-bold text-slate-800">Modul {activeTab} RJ SHOP</h3>
                                        <p className="text-slate-400 text-sm mt-2 max-w-xs text-center">Menyiapkan data realtime untuk modul ini. Silakan tambahkan data pertama Anda.</p>
                                        <button className="mt-6 px-6 py-2 bg-blue-600 text-white text-xs font-bold rounded-xl shadow-lg shadow-blue-100">Setup Modul</button>
                                    </div>
                                )}
                            </div>
                        </div>

                        {/* Mobile Nav */}
                        <nav className="lg:hidden bg-white border-t border-slate-100 px-6 py-3 flex justify-between items-center sticky bottom-0 z-50">
                            <button onClick={() => setActiveTab('Dashboard')} className={`flex flex-col items-center transition-colors ${activeTab === 'Dashboard' ? 'text-blue-600' : 'text-slate-400'}`}><Icon name="LayoutDashboard" size={20}/><span className="text-[10px] font-bold">Home</span></button>
                            <button onClick={() => setActiveTab('Produksi')} className={`flex flex-col items-center transition-colors ${activeTab === 'Produksi' ? 'text-blue-600' : 'text-slate-400'}`}><Icon name="Scissors" size={20}/><span className="text-[10px] font-bold">Prod</span></button>
                            <button className="bg-blue-600 text-white p-3 rounded-full -mt-10 shadow-xl border-4 border-white"><Icon name="Plus" size={24}/></button>
                            <button onClick={() => setActiveTab('Gudang')} className={`flex flex-col items-center transition-colors ${activeTab === 'Gudang' ? 'text-blue-600' : 'text-slate-400'}`}><Icon name="Warehouse" size={20}/><span className="text-[10px] font-bold">Gdg</span></button>
                            <button onClick={() => setActiveTab('Penjualan')} className={`flex flex-col items-center transition-colors ${activeTab === 'Penjualan' ? 'text-blue-600' : 'text-slate-400'}`}><Icon name="ShoppingCart" size={20}/><span className="text-[10px] font-bold">Shop</span></button>
                        </nav>
                    </main>

                    {/* Overlay for Sidebar */}
                    {isSidebarOpen && <div className="fixed inset-0 bg-slate-900/40 backdrop-blur-sm z-40 lg:hidden" onClick={() => setSidebarOpen(false)}></div>}
                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
