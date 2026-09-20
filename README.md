# kaiz sooq 
import React, { useState, useEffect } from 'react';
import { 
  LayoutDashboard, 
  PlusCircle, 
  ShoppingBag, 
  CheckCircle, 
  Users, 
  BarChart3, 
  Menu, 
  X, 
  Search, 
  Edit, 
  Trash2, 
  Phone, 
  Calendar, 
  DollarSign, 
  UserCheck, 
  Scissors, 
  Truck, 
  UserPlus, 
  AlertCircle 
} from 'lucide-react';

export default function App() {
  // Navigation State
  const [activeTab, setActiveTab] = useState('dashboard');
  const [isMobileMenuOpen, setIsMobileMenuOpen] = useState(false);

  // Persistent Data States
  const [orders, setOrders] = useState(() => {
    const saved = localStorage.getItem('biz_orders');
    return saved ? JSON.parse(saved) : [];
  });

  const [employees, setEmployees] = useState(() => {
    const saved = localStorage.getItem('biz_employees');
    return saved ? JSON.parse(saved) : [
      { id: '1', name: 'അനീഷ്', phone: '9876543210' },
      { id: '2', name: 'ഫാത്തിമ', phone: '9876543211' }
    ];
  });

  // Modal & Selection States
  const [selectedOrder, setSelectedOrder] = useState(null);
  const [selectedCustomerHistory, setSelectedCustomerHistory] = useState(null);
  const [editingOrder, setEditingOrder] = useState(null);
  const [searchTerm, setSearchTerm] = useState('');
  const [reportMonth, setReportMonth] = useState(new Date().toISOString().substring(0, 7));

  // New Employee Form State
  const [newEmp, setNewEmp] = useState({ name: '', phone: '' });

  // Order Form State
  const [formData, setFormData] = useState({
    category: 'Baby', // Baby or Lady
    customerName: '',
    whatsapp: '',
    ageOrDressDetails: '',
    date: new Date().toISOString().substring(0, 10),
    totalPrice: '',
    advanceAmount: '',
    isAdvanceCustom: false,
    materialCost: '',
    stitchingCharge: '',
    deliveryCharge: '0',
    assignedEmployeeId: '',
    status: 'Pending'
  });

  // LocalStorage Sync
  useEffect(() => {
    localStorage.setItem('biz_orders', JSON.stringify(orders));
  }, [orders]);

  useEffect(() => {
    localStorage.setItem('biz_employees', JSON.stringify(employees));
  }, [employees]);

  // Calculations for Form
  const totalPriceNum = parseFloat(formData.totalPrice) || 0;
  const advanceAmountNum = formData.isAdvanceCustom 
    ? (parseFloat(formData.advanceAmount) || 0) 
    : totalPriceNum / 2;
  const materialCostNum = parseFloat(formData.materialCost) || 0;
  const stitchingChargeNum = parseFloat(formData.stitchingCharge) || 0;
  const deliveryChargeNum = parseFloat(formData.deliveryCharge) || 0;

  // Advance Balance after Material Cost
  const advanceBalance = advanceAmountNum - materialCostNum;

  // Total Expenses & Net Profit Breakdown
  const totalExpenses = materialCostNum + stitchingChargeNum + deliveryChargeNum;
  const rawProfit = totalPriceNum - totalExpenses;
  
  // Owner Profit Logic (Up to 300 rupees)
  let ownerProfit = 0;
  let remainingProfit = 0;
  if (rawProfit > 0) {
    if (rawProfit >= 300) {
      ownerProfit = 300;
      remainingProfit = rawProfit - 300;
    } else {
      ownerProfit = rawProfit;
      remainingProfit = 0;
    }
  }

  // Handle Input Changes
  const handleInputChange = (e) => {
    const { name, value } = e.target;
    if (name === 'totalPrice') {
      const val = parseFloat(value) || 0;
      setFormData(prev => ({
        ...prev,
        totalPrice: value,
        advanceAmount: prev.isAdvanceCustom ? prev.advanceAmount : (val / 2).toString()
      }));
    } else if (name === 'advanceAmount') {
      setFormData(prev => ({
        ...prev,
        advanceAmount: value,
        isAdvanceCustom: true
      }));
    } else {
      setFormData(prev => ({ ...prev, [name]: value }));
    }
  };

  // Save Order
  const handleSaveOrder = (e) => {
    e.preventDefault();
    if (!formData.customerName || !formData.totalPrice) {
      alert('കസ്റ്റമറുടെ പേരും ടോട്ടൽ പ്രൈസും നൽകുക!');
      return;
    }

    const orderToSave = {
      ...formData,
      id: editingOrder ? editingOrder.id : Date.now().toString(),
      totalPrice: totalPriceNum,
      advanceAmount: advanceAmountNum,
      materialCost: materialCostNum,
      stitchingCharge: stitchingChargeNum,
      deliveryCharge: deliveryChargeNum,
      ownerProfit: ownerProfit,
      remainingProfit: remainingProfit,
      netProfit: rawProfit,
      createdAt: editingOrder ? editingOrder.createdAt : new Date().toISOString()
    };

    if (editingOrder) {
      setOrders(orders.map(o => o.id === editingOrder.id ? orderToSave : o));
      setEditingOrder(null);
    } else {
      setOrders([orderToSave, ...orders]);
    }

    // Reset Form
    setFormData({
      category: 'Baby',
      customerName: '',
      whatsapp: '',
      ageOrDressDetails: '',
      date: new Date().toISOString().substring(0, 10),
      totalPrice: '',
      advanceAmount: '',
      isAdvanceCustom: false,
      materialCost: '',
      stitchingCharge: '',
      deliveryCharge: '0',
      assignedEmployeeId: '',
      status: 'Pending'
    });

    setActiveTab('orders');
  };

  // Complete Order
  const handleCompleteOrder = (orderId) => {
    setOrders(orders.map(o => o.id === orderId ? { ...o, status: 'Completed' } : o));
  };

  // Delete Order
  const handleDeleteOrder = (orderId) => {
    if (window.confirm('ഈ ഓർഡർ മാറ്റാൻ ആഗ്രഹിക്കുന്നുണ്ടോ?')) {
      setOrders(orders.filter(o => o.id !== orderId));
      if (selectedOrder?.id === orderId) setSelectedOrder(null);
    }
  };

  // Add Employee
  const handleAddEmployee = (e) => {
    e.preventDefault();
    if (!newEmp.name) return;
    setEmployees([...employees, { ...newEmp, id: Date.now().toString() }]);
    setNewEmp({ name: '', phone: '' });
  };

  // Dashboard Aggregates
  const totalRevenue = orders.reduce((sum, o) => sum + o.totalPrice, 0);
  const totalProfit = orders.reduce((sum, o) => sum + o.netProfit, 0);
  const totalOwnerProfit = orders.reduce((sum, o) => sum + o.ownerProfit, 0);

  const babyOrders = orders.filter(o => o.category === 'Baby');
  const ladyOrders = orders.filter(o => o.category === 'Lady');

  const babyRevenue = babyOrders.reduce((sum, o) => sum + o.totalPrice, 0);
  const babyProfit = babyOrders.reduce((sum, o) => sum + o.netProfit, 0);
  const ladyRevenue = ladyOrders.reduce((sum, o) => sum + o.totalPrice, 0);
  const ladyProfit = ladyOrders.reduce((sum, o) => sum + o.netProfit, 0);

  const pendingOrders = orders.filter(o => o.status === 'Pending');
  const completedOrders = orders.filter(o => o.status === 'Completed');

  // Customer Grouping for Completed History
  const customerHistory = completedOrders.reduce((acc, curr) => {
    const key = curr.whatsapp || curr.customerName;
    if (!acc[key]) {
      acc[key] = {
        name: curr.customerName,
        whatsapp: curr.whatsapp,
        count: 0,
        totalSpent: 0,
        totalProfitGenerated: 0,
        orders: []
      };
    }
    acc[key].count += 1;
    acc[key].totalSpent += curr.totalPrice;
    acc[key].totalProfitGenerated += curr.netProfit;
    acc[key].orders.push(curr);
    return acc;
  }, {});

  return (
    <div className="min-h-screen bg-slate-900 text-slate-100 font-sans flex flex-col md:flex-row">
      
      {/* Mobile Top Header */}
      <div className="md:hidden flex items-center justify-between p-4 bg-slate-800 border-b border-slate-700 sticky top-0 z-50">
        <div className="flex items-center gap-2">
          <Scissors className="text-emerald-400" />
          <h1 className="font-bold text-lg text-emerald-400">BizBook Pro</h1>
        </div>
        <button onClick={() => setIsMobileMenuOpen(!isMobileMenuOpen)} className="p-2 text-slate-300">
          {isMobileMenuOpen ? <X /> : <Menu />}
        </button>
      </div>

      {/* Sidebar Navigation */}
      <aside className={`
        fixed inset-y-0 left-0 z-40 w-64 bg-slate-800 border-r border-slate-700 flex flex-col justify-between
        transform transition-transform duration-200 ease-in-out md:translate-x-0 md:static
        ${isMobileMenuOpen ? 'translate-x-0' : '-translate-x-full'}
      `}>
        <div>
          <div className="hidden md:flex items-center gap-3 p-6 border-b border-slate-700">
            <Scissors className="text-emerald-400 h-8 w-8" />
            <div>
              <h1 className="font-bold text-xl text-white">BizBook</h1>
              <p className="text-xs text-emerald-400 font-medium">Boutique & Tailoring</p>
            </div>
          </div>

          <nav className="p-4 space-y-2">
            {[
              { id: 'dashboard', label: 'ഡാഷ്‌ബോർഡ്', icon: LayoutDashboard },
              { id: 'add-order', label: 'പുതിയ ഓർഡർ', icon: PlusCircle },
              { id: 'orders', label: 'പെൻഡിങ് ഓർഡറുകൾ', icon: ShoppingBag, count: pendingOrders.length },
              { id: 'completed', label: 'കംപ്ലീറ്റ് ഓർഡറുകൾ', icon: CheckCircle },
              { id: 'employees', label: 'എംപ്ലോയീസ്', icon: Users },
              { id: 'reports', label: 'മാസ റിപ്പോർട്ട്', icon: BarChart3 }
            ].map(item => {
              const Icon = item.icon;
              return (
                <button
                  key={item.id}
                  onClick={() => {
                    setActiveTab(item.id);
                    setIsMobileMenuOpen(false);
                  }}
                  className={`w-full flex items-center justify-between p-3 rounded-xl font-medium transition ${
                    activeTab === item.id 
                      ? 'bg-emerald-600 text-white shadow-lg shadow-emerald-900/40' 
                      : 'text-slate-400 hover:bg-slate-700/50 hover:text-slate-200'
                  }`}
                >
                  <div className="flex items-center gap-3">
                    <Icon className="h-5 w-5" />
                    <span>{item.label}</span>
                  </div>
                  {item.count !== undefined && item.count > 0 && (
                    <span className="bg-rose-500 text-white text-xs px-2 py-0.5 rounded-full">
                      {item.count}
                    </span>
                  )}
                </button>
              );
            })}
          </nav>
        </div>

        <div className="p-4 border-t border-slate-700 text-xs text-slate-500 text-center">
          © 2026 BizBook Financial Tracker
        </div>
      </aside>

      {/* Main Content Area */}
      <main className="flex-1 p-4 md:p-8 overflow-y-auto max-w-7xl">
        
        {/* DASHBOARD VIEW */}
        {activeTab === 'dashboard' && (
          <div className="space-y-6">
            <div className="flex justify-between items-center">
              <h2 className="text-2xl font-bold text-white flex items-center gap-2">
                <LayoutDashboard className="text-emerald-400" /> പ്രധാന ഡാഷ്‌ബോർഡ്
              </h2>
              <button 
                onClick={() => setActiveTab('add-order')}
                className="bg-emerald-600 hover:bg-emerald-500 text-white px-4 py-2 rounded-xl flex items-center gap-2 text-sm font-semibold shadow-md"
              >
                <PlusCircle size={18} /> കസ്റ്റമറെ ആഡ് ചെയ്യുക
              </button>
            </div>

            {/* Top Cards Grid */}
            <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
              <div className="bg-slate-800 p-5 rounded-2xl border border-slate-700/60 shadow-sm">
                <p className="text-xs font-semibold text-slate-400 uppercase tracking-wider">Total Revenue</p>
                <p className="text-3xl font-extrabold text-white mt-2">₹{totalRevenue.toLocaleString()}</p>
                <p className="text-xs text-slate-400 mt-2">ആകെ ബിസിനസ്സ് തുക</p>
              </div>

              <div className="bg-slate-800 p-5 rounded-2xl border border-slate-700/60 shadow-sm">
                <p className="text-xs font-semibold text-emerald-400 uppercase tracking-wider">Total Net Profit</p>
                <p className="text-3xl font-extrabold text-emerald-400 mt-2">₹{totalProfit.toLocaleString()}</p>
                <p className="text-xs text-slate-400 mt-2">എല്ലാ ചെലവും കഴിച്ചുള്ള ലാഭം</p>
              </div>

              <div className="bg-slate-800 p-5 rounded-2xl border border-amber-500/30 shadow-sm">
                <p className="text-xs font-semibold text-amber-400 uppercase tracking-wider">Owner Profit (₹300/Order)</p>
                <p className="text-3xl font-extrabold text-amber-400 mt-2">₹{totalOwnerProfit.toLocaleString()}</p>
                <p className="text-xs text-slate-400 mt-2">ഓണർ ഫണ്ടിലേക്ക് മാറ്റിയത്</p>
              </div>

              <div className="bg-slate-800 p-5 rounded-2xl border border-blue-500/30 shadow-sm">
                <p className="text-xs font-semibold text-blue-400 uppercase tracking-wider">ഓർഡറുകൾ</p>
                <div className="flex justify-between items-end mt-2">
                  <div>
                    <span className="text-2xl font-bold text-white">{completedOrders.length}</span>
                    <span className="text-xs text-emerald-400 ml-1">കംപ്ലീറ്റ്</span>
                  </div>
                  <div>
                    <span className="text-2xl font-bold text-rose-400">{pendingOrders.length}</span>
                    <span className="text-xs text-rose-400 ml-1">പെൻഡിങ്</span>
                  </div>
                </div>
              </div>
            </div>

            {/* Baby vs Lady Performance */}
            <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
              <div className="bg-gradient-to-br from-slate-800 to-slate-800/80 p-5 rounded-2xl border border-pink-500/20">
                <div className="flex justify-between items-center mb-3">
                  <h3 className="font-bold text-pink-300 flex items-center gap-2">👶 ബേബി (Baby Category)</h3>
                  <span className="text-xs bg-pink-500/20 text-pink-300 px-2.5 py-1 rounded-full font-medium">{babyOrders.length} ഓർഡറുകൾ</span>
                </div>
                <div className="grid grid-cols-2 gap-4 mt-2">
                  <div>
                    <p className="text-xs text-slate-400">Revenue</p>
                    <p className="text-xl font-bold text-white">₹{babyRevenue.toLocaleString()}</p>
                  </div>
                  <div>
                    <p className="text-xs text-slate-400">Profit</p>
                    <p className="text-xl font-bold text-emerald-400">₹{babyProfit.toLocaleString()}</p>
                  </div>
                </div>
              </div>

              <div className="bg-gradient-to-br from-slate-800 to-slate-800/80 p-5 rounded-2xl border border-purple-500/20">
                <div className="flex justify-between items-center mb-3">
                  <h3 className="font-bold text-purple-300 flex items-center gap-2">💃 ലേഡി (Lady Category)</h3>
                  <span className="text-xs bg-purple-500/20 text-purple-300 px-2.5 py-1 rounded-full font-medium">{ladyOrders.length} ഓർഡറുകൾ</span>
                </div>
                <div className="grid grid-cols-2 gap-4 mt-2">
                  <div>
                    <p className="text-xs text-slate-400">Revenue</p>
                    <p className="text-xl font-bold text-white">₹{ladyRevenue.toLocaleString()}</p>
                  </div>
                  <div>
                    <p className="text-xs text-slate-400">Profit</p>
                    <p className="text-xl font-bold text-emerald-400">₹{ladyProfit.toLocaleString()}</p>
                  </div>
                </div>
              </div>
            </div>

            {/* Upcoming / Next Delivery Customer List */}
            <div className="bg-slate-800 rounded-2xl p-5 border border-slate-700">
              <h3 className="font-bold text-lg text-white mb-4 flex items-center gap-2">
                <Calendar className="text-emerald-400" /> അടുത്തതായി ഡെലിവറി ചെയ്യേണ്ട ഓർഡറുകൾ
              </h3>
              
              {pendingOrders.length === 0 ? (
                <p className="text-slate-400 text-sm py-4">പെൻഡിങ് ഡെലിവറികൾ ഒന്നുമില്ല!</p>
              ) : (
                <div className="overflow-x-auto">
                  <table className="w-full text-left text-sm">
                    <thead className="bg-slate-700/50 text-slate-300 uppercase text-xs">
                      <tr>
                        <th className="p-3 rounded-l-lg">കസ്റ്റമർ</th>
                        <th className="p-3">ഡെലിവറി ഡേറ്റ്</th>
                        <th className="p-3">കാറ്റഗറി</th>
                        <th className="p-3">ടോട്ടൽ പ്രൈസ്</th>
                        <th className="p-3 rounded-r-lg">ആക്ഷൻ</th>
                      </tr>
                    </thead>
                    <tbody className="divide-y divide-slate-700/40">
                      {pendingOrders.map(ord => (
                        <tr key={ord.id} className="hover:bg-slate-700/30 transition">
                          <td className="p-3 font-semibold text-white">{ord.customerName}</td>
                          <td className="p-3 text-emerald-400 font-medium">{ord.date}</td>
                          <td className="p-3">
                            <span className={`text-xs px-2 py-0.5 rounded-md ${ord.category === 'Baby' ? 'bg-pink-500/20 text-pink-300' : 'bg-purple-500/20 text-purple-300'}`}>
                              {ord.category}
                            </span>
                          </td>
                          <td className="p-3 font-bold">₹{ord.totalPrice}</td>
                          <td className="p-3">
                            <button 
                              onClick={() => setSelectedOrder(ord)}
                              className="bg-slate-700 hover:bg-slate-600 text-white text-xs px-3 py-1.5 rounded-lg border border-slate-600"
                            >
                              ഫുൾ ഡീറ്റെയിൽസ്
                            </button>
                          </td>
                        </tr>
                      ))}
                    </tbody>
                  </table>
                </div>
              )}
            </div>

          </div>
        )}

        {/* ADD / EDIT ORDER FORM */}
        {activeTab === 'add-order' && (
          <div className="max-w-3xl mx-auto bg-slate-800 p-6 md:p-8 rounded-2xl border border-slate-700 shadow-xl">
            <h2 className="text-2xl font-bold text-white mb-6 flex items-center gap-2">
              <PlusCircle className="text-emerald-400" /> {editingOrder ? 'ഓർഡർ എഡിറ്റ് ചെയ്യുക' : 'പുതിയ ഓർഡർ ചേര്ക്കുക'}
            </h2>

            <form onSubmit={handleSaveOrder} className="space-y-6">
              
              {/* 1. Category Selection */}
              <div>
                <label className="block text-sm font-semibold text-slate-300 mb-2">1. കാറ്റഗറി തിരഞ്ഞെടുക്കുക *</label>
                <div className="grid grid-cols-2 gap-4">
                  <button
                    type="button"
                    onClick={() => setFormData({ ...formData, category: 'Baby' })}
                    className={`p-4 rounded-xl font-bold text-center border-2 transition ${
                      formData.category === 'Baby' 
                        ? 'border-pink-500 bg-pink-500/10 text-pink-300' 
                        : 'border-slate-700 bg-slate-900 text-slate-400'
                    }`}
                  >
                    👶 ബേബി (Baby)
                  </button>
                  <button
                    type="button"
                    onClick={() => setFormData({ ...formData, category: 'Lady' })}
                    className={`p-4 rounded-xl font-bold text-center border-2 transition ${
                      formData.category === 'Lady' 
                        ? 'border-purple-500 bg-purple-500/10 text-purple-300' 
                        : 'border-slate-700 bg-slate-900 text-slate-400'
                    }`}
                  >
                    💃 ലേഡി (Lady)
                  </button>
                </div>
              </div>

              {/* Customer Info */}
              <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                  <label className="block text-xs font-medium text-slate-300 mb-1">കസ്റ്റമറുടെ പേര് *</label>
                  <input
                    type="text"
                    name="customerName"
                    value={formData.customerName}
                    onChange={handleInputChange}
                    placeholder="പേര് നൽകുക"
                    required
                    className="w-full bg-slate-900 border border-slate-700 rounded-xl p-3 text-white focus:outline-none focus:border-emerald-500"
                  />
                </div>

                <div>
                  <label className="block text-xs font-medium text-slate-300 mb-1">WhatsApp നമ്പർ</label>
                  <input
                    type="text"
                    name="whatsapp"
                    value={formData.whatsapp}
                    onChange={handleInputChange}
                    placeholder="9876543210"
                    className="w-full bg-slate-900 border border-slate-700 rounded-xl p-3 text-white focus:outline-none focus:border-emerald-500"
                  />
                </div>
              </div>

              {/* Age / Dress Details & Delivery Date */}
              <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                  <label className="block text-xs font-medium text-slate-300 mb-1">വയസ്സ് / ഡ്രസ്സ് വിവരങ്ങൾ</label>
                  <input
                    type="text"
                    name="ageOrDressDetails"
                    value={formData.ageOrDressDetails}
                    onChange={handleInputChange}
                    placeholder="ഉദാഹരണത്തിന്: 5 വയസ്സ്, ലെഹംഗ"
                    className="w-full bg-slate-900 border border-slate-700 rounded-xl p-3 text-white focus:outline-none focus:border-emerald-500"
                  />
                </div>

                <div>
                  <label className="block text-xs font-medium text-slate-300 mb-1">ഡെലിവറി ഡേറ്റ് *</label>
                  <input
                    type="date"
                    name="date"
                    value={formData.date}
                    onChange={handleInputChange}
                    required
                    className="w-full bg-slate-900 border border-slate-700 rounded-xl p-3 text-white focus:outline-none focus:border-emerald-500"
                  />
                </div>
              </div>

              {/* Financial Accounts Breakdown */}
              <div className="bg-slate-900 p-4 rounded-xl border border-slate-700 space-y-4">
                <h3 className="font-bold text-sm text-emerald-400 border-b border-slate-800 pb-2">കണക്കുകൾ & പേയ്‌മെന്റുകൾ</h3>

                <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                  {/* Total Price */}
                  <div>
                    <label className="block text-xs font-medium text-slate-300 mb-1">ടോട്ടൽ പ്രൈസ് (Total Price) *</label>
                    <input
                      type="number"
                      name="totalPrice"
                      value={formData.totalPrice}
                      onChange={handleInputChange}
                      placeholder="₹ 0.00"
                      required
                      className="w-full bg-slate-800 border border-slate-700 rounded-xl p-3 text-white font-bold text-lg focus:outline-none focus:border-emerald-500"
                    />
                  </div>

                  {/* Advance Amount (Auto 50%) */}
                  <div>
                    <label className="block text-xs font-medium text-slate-300 mb-1">
                      അഡ്വാൻസ് എമൗണ്ട് (50% Auto)
                    </label>
                    <input
                      type="number"
                      name="advanceAmount"
                      value={formData.advanceAmount !== '' ? formData.advanceAmount : (totalPriceNum / 2 || '')}
                      onChange={handleInputChange}
                      placeholder="₹ 0.00"
                      className="w-full bg-slate-800 border border-slate-700 rounded-xl p-3 text-emerald-400 font-bold text-lg focus:outline-none focus:border-emerald-500"
                    />
                    {/* Advance Balance Display */}
                    <div className="mt-1 text-xs text-amber-400 font-medium">
                      മെറ്റീരിയൽ വാങ്ങിയ ശേഷമുള്ള അഡ്വാൻസ് ബാലൻസ്: <span className="font-bold text-white">₹{advanceBalance}</span>
                    </div>
                  </div>
                </div>

                <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
                  {/* Material Cost */}
                  <div>
                    <label className="block text-xs font-medium text-slate-300 mb-1">മെറ്റീരിയൽ കോസ്റ്റ്</label>
                    <input
                      type="number"
                      name="materialCost"
                      value={formData.materialCost}
                      onChange={handleInputChange}
                      placeholder="₹ 0.00"
                      className="w-full bg-slate-800 border border-slate-700 rounded-xl p-2.5 text-white focus:outline-none focus:border-emerald-500"
                    />
                  </div>

                  {/* Stitching Charge */}
                  <div>
                    <label className="block text-xs font-medium text-slate-300 mb-1">സ്റ്റിച്ചിങ് ചാർജ്</label>
                    <input
                      type="number"
                      name="stitchingCharge"
                      value={formData.stitchingCharge}
                      onChange={handleInputChange}
                      placeholder="₹ 0.00"
                      className="w-full bg-slate-800 border border-slate-700 rounded-xl p-2.5 text-white focus:outline-none focus:border-emerald-500"
                    />
                  </div>

                  {/* Delivery Charge */}
                  <div>
                    <label className="block text-xs font-medium text-slate-300 mb-1">ഡെലിവറി ചാർജ്</label>
                    <input
                      type="number"
                      name="deliveryCharge"
                      value={formData.deliveryCharge}
                      onChange={handleInputChange}
                      placeholder="₹ 0.00"
                      className="w-full bg-slate-800 border border-slate-700 rounded-xl p-2.5 text-white focus:outline-none focus:border-emerald-500"
                    />
                  </div>
                </div>

                {/* Assigned Employee */}
                <div>
                  <label className="block text-xs font-medium text-slate-300 mb-1">സ്റ്റിച്ചിങ് ചെയ്യുന്ന എംപ്ലോയി</label>
                  <select
                    name="assignedEmployeeId"
                    value={formData.assignedEmployeeId}
                    onChange={handleInputChange}
                    className="w-full bg-slate-800 border border-slate-700 rounded-xl p-3 text-white focus:outline-none focus:border-emerald-500"
                  >
                    <option value="">എംപ്ലോയിയെ തിരഞ്ഞെടുക്കുക</option>
                    {employees.map(emp => (
                      <option key={emp.id} value={emp.id}>{emp.name}</option>
                    ))}
                  </select>
                </div>
              </div>

              {/* Profit Calculation Summary Preview */}
              <div className="bg-slate-900/60 p-4 rounded-xl border border-slate-700/80 space-y-2 text-xs">
                <div className="flex justify-between">
                  <span className="text-slate-400">മൊത്തം ചെലവുകൾ (Material + Stitching + Delivery):</span>
                  <span className="font-bold text-rose-400">₹{totalExpenses}</span>
                </div>
                <div className="flex justify-between border-t border-slate-800 pt-2">
                  <span className="text-slate-300">ഓണർ പ്രോഫിറ്റ് (Max ₹300):</span>
                  <span className="font-bold text-amber-400">₹{ownerProfit}</span>
                </div>
                <div className="flex justify-between">
                  <span className="text-slate-300">ബാക്കി നെറ്റ് പ്രോഫിറ്റ്:</span>
                  <span className="font-bold text-emerald-400">₹{remainingProfit}</span>
                </div>
                <div className="flex justify-between border-t border-slate-800 pt-2 text-sm">
                  <span className="font-bold text-white">മൊത്തം ലഭിക്കുന്ന പ്രോഫിറ്റ്:</span>
                  <span className="font-extrabold text-emerald-400">₹{rawProfit}</span>
                </div>
              </div>

              <div className="flex gap-4">
                <button
                  type="submit"
                  className="flex-1 bg-emerald-600 hover:bg-emerald-500 text-white font-bold py-3.5 rounded-xl shadow-lg transition"
                >
                  {editingOrder ? 'അപ്ഡേറ്റ് ചെയ്യുക' : 'സേവ് ചെയ്യുക'}
                </button>
                {editingOrder && (
                  <button
                    type="button"
                    onClick={() => {
                      setEditingOrder(null);
                      setActiveTab('orders');
                    }}
                    className="bg-slate-700 text-slate-300 py-3.5 px-6 rounded-xl font-bold"
                  >
                    ക്യാൻസൽ
                  </button>
                )}
              </div>

            </form>
          </div>
        )}

        {/* PENDING ORDERS LIST */}
        {activeTab === 'orders' && (
          <div className="space-y-6">
            <div className="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4">
              <h2 className="text-2xl font-bold text-white flex items-center gap-2">
                <ShoppingBag className="text-emerald-400" /> പെൻഡിങ് ഓർഡറുകൾ ({pendingOrders.length})
              </h2>

              {/* Search Box */}
              <div className="relative w-full sm:w-64">
                <Search className="absolute left-3 top-3 h-4 w-4 text-slate-400" />
                <input
                  type="text"
                  placeholder="കസ്റ്റമറെ തപ്പുക..."
                  value={searchTerm}
                  onChange={(e) => setSearchTerm(e.target.value)}
                  className="w-full bg-slate-800 border border-slate-700 pl-9 pr-4 py-2 rounded-xl text-sm text-white focus:outline-none"
                />
              </div>
            </div>

            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
              {pendingOrders
                .filter(o => o.customerName.toLowerCase().includes(searchTerm.toLowerCase()))
                .map(ord => (
                  <div key={ord.id} className="bg-slate-800 rounded-2xl border border-slate-700 p-5 flex flex-col justify-between space-y-4 hover:border-slate-600 transition">
                    <div>
                      <div className="flex justify-between items-start mb-2">
                        <span className={`text-xs px-2.5 py-1 rounded-md font-bold ${ord.category === 'Baby' ? 'bg-pink-500/20 text-pink-300' : 'bg-purple-500/20 text-purple-300'}`}>
                          {ord.category}
                        </span>
                        <span className="text-xs text-slate-400 flex items-center gap-1">
                          <Calendar size={12} /> {ord.date}
                        </span>
                      </div>

                      <h3 className="text-lg font-bold text-white mt-1">{ord.customerName}</h3>
                      {ord.whatsapp && (
                        <p className="text-xs text-slate-400 flex items-center gap-1 mt-0.5">
                          <Phone size={12} /> {ord.whatsapp}
                        </p>
                      )}

                      {/* Summary Cards Required on Card */}
                      <div className="mt-4 grid grid-cols-2 gap-2 bg-slate-900/60 p-3 rounded-xl text-center">
                        <div>
                          <p className="text-[10px] text-slate-400 uppercase">Total Price</p>
                          <p className="text-base font-extrabold text-white">₹{ord.totalPrice}</p>
                        </div>
                        <div>
                          <p className="text-[10px] text-emerald-400 uppercase">Profit</p>
                          <p className="text-base font-extrabold text-emerald-400">₹{ord.netProfit}</p>
                        </div>
                      </div>
                    </div>

                    <div className="space-y-2 pt-2 border-t border-slate-700/60">
                      <button
                        onClick={() => setSelectedOrder(ord)}
                        className="w-full bg-slate-700 hover:bg-slate-600 text-white text-xs font-semibold py-2 rounded-lg transition"
                      >
                        ഫുൾ ഓർഡർ ഡീറ്റെയിൽസ് കാണുക
                      </button>

                      <div className="flex gap-2">
                        <button
                          onClick={() => handleCompleteOrder(ord.id)}
                          className="flex-1 bg-emerald-600 hover:bg-emerald-500 text-white text-xs font-bold py-2 rounded-lg flex items-center justify-center gap-1 transition"
                        >
                          <CheckCircle size={14} /> ഓർഡർ കംപ്ലീറ്റ്
                        </button>
                        <button
                          onClick={() => {
                            setEditingOrder(ord);
                            setFormData(ord);
                            setActiveTab('add-order');
                          }}
                          className="p-2 bg-slate-700 hover:bg-slate-600 text-slate-300 rounded-lg"
                        >
                          <Edit size={14} />
                        </button>
                        <button
                          onClick={() => handleDeleteOrder(ord.id)}
                          className="p-2 bg-rose-500/20 hover:bg-rose-500/30 text-rose-300 rounded-lg"
                        >
                          <Trash2 size={14} />
                        </button>
                      </div>
                    </div>
                  </div>
                ))}
            </div>
          </div>
        )}

        {/* COMPLETED ORDERS & CUSTOMER HISTORY */}
        {activeTab === 'completed' && (
          <div className="space-y-6">
            <h2 className="text-2xl font-bold text-white flex items-center gap-2">
              <CheckCircle className="text-emerald-400" /> കംപ്ലീറ്റ് ആയ ഓർഡറുകളും കസ്റ്റമർ ഹിസ്റ്ററിയും
            </h2>

            <div className="bg-slate-800 rounded-2xl p-5 border border-slate-700">
              <div className="overflow-x-auto">
                <table className="w-full text-left text-sm">
                  <thead className="bg-slate-700/50 text-slate-300 uppercase text-xs">
                    <tr>
                      <th className="p-3">കസ്റ്റമർ</th>
                      <th className="p-3">ഫോൺ</th>
                      <th className="p-3">ആകെ ചെയ്ത പർച്ചേസ്</th>
                      <th className="p-3">ആകെ വാങ്ങിയ എമൗണ്ട്</th>
                      <th className="p-3">ലഭിച്ച ടോട്ടൽ പ്രോഫിറ്റ്</th>
                      <th className="p-3">ഡീറ്റെയിൽസ്</th>
                    </tr>
                  </thead>
                  <tbody className="divide-y divide-slate-700/40">
                    {Object.keys(customerHistory).length === 0 ? (
                      <tr>
                        <td colSpan={6} className="p-4 text-center text-slate-400">
                          കംപ്ലീറ്റ് ആയ ഓർഡറുകൾ ഒന്നും ഇതുവരെയില്ല.
                        </td>
                      </tr>
                    ) : (
                      Object.values(customerHistory).map((cust, idx) => (
                        <tr key={idx} className="hover:bg-slate-700/30 transition">
                          <td className="p-3 font-semibold text-white">{cust.name}</td>
                          <td className="p-3 text-slate-400">{cust.whatsapp || 'N/A'}</td>
                          <td className="p-3 font-medium text-blue-400">{cust.count} തവണ</td>
                          <td className="p-3 font-bold text-white">₹{cust.totalSpent.toLocaleString()}</td>
                          <td className="p-3 font-bold text-emerald-400">₹{cust.totalProfitGenerated.toLocaleString()}</td>
                          <td className="p-3">
                            <button
                              onClick={() => setSelectedCustomerHistory(cust)}
                              className="bg-slate-700 hover:bg-slate-600 text-white text-xs px-3 py-1.5 rounded-lg"
                            >
                              ഓർഡറുകൾ കാണുക
                            </button>
                          </td>
                        </tr>
                      ))
                    )}
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        )}

        {/* EMPLOYEES MANAGEMENT */}
        {activeTab === 'employees' && (
          <div className="space-y-6">
            <h2 className="text-2xl font-bold text-white flex items-center gap-2">
              <Users className="text-emerald-400" /> എംപ്ലോയീസ് മാനേജ്‌മെന്റ്
            </h2>

            {/* Add Employee Form */}
            <div className="bg-slate-800 p-5 rounded-2xl border border-slate-700">
              <h3 className="font-bold text-white mb-3 flex items-center gap-2 text-sm">
                <UserPlus size={16} className="text-emerald-400" /> പുതിയ എംപ്ലോയിയെ ചേർക്കുക
              </h3>
              <form onSubmit={handleAddEmployee} className="flex flex-col sm:flex-row gap-3">
                <input
                  type="text"
                  placeholder="എംപ്ലോയി പേര്"
                  value={newEmp.name}
                  onChange={(e) => setNewEmp({ ...newEmp, name: e.target.value })}
                  className="bg-slate-900 border border-slate-700 rounded-xl p-2.5 text-sm text-white focus:outline-none"
                  required
                />
                <input
                  type="text"
                  placeholder="ഫോൺ നമ്പർ"
                  value={newEmp.phone}
                  onChange={(e) => setNewEmp({ ...newEmp, phone: e.target.value })}
                  className="bg-slate-900 border border-slate-700 rounded-xl p-2.5 text-sm text-white focus:outline-none"
                />
                <button type="submit" className="bg-emerald-600 hover:bg-emerald-500 text-white text-sm font-bold px-5 py-2.5 rounded-xl">
                  ആഡ് ചെയ്യുക
                </button>
              </form>
            </div>

            {/* Employees Stats List */}
            <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
              {employees.map(emp => {
                const empOrders = orders.filter(o => o.assignedEmployeeId === emp.id);
                const empCompleted = empOrders.filter(o => o.status === 'Completed');
                const empPending = empOrders.filter(o => o.status === 'Pending');
                const empTotalStitchingAmount = empOrders.reduce((sum, o) => sum + (o.stitchingCharge || 0), 0);

                return (
                  <div key={emp.id} className="bg-slate-800 p-5 rounded-2xl border border-slate-700 space-y-4">
                    <div className="flex justify-between items-start">
                      <div>
                        <h3 className="font-bold text-lg text-white">{emp.name}</h3>
                        <p className="text-xs text-slate-400">{emp.phone}</p>
                      </div>
                      <span className="bg-slate-700 text-xs px-2.5 py-1 rounded-full text-slate-300 font-semibold">
                        മൊത്തം: {empOrders.length} ഡ്രസ്സ്
                      </span>
                    </div>

                    <div className="grid grid-cols-3 gap-2 bg-slate-900/60 p-3 rounded-xl text-center">
                      <div>
                        <p className="text-[10px] text-emerald-400 uppercase">കംപ്ലീറ്റ്</p>
                        <p className="text-lg font-bold text-emerald-400">{empCompleted.length}</p>
                      </div>
                      <div>
                        <p className="text-[10px] text-rose-400 uppercase">പെൻഡിങ്</p>
                        <p className="text-lg font-bold text-rose-400">{empPending.length}</p>
                      </div>
                      <div>
                        <p className="text-[10px] text-amber-400 uppercase">തുന്നൽ ചാർജ്</p>
                        <p className="text-lg font-bold text-amber-400">₹{empTotalStitchingAmount}</p>
                      </div>
                    </div>
                  </div>
                );
              })}
            </div>
          </div>
        )}

        {/* MONTHLY FINANCIAL REPORTS */}
        {activeTab === 'reports' && (
          <div className="space-y-6">
            <div className="flex justify-between items-center">
              <h2 className="text-2xl font-bold text-white flex items-center gap-2">
                <BarChart3 className="text-emerald-400" /> ഓരോ മാസത്തെയും റിപോർട്ടുകൾ
              </h2>
              <input
                type="month"
                value={reportMonth}
                onChange={(e) => setReportMonth(e.target.value)}
                className="bg-slate-800 border border-slate-700 rounded-xl p-2 text-sm text-white focus:outline-none"
              />
            </div>

            {(() => {
              const monthOrders = orders.filter(o => o.date.startsWith(reportMonth));
              const monthRev = monthOrders.reduce((sum, o) => sum + o.totalPrice, 0);
              const monthProf = monthOrders.reduce((sum, o) => sum + o.netProfit, 0);
              const monthOwner = monthOrders.reduce((sum, o) => sum + o.ownerProfit, 0);

              return (
                <div className="space-y-4">
                  <div className="grid grid-cols-1 sm:grid-cols-3 gap-4">
                    <div className="bg-slate-800 p-5 rounded-2xl border border-slate-700">
                      <p className="text-xs text-slate-400 uppercase">മാസ റവന്യൂ</p>
                      <p className="text-2xl font-bold text-white mt-1">₹{monthRev.toLocaleString()}</p>
                    </div>
                    <div className="bg-slate-800 p-5 rounded-2xl border border-slate-700">
                      <p className="text-xs text-emerald-400 uppercase">മാസ നെറ്റ് പ്രോഫിറ്റ്</p>
                      <p className="text-2xl font-bold text-emerald-400 mt-1">₹{monthProf.toLocaleString()}</p>
                    </div>
                    <div className="bg-slate-800 p-5 rounded-2xl border border-slate-700">
                      <p className="text-xs text-amber-400 uppercase">ഓണർ പ്രോഫിറ്റ്</p>
                      <p className="text-2xl font-bold text-amber-400 mt-1">₹{monthOwner.toLocaleString()}</p>
                    </div>
                  </div>

                  <div className="bg-slate-800 rounded-2xl p-5 border border-slate-700">
                    <h3 className="font-bold text-white mb-3">ഈ മാസത്തെ ഓർഡറുകൾ ({monthOrders.length})</h3>
                    <div className="space-y-2">
                      {monthOrders.map(o => (
                        <div key={o.id} className="flex justify-between items-center p-3 bg-slate-900/50 rounded-xl">
                          <div>
                            <p className="font-semibold text-white">{o.customerName}</p>
                            <p className="text-xs text-slate-400">{o.date} | {o.category}</p>
                          </div>
                          <div className="text-right">
                            <p className="font-bold text-white">₹{o.totalPrice}</p>
                            <p className="text-xs text-emerald-400 font-semibold">പ്രോഫിറ്റ്: ₹{o.netProfit}</p>
                          </div>
                        </div>
                      ))}
                    </div>
                  </div>
                </div>
              );
            })()}
          </div>
        )}

      </main>

      {/* FULL ORDER DETAILS MODAL */}
      {selectedOrder && (
        <div className="fixed inset-0 bg-black/70 backdrop-blur-sm flex items-center justify-center p-4 z-50">
          <div className="bg-slate-800 border border-slate-700 w-full max-w-lg rounded-2xl p-6 space-y-4 max-h-[90vh] overflow-y-auto">
            <div className="flex justify-between items-center border-b border-slate-700 pb-3">
              <h3 className="font-bold text-lg text-white">ഫുൾ ഓർഡർ വിവരങ്ങൾ</h3>
              <button onClick={() => setSelectedOrder(null)} className="text-slate-400 hover:text-white">
                <X size={20} />
              </button>
            </div>

            <div className="space-y-3 text-sm">
              <div className="flex justify-between">
                <span className="text-slate-400">കസ്റ്റമർ പേര്:</span>
                <span className="font-bold text-white">{selectedOrder.customerName}</span>
              </div>
              <div className="flex justify-between">
                <span className="text-slate-400">കാറ്റഗറി:</span>
                <span className="font-bold text-emerald-400">{selectedOrder.category}</span>
              </div>
              <div className="flex justify-between">
                <span className="text-slate-400">വാട്സാപ്പ് നമ്പർ:</span>
                <span className="text-white">{selectedOrder.whatsapp || 'N/A'}</span>
              </div>
              <div className="flex justify-between">
                <span className="text-slate-400">വയസ്സ് / ഡ്രസ്സ് ഡീറ്റെയിൽസ്:</span>
                <span className="text-white">{selectedOrder.ageOrDressDetails || 'N/A'}</span>
              </div>
              <div className="flex justify-between">
                <span className="text-slate-400">ഡെലിവറി ഡേറ്റ്:</span>
                <span className="text-white">{selectedOrder.date}</span>
              </div>

              <div className="bg-slate-900 p-4 rounded-xl space-y-2 border border-slate-700 my-2">
                <div className="flex justify-between">
                  <span>ടോട്ടൽ പ്രൈസ്:</span>
                  <span className="font-bold text-white">₹{selectedOrder.totalPrice}</span>
                </div>
                <div className="flex justify-between text-emerald-400">
                  <span>വാങ്ങിയ അഡ്വാൻസ്:</span>
                  <span>₹{selectedOrder.advanceAmount}</span>
                </div>
                <div className="flex justify-between text-rose-400">
                  <span>മെറ്റീരിയൽ കോസ്റ്റ്:</span>
                  <span>- ₹{selectedOrder.materialCost}</span>
                </div>
                <div className="flex justify-between text-rose-400">
                  <span>സ്റ്റിച്ചിങ് ചാർജ്:</span>
                  <span>- ₹{selectedOrder.stitchingCharge}</span>
                </div>
                <div className="flex justify-between text-rose-400">
                  <span>ഡെലിവറി ചാർജ്:</span>
                  <span>- ₹{selectedOrder.deliveryCharge}</span>
                </div>
                <div className="border-t border-slate-800 pt-2 flex justify-between text-amber-400">
                  <span>ഓണർ പ്രോഫിറ്റ്:</span>
                  <span className="font-bold">₹{selectedOrder.ownerProfit}</span>
                </div>
                <div className="flex justify-between text-emerald-400 font-bold text-base border-t border-slate-800 pt-1">
                  <span>ആകെ നെറ്റ് പ്രോഫിറ്റ്:</span>
                  <span>₹{selectedOrder.netProfit}</span>
                </div>
              </div>
            </div>

            <button
              onClick={() => setSelectedOrder(null)}
              className="w-full bg-slate-700 text-white font-bold py-2.5 rounded-xl"
            >
              ക്ലോസ് ചെയ്യുക
            </button>
          </div>
        </div>
      )}

      {/* CUSTOMER HISTORY MODAL */}
      {selectedCustomerHistory && (
        <div className="fixed inset-0 bg-black/70 backdrop-blur-sm flex items-center justify-center p-4 z-50">
          <div className="bg-slate-800 border border-slate-700 w-full max-w-xl rounded-2xl p-6 space-y-4 max-h-[90vh] overflow-y-auto">
            <div className="flex justify-between items-center border-b border-slate-700 pb-3">
              <div>
                <h3 className="font-bold text-lg text-white">{selectedCustomerHistory.name}</h3>
                <p className="text-xs text-slate-400">പർച്ചേസ് ഹിസ്റ്ററി ({selectedCustomerHistory.count} ഓർഡറുകൾ)</p>
              </div>
              <button onClick={() => setSelectedCustomerHistory(null)} className="text-slate-400 hover:text-white">
                <X size={20} />
              </button>
            </div>

            <div className="space-y-3">
              {selectedCustomerHistory.orders.map((o, i) => (
                <div key={i} className="bg-slate-900 p-4 rounded-xl space-y-1">
                  <div className="flex justify-between text-xs text-slate-400">
                    <span>{o.date}</span>
                    <span className="text-emerald-400">{o.category}</span>
                  </div>
                  <div className="flex justify-between font-bold text-white text-sm">
                    <span>{o.ageOrDressDetails || 'Dress Order'}</span>
                    <span>₹{o.totalPrice}</span>
                  </div>
                  <div className="text-xs text-emerald-400">
                    ഈ ഓർഡറിൽ ലഭിച്ച പ്രോഫിറ്റ്: ₹{o.netProfit}
                  </div>
                </div>
              ))}
            </div>

            <button
              onClick={() => setSelectedCustomerHistory(null)}
              className="w-full bg-slate-700 text-white font-bold py-2.5 rounded-xl"
            >
              ക്ലോസ് ചെയ്യുക
            </button>
          </div>
        </div>
      )}

    </div>
  );
}
