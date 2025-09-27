const { useState, useEffect } = React;
const { PlusCircle, TrendingUp, TrendingDown, DollarSign, CreditCard, Calendar, Trash2, Edit3, Download, AlertTriangle, Target, Bell, Settings, Filter, BarChart3 } = lucide;
const { LineChart, Line, AreaChart, Area, BarChart, Bar, PieChart, Pie, Cell, XAxis, YAxis, CartesianGrid, Tooltip, Legend, ResponsiveContainer } = Recharts;

const FinancialDashboard = () => {
  const [transactions, setTransactions] = useState([
    { id: 1, type: 'ingreso', category: 'Salario', amount: 3500000, description: 'Salario mensual', date: '2025-09-01' },
    { id: 2, type: 'gasto', category: 'Vivienda', amount: 800000, description: 'Arriendo', date: '2025-09-05' },
    { id: 3, type: 'gasto', category: 'Alimentación', amount: 450000, description: 'Mercado', date: '2025-09-10' },
    { id: 4, type: 'deuda', category: 'Tarjeta', amount: 1200000, description: 'Tarjeta de crédito', date: '2025-09-15' },
  ]);

  const [budgets, setBudgets] = useState([
    { id: 1, category: 'Alimentación', limit: 500000, period: 'mensual' },
    { id: 2, category: 'Transporte', limit: 300000, period: 'mensual' },
    { id: 3, category: 'Entretenimiento', limit: 200000, period: 'mensual' },
  ]);

  const [newTransaction, setNewTransaction] = useState({
    type: 'gasto',
    category: '',
    amount: '',
    description: '',
    date: new Date().toISOString().split('T')[0]
  });

  const [newBudget, setNewBudget] = useState({
    category: '',
    limit: '',
    period: 'mensual'
  });

  const [editingId, setEditingId] = useState(null);
  const [filter, setFilter] = useState('all');
  const [activeTab, setActiveTab] = useState('dashboard');
  const [alerts, setAlerts] = useState([]);
  const [projectionMonths, setProjectionMonths] = useState(6);

  const categories = {
    ingreso: ['Salario', 'Freelance', 'Inversiones', 'Bonos', 'Ventas', 'Otro ingreso'],
    gasto: ['Vivienda', 'Alimentación', 'Transporte', 'Entretenimiento', 'Salud', 'Educación', 'Ropa', 'Servicios', 'Tecnología', 'Otro gasto'],
    deuda: ['Tarjeta', 'Préstamo', 'Hipoteca', 'Crédito educativo', 'Otra deuda']
  };

  const colors = {
    ingreso: '#10B981',
    gasto: '#EF4444',
    deuda: '#F59E0B'
  };

  const pieColors = ['#10B981', '#EF4444', '#F59E0B', '#8B5CF6', '#06B6D4', '#F97316', '#EC4899', '#84CC16'];

  // Cálculos financieros
  const totalIngresos = transactions
    .filter(t => t.type === 'ingreso')
    .reduce((sum, t) => sum + t.amount, 0);

  const totalGastos = transactions
    .filter(t => t.type === 'gasto')
    .reduce((sum, t) => sum + t.amount, 0);

  const totalDeudas = transactions
    .filter(t => t.type === 'deuda')
    .reduce((sum, t) => sum + t.amount, 0);

  const balance = totalIngresos - totalGastos;
  const disponible = balance - totalDeudas;

  // Cálculo de presupuestos y alertas
  const currentMonth = new Date().toISOString().substring(0, 7);
  const monthlyTransactions = transactions.filter(t => t.date.startsWith(currentMonth));
  
  const budgetAlerts = budgets.map(budget => {
    const spent = monthlyTransactions
      .filter(t => t.type === 'gasto' && t.category === budget.category)
      .reduce((sum, t) => sum + t.amount, 0);
    
    const percentage = (spent / budget.limit) * 100;
    const remaining = budget.limit - spent;
    
    return {
      id: budget.id,
      category: budget.category,
      spent,
      limit: budget.limit,
      percentage,
      remaining,
      status: percentage >= 100 ? 'exceeded' : percentage >= 80 ? 'warning' : 'good'
    };
  });

  // Generar proyecciones
  const generateProjections = () => {
    const avgIncome = totalIngresos;
    const avgExpenses = totalGastos;
    const projections = [];
    
    for (let i = 1; i <= projectionMonths; i++) {
      const projectedDate = new Date();
      projectedDate.setMonth(projectedDate.getMonth() + i);
      const monthStr = projectedDate.toISOString().substring(0, 7);
      
      projections.push({
        month: monthStr,
        ingresos: avgIncome,
        gastos: avgExpenses,
        balance: avgIncome - avgExpenses,
        acumulado: (avgIncome - avgExpenses) * i + disponible
      });
    }
    
    return projections;
  };

  const projectionData = generateProjections();

  // Datos para gráficos
  const monthlyData = transactions.reduce((acc, t) => {
    const month = t.date.substring(0, 7);
    if (!acc[month]) {
      acc[month] = { month, ingresos: 0, gastos: 0, deudas: 0 };
    }
    acc[month][t.type === 'ingreso' ? 'ingresos' : t.type === 'gasto' ? 'gastos' : 'deudas'] += t.amount;
    return acc;
  }, {});

  const chartData = Object.values(monthlyData);

  const categoryData = transactions.reduce((acc, t) => {
    if (!acc[t.category]) {
      acc[t.category] = { name: t.category, value: 0, type: t.type };
    }
    acc[t.category].value += t.amount;
    return acc;
  }, {});

  const pieData = Object.values(categoryData);

  const formatCurrency = (amount) => {
    return new Intl.NumberFormat('es-CO', {
      style: 'currency',
      currency: 'COP',
      minimumFractionDigits: 0
    }).format(amount);
  };

  const exportToJSON = () => {
    const data = {
      transactions,
      budgets,
      summary: {
        totalIngresos,
        totalGastos,
        totalDeudas,
        balance,
        disponible
      },
      exportDate: new Date().toISOString()
    };
    
    const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `finanzas_${new Date().toISOString().split('T')[0]}.json`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  };

  const exportToCSV = () => {
    const headers = ['Fecha', 'Tipo', 'Categoría', 'Descripción', 'Monto'];
    const csvContent = [
      headers.join(','),
      ...transactions.map(t => [
        t.date,
        t.type,
        t.category,
        `"${t.description}"`,
        t.amount
      ].join(','))
    ].join('\n');
    
    const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `transacciones_${new Date().toISOString().split('T')[0]}.csv`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  };

  const handleSubmit = () => {
    if (!newTransaction.category || !newTransaction.amount) return;

    const transaction = {
      id: editingId || Date.now(),
      ...newTransaction,
      amount: parseFloat(newTransaction.amount)
    };

    if (editingId) {
      setTransactions(prev => prev.map(t => t.id === editingId ? transaction : t));
      setEditingId(null);
    } else {
      setTransactions(prev => [...prev, transaction]);
    }

    setNewTransaction({
      type: 'gasto',
      category: '',
      amount: '',
      description: '',
      date: new Date().toISOString().split('T')[0]
    });
  };

  const handleBudgetSubmit = () => {
    if (!newBudget.category || !newBudget.limit) return;

    const budget = {
      id: Date.now(),
      ...newBudget,
      limit: parseFloat(newBudget.limit)
    };

    setBudgets(prev => [...prev, budget]);
    setNewBudget({ category: '', limit: '', period: 'mensual' });
  };

  const handleEdit = (transaction) => {
    setNewTransaction({
      type: transaction.type,
      category: transaction.category,
      amount: transaction.amount.toString(),
      description: transaction.description,
      date: transaction.date
    });
    setEditingId(transaction.id);
  };

  const handleDelete = (id) => {
    setTransactions(prev => prev.filter(t => t.id !== id));
  };

  const deleteBudget = (id) => {
    setBudgets(prev => prev.filter(b => b.id !== id));
  };

  const filteredTransactions = transactions.filter(t => 
    filter === 'all' || t.type === filter
  );

  // Actualizar alertas
  useEffect(() => {
    const newAlerts = budgetAlerts
      .filter(alert => alert.status !== 'good')
      .map(alert => ({
        id: alert.id,
        type: alert.status,
        message: alert.status === 'exceeded' 
          ? `¡Has excedido el presupuesto de ${alert.category} por ${formatCurrency(Math.abs(alert.remaining))}!`
          : `Alerta: Has gastado el ${alert.percentage.toFixed(0)}% del presupuesto de ${alert.category}`
      }));
    
    setAlerts(newAlerts);
  }, [transactions, budgets]);

  const TabButton = ({ id, label, icon: Icon }) => (
    <button
      onClick={() => setActiveTab(id)}
      className={`flex items-center space-x-2 px-4 py-2 rounded-lg font-medium transition-colors ${
        activeTab === id 
          ? 'bg-blue-600 text-white' 
          : 'text-gray-600 hover:bg-gray-100'
      }`}
    >
      <Icon className="h-4 w-4" />
      <span>{label}</span>
    </button>
  );

  return (
    <div className="min-h-screen bg-gradient-to-br from-slate-50 to-blue-50 p-6">
      <div className="max-w-7xl mx-auto space-y-6">
        {/* Header */}
        <div className="text-center">
          <h1 className="text-4xl font-bold text-gray-800 mb-2">Dashboard Financiero Avanzado</h1>
          <p className="text-gray-600">Gestiona tus finanzas con análisis inteligente y proyecciones</p>
        </div>

        {/* Alertas */}
        {alerts.length > 0 && (
          <div className="bg-white rounded-xl shadow-lg p-6 border-l-4 border-yellow-500">
            <div className="flex items-center space-x-2 mb-4">
              <Bell className="h-5 w-5 text-yellow-600" />
              <h3 className="font-semibold text-gray-800">Alertas de Presupuesto</h3>
            </div>
            <div className="space-y-2">
              {alerts.map(alert => (
                <div key={alert.id} className={`p-3 rounded-lg ${
                  alert.type === 'exceeded' ? 'bg-red-50 text-red-800' : 'bg-yellow-50 text-yellow-800'
                }`}>
                  <div className="flex items-center space-x-2">
                    <AlertTriangle className="h-4 w-4" />
                    <span className="text-sm">{alert.message}</span>
                  </div>
                </div>
              ))}
            </div>
          </div>
        )}

        {/* Navegación por tabs */}
        <div className="bg-white rounded-xl shadow-lg p-6">
          <div className="flex space-x-2 overflow-x-auto">
            <TabButton id="dashboard" label="Dashboard" icon={TrendingUp} />
            <TabButton id="budgets" label="Presupuestos" icon={Target} />
            <TabButton id="projections" label="Proyecciones" icon={BarChart3} />
            <TabButton id="export" label="Exportar" icon={Download} />
          </div>
        </div>

        {/* Tab: Dashboard */}
        {activeTab === 'dashboard' && (
          <>
            {/* Métricas principales */}
            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
              <div className="bg-white rounded-xl shadow-lg p-6 border-l-4 border-green-500">
                <div className="flex items-center justify-between">
                  <div>
                    <p className="text-sm font-medium text-gray-600">Total Ingresos</p>
                    <p className="text-2xl font-bold text-green-600">{formatCurrency(totalIngresos)}</p>
                  </div>
                  <TrendingUp className="h-8 w-8 text-green-500" />
                </div>
              </div>

              <div className="bg-white rounded-xl shadow-lg p-6 border-l-4 border-red-500">
                <div className="flex items-center justify-between">
                  <div>
                    <p className="text-sm font-medium text-gray-600">Total Gastos</p>
                    <p className="text-2xl font-bold text-red-600">{formatCurrency(totalGastos)}</p>
                  </div>
                  <TrendingDown className="h-8 w-8 text-red-500" />
                </div>
              </div>

              <div className="bg-white rounded-xl shadow-lg p-6 border-l-4 border-yellow-500">
                <div className="flex items-center justify-between">
                  <div>
                    <p className="text-sm font-medium text-gray-600">Total Deudas</p>
                    <p className="text-2xl font-bold text-yellow-600">{formatCurrency(totalDeudas)}</p>
                  </div>
                  <CreditCard className="h-8 w-8 text-yellow-500" />
                </div>
              </div>

              <div className={`bg-white rounded-xl shadow-lg p-6 border-l-4 ${disponible >= 0 ? 'border-blue-500' : 'border-red-500'}`}>
                <div className="flex items-center justify-between">
                  <div>
                    <p className="text-sm font-medium text-gray-600">Disponible</p>
                    <p className={`text-2xl font-bold ${disponible >= 0 ? 'text-blue-600' : 'text-red-600'}`}>
                      {formatCurrency(disponible)}
                    </p>
                  </div>
                  <DollarSign className={`h-8 w-8 ${disponible >= 0 ? 'text-blue-500' : 'text-red-500'}`} />
                </div>
              </div>
            </div>

            {/* Gráficos */}
            <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
              <div className="bg-white rounded-xl shadow-lg p-6">
                <h3 className="text-lg font-semibold text-gray-800 mb-4">Flujo de Caja Mensual</h3>
                <ResponsiveContainer width="100%" height={300}>
                  <AreaChart data={chartData}>
                    <CartesianGrid strokeDasharray="3 3" />
                    <XAxis dataKey="month" />
                    <YAxis tickFormatter={(value) => `${(value / 1000000).toFixed(1)}M`} />
                    <Tooltip formatter={(value) => formatCurrency(value)} />
                    <Legend />
                    <Area type="monotone" dataKey="ingresos" stackId="1" stroke="#10B981" fill="#10B981" fillOpacity={0.6} />
                    <Area type="monotone" dataKey="gastos" stackId="2" stroke="#EF4444" fill="#EF4444" fillOpacity={0.6} />
                    <Area type="monotone" dataKey="deudas" stackId="3" stroke="#F59E0B" fill="#F59E0B" fillOpacity={0.6} />
                  </AreaChart>
                </ResponsiveContainer>
              </div>

              <div className="bg-white rounded-xl shadow-lg p-6">
                <h3 className="text-lg font-semibold text-gray-800 mb-4">Distribución por Categoría</h3>
                <ResponsiveContainer width="100%" height={300}>
                  <PieChart>
                    <Pie
                      data={pieData}
                      cx="50%"
                      cy="50%"
                      labelLine={false}
                      label={({name, value}) => `${name}: ${formatCurrency(value)}`}
                      outerRadius={80}
                      fill="#8884d8"
                      dataKey="value"
                    >
                      {pieData.map((entry, index) => (
                        <Cell key={`cell-${index}`} fill={pieColors[index % pieColors.length]} />
                      ))}
                    </Pie>
                    <Tooltip formatter={(value) => formatCurrency(value)} />
                  </PieChart>
                </ResponsiveContainer>
              </div>
            </div>

            {/* Formulario y Transacciones */}
            <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
              <div className="bg-white rounded-xl shadow-lg p-6">
                <h3 className="text-lg font-semibold text-gray-800 mb-4">
                  {editingId ? 'Editar Transacción' : 'Nueva Transacción'}
                </h3>
                <div className="space-y-4">
                  <select
                    value={newTransaction.type}
                    onChange={(e) => setNewTransaction({...newTransaction, type: e.target.value, category: ''})}
                    className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
                  >
                    <option value="gasto">Gasto</option>
                    <option value="ingreso">Ingreso</option>
                    <option value="deuda">Deuda</option>
                  </select>

                  <select
                    value={newTransaction.category}
                    onChange={(e) => setNewTransaction({...newTransaction, category: e.target.value})}
                    className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
                    required
                  >
                    <option value="">Selecciona categoría</option>
                    {categories[newTransaction.type].map(cat => (
                      <option key={cat} value={cat}>{cat}</option>
                    ))}
                  </select>

                  <input
                    type="number"
                    placeholder="Monto"
                    value={newTransaction.amount}
                    onChange={(e) => setNewTransaction({...newTransaction, amount: e.target.value})}
                    className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
                    required
                  />

                  <input
                    type="text"
                    placeholder="Descripción"
                    value={newTransaction.description}
                    onChange={(e) => setNewTransaction({...newTransaction, description: e.target.value})}
                    className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
                  />

                  <input
                    type="date"
                    value={newTransaction.date}
                    onChange={(e) => setNewTransaction({...newTransaction, date: e.target.value})}
                    className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
                    required
                  />

                  <button
                    onClick={handleSubmit}
                    className="w-full bg-blue-600 hover:bg-blue-700 text-white font-semibold py-3 px-4 rounded-lg transition-colors flex items-center justify-center"
                  >
                    <PlusCircle className="h-5 w-5 mr-2" />
                    {editingId ? 'Actualizar' : 'Agregar'} Transacción
                  </button>

                  {editingId && (
                    <button
                      onClick={() => {
                        setEditingId(null);
                        setNewTransaction({
                          type: 'gasto',
                          category: '',
                          amount: '',
                          description: '',
                          date: new Date().toISOString().split('T')[0]
                        });
                      }}
                      className="w-full bg-gray-500 hover:bg-gray-600 text-white font-semibold py-3 px-4 rounded-lg transition-colors"
                    >
                      Cancelar
                    </button>
                  )}
                </div>
              </div>

              <div className="lg:col-span-2 bg-white rounded-xl shadow-lg p-6">
                <div className="flex justify-between items-center mb-4">
                  <h3 className="text-lg font-semibold text-gray-800">Transacciones Recientes</h3>
                  <select
                    value={filter}
                    onChange={(e) => setFilter(e.target.value)}
                    className="p-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
                  >
                    <option value="all">Todas</option>
                    <option value="ingreso">Ingresos</option>
                    <option value="gasto">Gastos</option>
                    <option value="deuda">Deudas</option>
                  </select>
                </div>

                <div className="max-h-96 overflow-y-auto space-y-3">
                  {filteredTransactions.map(transaction => (
                    <div
                      key={transaction.id}
                      className="flex items-center justify-between p-4 border border-gray-200 rounded-lg hover:bg-gray-50 transition-colors"
                    >
                      <div className="flex-1">
                        <div className="flex items-center space-x-3">
                          <div className={`w-3 h-3 rounded-full ${
                            transaction.type === 'ingreso' ? 'bg-green-500' :
                            transaction.type === 'gasto' ? 'bg-red-500' : 'bg-yellow-500'
                          }`}></div>
                          <div>
                            <p className="font-medium text-gray-800">{transaction.category}</p>
                            <p className="text-sm text-gray-600">{transaction.description}</p>
                            <p className="text-xs text-gray-500">{transaction.date}</p>
                          </div>
                        </div>
                      </div>
                      <div className="flex items-center space-x-3">
                        <span className={`font-semibold ${
                          transaction.type === 'ingreso' ? 'text-green-600' :
                          transaction.type === 'gasto' ? 'text-red-600' : 'text-yellow-600'
                        }`}>
                          {formatCurrency(transaction.amount)}
                        </span>
                        <button
                          onClick={() => handleEdit(transaction)}
                          className="p-1 text-blue-600 hover:bg-blue-100 rounded"
                        >
                          <Edit3 className="h-4 w-4" />
                        </button>
                        <button
                          onClick={() => handleDelete(transaction.id)}
                          className="p-1 text-red-600 hover:bg-red-100 rounded"
                        >
                          <Trash2 className="h-4 w-4" />
                        </button>
                      </div>
                    </div>
                  ))}
                </div>
              </div>
            </div>
          </>
        )}

        {/* Tab: Presupuestos */}
        {activeTab === 'budgets' && (
          <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
            <div className="bg-white rounded-xl shadow-lg p-6">
              <h3 className="text-lg font-semibold text-gray-800 mb-4">Crear Nuevo Presupuesto</h3>
              <div className="space-y-4">
                <select
                  value={newBudget.category}
                  onChange={(e) => setNewBudget({...newBudget, category: e.target.value})}
                  className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
                >
                  <option value="">Selecciona categoría</option>
                  {categories.gasto.map(cat => (
                    <option key={cat} value={cat}>{cat}</option>
                  ))}
                </select>

                <input
                  type="number"
                  placeholder="Límite mensual"
                  value={newBudget.limit}
                  onChange={(e) => setNewBudget({...newBudget, limit: e.target.value})}
                  className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
                />

                <select
                  value={newBudget.period}
                  onChange={(e) => setNewBudget({...newBudget, period: e.target.value})}
                  className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
                >
                  <option value="mensual">Mensual</option>
                  <option value="semanal">Semanal</option>
                </select>

                <button
                  onClick={handleBudgetSubmit}
                  className="w-full bg-green-600 hover:bg-green-700 text-white font-semibold py-3 px-4 rounded-lg transition-colors flex items-center justify-center"
                >
                  <Target className="h-5 w-5 mr-2" />
                  Crear Presupuesto
                </button>
              </div>
            </div>

            <div className="bg-white rounded-xl shadow-lg p-6">
              <h3 className="text-lg font-semibold text-gray-800 mb-4">Estado de Presupuestos</h3>
              <div className="space-y-4 max-h-96 overflow-y-auto">
                {budgetAlerts.map(budget => (
                  <div key={budget.id} className="border border-gray-200 rounded-lg p-4">
                    <div className="flex justify-between items-start mb-2">
                      <h4 className="font-medium text-gray-800">{budget.category}</h4>
                      <button
                        onClick={() => deleteBudget(budget.id)}
                        className="text-red-600 hover:bg-red-100 rounded p-1"
                      >
                        <Trash2 className="h-4 w-4" />
                      </button>
                    </div>
                    <div className="space-y-2">
                      <div className="flex justify-between text-sm">
                        <span>Gastado: {formatCurrency(budget.spent)}</span>
                        <span>Límite: {formatCurrency(budget.limit)}</span>
                      </div>
                      <div className="w-full bg-gray-200 rounded-full h-2">
                        <div
                          className={`h-2 rounded-full transition-all ${
                            budget.percentage >= 100 ? 'bg-red-500' :
                            budget.percentage >= 80 ? 'bg-yellow-500' : 'bg-green-500'
                          }`}
                          style={{ width: `${Math.min(budget.percentage, 100)}%` }}
                        ></div>
                      </div>
                      <div className="flex justify-between text-sm">
                        <span className={
                          budget.status === 'exceeded' ? 'text-red-600' :
                          budget.status === 'warning' ? 'text-yellow-600' : 'text-green-600'
                        }>
                          {budget.percentage.toFixed(0)}% usado
                        </span>
                        <span className={budget.remaining >= 0 ? 'text-green-600' : 'text-red-600'}>
                          {budget.remaining >= 0 ? `Quedan ${formatCurrency(budget.remaining)}` : `Excedido por ${formatCurrency(Math.abs(budget.remaining))}`}
                        </span>
                      </div>
                    </div>
                  </div>
                ))}
              </div>
            </div>
          </div>
        )}

        {/* Tab: Proyecciones */}
        {activeTab === 'projections' && (
          <div className="space-y-6">
            <div className="bg-white rounded-xl shadow-lg p-6">
              <div className="flex justify-between items-center mb-4">
                <h3 className="text-lg font-semibold text-gray-800">Proyecciones Financieras</h3>
                <select
                  value={projectionMonths}
                  onChange={(e) => setProjectionMonths(parseInt(e.target.value))}
                  className="p-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
                >
                  <option value="3">3 meses</option>
                  <option value="6">6 meses</option>
                  <option value="12">12 meses</option>
                </select>
              </div>
              
              <ResponsiveContainer width="100%" height={400}>
                <LineChart data={projectionData}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="month" />
                  <YAxis tickFormatter={(value) => `${(value / 1000000).toFixed(1)}M`} />
                  <Tooltip formatter={(value) => formatCurrency(value)} />
                  <Legend />
                  <Line type="monotone" dataKey="ingresos" stroke="#10B981" strokeWidth={2} />
                  <Line type="monotone" dataKey="gastos" stroke="#EF4444" strokeWidth={2} />
                  <Line type="monotone" dataKey="acumulado" stroke="#3B82F6" strokeWidth={3} strokeDasharray="5 5" />
                </LineChart>
              </ResponsiveContainer>
            </div>

            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
              {projectionData.slice(0, 3).map((proj, index) => (
                <div key={proj.month} className="bg-white rounded-xl shadow-lg p-6">
                  <h4 className="font-semibold text-gray-800 mb-3">
                    {new Date(proj.month + '-01').toLocaleDateString('es', { month: 'long', year: 'numeric' })}
                  </h4>
                  <div className="space-y-2">
                    <div className="flex justify-between">
                      <span className="text-sm text-gray-600">Ingresos proyectados:</span>
                      <span className="font-medium text-green-600">{formatCurrency(proj.ingresos)}</span>
                    </div>
                    <div className="flex justify-between">
                      <span className="text-sm text-gray-600">Gastos proyectados:</span>
                      <span className="font-medium text-red-600">{formatCurrency(proj.gastos)}</span>
                    </div>
                    <div className="flex justify-between border-t pt-2">
                      <span className="text-sm font-medium text-gray-800">Balance mensual:</span>
                      <span className={`font-bold ${proj.balance >= 0 ? 'text-green-600' : 'text-red-600'}`}>
                        {formatCurrency(proj.balance)}
                      </span>
                    </div>
                    <div className="flex justify-between">
                      <span className="text-sm font-medium text-gray-800">Acumulado:</span>
                      <span className={`font-bold ${proj.acumulado >= 0 ? 'text-blue-600' : 'text-red-600'}`}>
                        {formatCurrency(proj.acumulado)}
                      </span>
                    </div>
                  </div>
                </div>
              ))}
            </div>

            <div className="bg-white rounded-xl shadow-lg p-6">
              <h3 className="text-lg font-semibold text-gray-800 mb-4">Análisis de Tendencias</h3>
              <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                <div className="space-y-4">
                  <h4 className="font-medium text-gray-700">Recomendaciones</h4>
                  <div className="space-y-2">
                    {balance > 0 ? (
                      <div className="flex items-start space-x-2">
                        <div className="w-2 h-2 bg-green-500 rounded-full mt-2"></div>
                        <p className="text-sm text-gray-600">Tienes un balance positivo. Considera aumentar tus ahorros o inversiones.</p>
                      </div>
                    ) : (
                      <div className="flex items-start space-x-2">
                        <div className="w-2 h-2 bg-red-500 rounded-full mt-2"></div>
                        <p className="text-sm text-gray-600">Tu balance es negativo. Revisa tus gastos y busca áreas de optimización.</p>
                      </div>
                    )}
                    
                    {totalDeudas > totalIngresos * 0.3 && (
                      <div className="flex items-start space-x-2">
                        <div className="w-2 h-2 bg-yellow-500 rounded-full mt-2"></div>
                        <p className="text-sm text-gray-600">Tus deudas representan más del 30% de tus ingresos. Considera un plan de reducción.</p>
                      </div>
                    )}
                    
                    <div className="flex items-start space-x-2">
                      <div className="w-2 h-2 bg-blue-500 rounded-full mt-2"></div>
                      <p className="text-sm text-gray-600">Mantén un fondo de emergencia equivalente a 3-6 meses de gastos.</p>
                    </div>
                  </div>
                </div>
                
                <div className="space-y-4">
                  <h4 className="font-medium text-gray-700">Métricas Clave</h4>
                  <div className="space-y-3">
                    <div className="flex justify-between">
                      <span className="text-sm text-gray-600">Tasa de ahorro:</span>
                      <span className="font-medium">{((balance / totalIngresos) * 100).toFixed(1)}%</span>
                    </div>
                    <div className="flex justify-between">
                      <span className="text-sm text-gray-600">Ratio deuda/ingresos:</span>
                      <span className="font-medium">{((totalDeudas / totalIngresos) * 100).toFixed(1)}%</span>
                    </div>
                    <div className="flex justify-between">
                      <span className="text-sm text-gray-600">Proyección 6 meses:</span>
                      <span className={`font-medium ${projectionData[5]?.acumulado >= 0 ? 'text-green-600' : 'text-red-600'}`}>
                        {formatCurrency(projectionData[5]?.acumulado || 0)}
                      </span>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        )}

        {/* Tab: Exportar */}
        {activeTab === 'export' && (
          <div className="space-y-6">
            <div className="bg-white rounded-xl shadow-lg p-6">
              <h3 className="text-lg font-semibold text-gray-800 mb-4">Exportar Datos</h3>
              <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                <div className="space-y-4">
                  <div className="border border-gray-200 rounded-lg p-4">
                    <h4 className="font-medium text-gray-800 mb-2">Exportar como JSON</h4>
                    <p className="text-sm text-gray-600 mb-4">
                      Descarga todos tus datos financieros en formato JSON, incluyendo transacciones, presupuestos y resumen.
                    </p>
                    <button
                      onClick={exportToJSON}
                      className="w-full bg-blue-600 hover:bg-blue-700 text-white font-semibold py-2 px-4 rounded-lg transition-colors flex items-center justify-center"
                    >
                      <Download className="h-4 w-4 mr-2" />
                      Descargar JSON
                    </button>
                  </div>
                </div>

                <div className="space-y-4">
                  <div className="border border-gray-200 rounded-lg p-4">
                    <h4 className="font-medium text-gray-800 mb-2">Exportar como CSV</h4>
                    <p className="text-sm text-gray-600 mb-4">
                      Descarga tus transacciones en formato CSV para usar en Excel, Google Sheets u otras aplicaciones.
                    </p>
                    <button
                      onClick={exportToCSV}
                      className="w-full bg-green-600 hover:bg-green-700 text-white font-semibold py-2 px-4 rounded-lg transition-colors flex items-center justify-center"
                    >
                      <Download className="h-4 w-4 mr-2" />
                      Descargar CSV
                    </button>
                  </div>
                </div>
              </div>
            </div>

            <div className="bg-white rounded-xl shadow-lg p-6">
              <h3 className="text-lg font-semibold text-gray-800 mb-4">Resumen de Datos</h3>
              <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
                <div className="text-center p-4 bg-gray-50 rounded-lg">
                  <p className="text-2xl font-bold text-blue-600">{transactions.length}</p>
                  <p className="text-sm text-gray-600">Total Transacciones</p>
                </div>
                <div className="text-center p-4 bg-gray-50 rounded-lg">
                  <p className="text-2xl font-bold text-green-600">{transactions.filter(t => t.type === 'ingreso').length}</p>
                  <p className="text-sm text-gray-600">Ingresos Registrados</p>
                </div>
                <div className="text-center p-4 bg-gray-50 rounded-lg">
                  <p className="text-2xl font-bold text-red-600">{transactions.filter(t => t.type === 'gasto').length}</p>
                  <p className="text-sm text-gray-600">Gastos Registrados</p>
                </div>
                <div className="text-center p-4 bg-gray-50 rounded-lg">
                  <p className="text-2xl font-bold text-purple-600">{budgets.length}</p>
                  <p className="text-sm text-gray-600">Presupuestos Activos</p>
                </div>
              </div>
            </div>
          </div>
        )}
      </div>
    </div>
  );
};

ReactDOM.render(<FinancialDashboard />, document.getElementById('root'));