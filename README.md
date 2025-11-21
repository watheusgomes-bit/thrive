# thrive
import React, { useState, useEffect } from 'react';
import { Calendar, TrendingUp, Award, Plus, Check, X, Edit2, Trash2, Target, Clock, Percent, Flame, ChevronRight, Sparkles } from 'lucide-react';

export default function ThriveApp() {
  const [activeTab, setActiveTab] = useState('habitos');
  const [habitos, setHabitos] = useState([]);
  const [projetos, setProjetos] = useState([]);
  const [marcos, setMarcos] = useState([]);
  const [showAddModal, setShowAddModal] = useState(false);
  const [editingItem, setEditingItem] = useState(null);
  const [stats, setStats] = useState({
    sequenciaAtual: 0,
    taxaConclusao: 0,
    habitosAtivos: 0,
    diasTotais: 0
  });

  const [newItem, setNewItem] = useState({
    titulo: '',
    descricao: '',
    frequencia: 'diaria',
    tipo: 'habito',
    progressoTipo: 'porcentagem',
    progressoAtual: 0,
    progressoMeta: 100,
    data: new Date().toISOString().split('T')[0],
    cor: 'emerald'
  });

  const cores = [
    { nome: 'emerald', bg: 'bg-emerald-500', hover: 'hover:bg-emerald-600', light: 'bg-emerald-50', border: 'border-emerald-200' },
    { nome: 'teal', bg: 'bg-teal-500', hover: 'hover:bg-teal-600', light: 'bg-teal-50', border: 'border-teal-200' },
    { nome: 'cyan', bg: 'bg-cyan-500', hover: 'hover:bg-cyan-600', light: 'bg-cyan-50', border: 'border-cyan-200' },
    { nome: 'blue', bg: 'bg-blue-500', hover: 'hover:bg-blue-600', light: 'bg-blue-50', border: 'border-blue-200' },
    { nome: 'purple', bg: 'bg-purple-500', hover: 'hover:bg-purple-600', light: 'bg-purple-50', border: 'border-purple-200' },
    { nome: 'amber', bg: 'bg-amber-500', hover: 'hover:bg-amber-600', light: 'bg-amber-50', border: 'border-amber-200' }
  ];

  // Carregar dados do localStorage
  useEffect(() => {
    loadData();
  }, []);

  useEffect(() => {
    calcularEstatisticas();
  }, [habitos, projetos, marcos]);

  const loadData = () => {
    try {
      const habitosData = localStorage.getItem('thrive-habitos');
      const projetosData = localStorage.getItem('thrive-projetos');
      const marcosData = localStorage.getItem('thrive-marcos');
      
      if (habitosData) setHabitos(JSON.parse(habitosData));
      if (projetosData) setProjetos(JSON.parse(projetosData));
      if (marcosData) setMarcos(JSON.parse(marcosData));
    } catch (error) {
      console.log('Primeira vez usando o app - iniciando com dados vazios');
    }
  };

  const saveData = (type, data) => {
    try {
      localStorage.setItem(`thrive-${type}`, JSON.stringify(data));
    } catch (error) {
      console.error('Erro ao salvar:', error);
    }
  };

  const calcularEstatisticas = () => {
    const hoje = new Date().toISOString().split('T')[0];
    
    // Maior sequência atual entre todos os hábitos
    const maiorSequencia = Math.max(...habitos.map(h => h.sequencia || 0), 0);
    
    // Taxa de conclusão geral (hábitos marcados hoje)
    const habitosMarcadosHoje = habitos.filter(h => h.ultimaMarcacao === hoje).length;
    const taxaConclusao = habitos.length > 0 ? Math.round((habitosMarcadosHoje / habitos.length) * 100) : 0;
    
    // Total de dias registrados (únicos)
    const todosDias = new Set();
    habitos.forEach(h => h.historico?.forEach(d => todosDias.add(d)));
    
    setStats({
      sequenciaAtual: maiorSequencia,
      taxaConclusao,
      habitosAtivos: habitos.length,
      diasTotais: todosDias.size
    });
  };

  const calcularDiasDesdeMarco = (dataMarco) => {
    const umDia = 24 * 60 * 60 * 1000;
    const hoje = new Date();
    const data = new Date(dataMarco);
    return Math.floor((hoje - data) / umDia);
  };

  const getCor = (nome) => cores.find(c => c.nome === nome) || cores[0];

  const addItem = () => {
    const item = {
      ...newItem,
      id: Date.now(),
      sequencia: 0,
      ultimaMarcacao: null,
      historico: [],
      dataCriacao: new Date().toISOString()
    };

    if (activeTab === 'habitos') {
      const updated = [...habitos, item];
      setHabitos(updated);
      saveData('habitos', updated);
    } else if (activeTab === 'projetos') {
      const updated = [...projetos, item];
      setProjetos(updated);
      saveData('projetos', updated);
    } else {
      const updated = [...marcos, item];
      setMarcos(updated);
      saveData('marcos', updated);
    }

    resetModal();
  };

  const updateItem = () => {
    if (activeTab === 'habitos') {
      const updated = habitos.map(h => h.id === editingItem.id ? { ...h, ...newItem } : h);
      setHabitos(updated);
      saveData('habitos', updated);
    } else if (activeTab === 'projetos') {
      const updated = projetos.map(p => p.id === editingItem.id ? { ...p, ...newItem } : p);
      setProjetos(updated);
      saveData('projetos', updated);
    } else {
      const updated = marcos.map(m => m.id === editingItem.id ? { ...m, ...newItem } : m);
      setMarcos(updated);
      saveData('marcos', updated);
    }
    resetModal();
  };

  const deleteItem = (id) => {
    if (confirm('Tem certeza que deseja excluir este item?')) {
      if (activeTab === 'habitos') {
        const updated = habitos.filter(h => h.id !== id);
        setHabitos(updated);
        saveData('habitos', updated);
      } else if (activeTab === 'projetos') {
        const updated = projetos.filter(p => p.id !== id);
        setProjetos(updated);
        saveData('projetos', updated);
      } else {
        const updated = marcos.filter(m => m.id !== id);
        setMarcos(updated);
        saveData('marcos', updated);
      }
    }
  };

  const marcarHabito = (id) => {
    const hoje = new Date().toISOString().split('T')[0];
    const updated = habitos.map(h => {
      if (h.id === id) {
        const jaMarcado = h.ultimaMarcacao === hoje;
        const novaSequencia = jaMarcado ? Math.max(0, h.sequencia - 1) : h.sequencia + 1;
        
        return {
          ...h,
          sequencia: novaSequencia,
          ultimaMarcacao: jaMarcado ? null : hoje,
          historico: jaMarcado 
            ? h.historico.filter(d => d !== hoje)
            : [...(h.historico || []), hoje]
        };
      }
      return h;
    });
    setHabitos(updated);
    saveData('habitos', updated);
  };

  const atualizarProgresso = (id, valor) => {
    const updated = projetos.map(p => {
      if (p.id === id) {
        return { ...p, progressoAtual: Math.min(Math.max(0, valor), p.progressoMeta) };
      }
      return p;
    });
    setProjetos(updated);
    saveData('projetos', updated);
  };

  const resetModal = () => {
    setShowAddModal(false);
    setEditingItem(null);
    setNewItem({
      titulo: '',
      descricao: '',
      frequencia: 'diaria',
      tipo: 'habito',
      progressoTipo: 'porcentagem',
      progressoAtual: 0,
      progressoMeta: 100,
      data: new Date().toISOString().split('T')[0],
      cor: 'emerald'
    });
  };

  const openEditModal = (item) => {
    setEditingItem(item);
    setNewItem(item);
    setShowAddModal(true);
  };

  const hoje = new Date().toISOString().split('T')[0];

  return (
    <div className="min-h-screen bg-gradient-to-br from-emerald-50 via-teal-50 to-cyan-50 p-4 md:p-8">
      <div className="max-w-6xl mx-auto">
        {/* Header */}
        <div className="text-center mb-8">
          <div className="flex items-center justify-center gap-3 mb-2">
            <div className="w-12 h-12 bg-gradient-to-r from-emerald-500 to-teal-500 rounded-2xl flex items-center justify-center shadow-lg">
              <Sparkles className="w-6 h-6 text-white" />
            </div>
            <h1 className="text-4xl md:text-5xl font-bold text-emerald-800">Thrive</h1>
          </div>
          <p className="text-emerald-600 text-lg">Cultivando uma vida com propósito</p>
        </div>

        {/* Stats Cards */}
        <div className="grid grid-cols-2 md:grid-cols-4 gap-4 mb-6">
          <div className="bg-white rounded-2xl p-4 shadow-sm border border-emerald-100">
            <div className="flex items-center gap-3">
              <div className="w-10 h-10 bg-orange-100 rounded-lg flex items-center justify-center">
                <Flame className="w-5 h-5 text-orange-600" />
              </div>
              <div>
                <p className="text-2xl font-bold text-gray-800">{stats.sequenciaAtual}</p>
                <p className="text-xs text-gray-500">Sequência</p>
              </div>
            </div>
          </div>
          <div className="bg-white rounded-2xl p-4 shadow-sm border border-emerald-100">
            <div className="flex items-center gap-3">
              <div className="w-10 h-10 bg-emerald-100 rounded-lg flex items-center justify-center">
                <TrendingUp className="w-5 h-5 text-emerald-600" />
              </div>
              <div>
                <p className="text-2xl font-bold text-gray-800">{stats.taxaConclusao}%</p>
                <p className="text-xs text-gray-500">Conclusão</p>
              </div>
            </div>
          </div>
          <div className="bg-white rounded-2xl p-4 shadow-sm border border-emerald-100">
            <div className="flex items-center gap-3">
              <div className="w-10 h-10 bg-cyan-100 rounded-lg flex items-center justify-center">
                <Target className="w-5 h-5 text-cyan-600" />
              </div>
              <div>
                <p className="text-2xl font-bold text-gray-800">{stats.habitosAtivos}</p>
                <p className="text-xs text-gray-500">Hábitos</p>
              </div>
            </div>
          </div>
          <div className="bg-white rounded-2xl p-4 shadow-sm border border-emerald-100">
            <div className="flex items-center gap-3">
              <div className="w-10 h-10 bg-purple-100 rounded-lg flex items-center justify-center">
                <Calendar className="w-5 h-5 text-purple-600" />
              </div>
              <div>
                <p className="text-2xl font-bold text-gray-800">{stats.diasTotais}</p>
                <p className="text-xs text-gray-500">Dias Ativos</p>
              </div>
            </div>
          </div>
        </div>

        {/* Tabs */}
        <div className="flex gap-2 mb-6 bg-white rounded-2xl p-2 shadow-sm border border-emerald-100">
          {[
            { id: 'habitos', icon: Calendar, label: 'Hábitos', color: 'emerald' },
            { id: 'projetos', icon: TrendingUp, label: 'Projetos', color: 'teal' },
            { id: 'marcos', icon: Award, label: 'Marcos', color: 'cyan' }
          ].map(tab => {
            const Icon = tab.icon;
            const cor = getCor(tab.color);
            return (
              <button
                key={tab.id}
                onClick={() => setActiveTab(tab.id)}
                className={`flex-1 py-3 px-4 rounded-xl font-medium transition-all flex items-center justify-center gap-2 ${
                  activeTab === tab.id
                    ? `${cor.bg} text-white shadow-md`
                    : 'text-gray-600 hover:bg-gray-50'
                }`}
              >
                <Icon className="w-5 h-5" />
                <span className="hidden sm:inline">{tab.label}</span>
              </button>
            );
          })}
        </div>

        {/* Content */}
        <div className="bg-white rounded-2xl shadow-lg border border-emerald-100 p-6 mb-6 min-h-96">
          {/* Hábitos */}
          {activeTab === 'habitos' && (
            <div>
              <div className="flex justify-between items-center mb-6">
                <div>
                  <h2 className="text-2xl font-bold text-gray-800">Seus Hábitos</h2>
                  <p className="text-gray-500 text-sm">Ações diárias que transformam sua rotina</p>
                </div>
                <button
                  onClick={() => setShowAddModal(true)}
                  className="bg-emerald-500 text-white px-4 py-2 rounded-xl hover:bg-emerald-600 transition-colors flex items-center gap-2 shadow-sm"
                >
                  <Plus className="w-5 h-5" />
                  Novo Hábito
                </button>
              </div>
              
              {habitos.length === 0 ? (
                <div className="text-center py-12 text-gray-400">
                  <Calendar className="w-16 h-16 mx-auto mb-4 opacity-50" />
                  <p className="text-lg mb-2">Nenhum hábito ainda</p>
                  <p className="text-sm">Comece adicionando seu primeiro hábito!</p>
                </div>
              ) : (
                <div className="grid gap-3">
                  {habitos.map(habito => {
                    const cor = getCor(habito.cor);
                    return (
                      <div key={habito.id} className={`border ${cor.border} rounded-xl p-4 hover:shadow-md transition-all duration-300`}>
                        <div className="flex items-start justify-between">
                          <div className="flex items-start gap-3 flex-1">
                            <button
                              onClick={() => marcarHabito(habito.id)}
                              className={`w-10 h-10 rounded-xl flex items-center justify-center transition-all mt-1 ${
                                habito.ultimaMarcacao === hoje
                                  ? `${cor.bg} text-white shadow-sm`
                                  : `border-2 ${cor.border} text-gray-300 hover:${cor.bg} hover:text-white`
                              }`}
                            >
                              {habito.ultimaMarcacao === hoje && <Check className="w-5 h-5" />}
                            </button>
                            <div className="flex-1">
                              <h3 className="font-semibold text-gray-800 mb-1">{habito.titulo}</h3>
                              {habito.descricao && (
                                <p className="text-sm text-gray-500 mb-2">{habito.descricao}</p>
                              )}
                              <div className="flex items-center gap-4 text-sm">
                                {habito.sequencia > 0 && (
                                  <span className={`${cor.bg} text-white px-2 py-1 rounded-lg text-xs font-medium flex items-center gap-1`}>
                                    <Flame className="w-3 h-3" />
                                    {habito.sequencia} dia{habito.sequencia !== 1 ? 's' : ''}
                                  </span>
                                )}
                                <span className="text-gray-500 capitalize">{habito.frequencia}</span>
                              </div>
                            </div>
                          </div>
                          <div className="flex gap-1">
                            <button
                              onClick={() => openEditModal(habito)}
                              className="p-2 text-gray-400 hover:text-gray-600 transition-colors"
                            >
                              <Edit2 className="w-4 h-4" />
                            </button>
                            <button
                              onClick={() => deleteItem(habito.id)}
                              className="p-2 text-gray-400 hover:text-red-600 transition-colors"
                            >
                              <Trash2 className="w-4 h-4" />
                            </button>
                          </div>
                        </div>
                      </div>
                    );
                  })}
                </div>
              )}
            </div>
          )}

          {/* Projetos */}
          {activeTab === 'projetos' && (
            <div>
              <div className="flex justify-between items-center mb-6">
                <div>
                  <h2 className="text-2xl font-bold text-gray-800">Seus Projetos</h2>
                  <p className="text-gray-500 text-sm">Jornadas contínuas de crescimento</p>
                </div>
                <button
                  onClick={() => setShowAddModal(true)}
                  className="bg-teal-500 text-white px-4 py-2 rounded-xl hover:bg-teal-600 transition-colors flex items-center gap-2 shadow-sm"
                >
                  <Plus className="w-5 h-5" />
                  Novo Projeto
                </button>
              </div>
              
              {projetos.length === 0 ? (
                <div className="text-center py-12 text-gray-400">
                  <TrendingUp className="w-16 h-16 mx-auto mb-4 opacity-50" />
                  <p className="text-lg mb-2">Nenhum projeto ainda</p>
                  <p className="text-sm">Comece uma nova jornada de aprendizado!</p>
                </div>
              ) : (
                <div className="grid gap-4">
                  {projetos.map(projeto => {
                    const cor = getCor(projeto.cor);
                    const progressoPercent = (projeto.progressoAtual / projeto.progressoMeta) * 100;
                    return (
                      <div key={projeto.id} className={`border ${cor.border} rounded-xl p-4 hover:shadow-md transition-all duration-300`}>
                        <div className="flex items-start justify-between mb-4">
                          <div className="flex-1">
                            <h3 className="font-semibold text-lg text-gray-800 mb-1">{projeto.titulo}</h3>
                            <p className="text-sm text-gray-500">{projeto.descricao}</p>
                          </div>
                          <div className="flex gap-1">
                            <button
                              onClick={() => openEditModal(projeto)}
                              className="p-2 text-gray-400 hover:text-gray-600 transition-colors"
                            >
                              <Edit2 className="w-4 h-4" />
                            </button>
                            <button
                              onClick={() => deleteItem(projeto.id)}
                              className="p-2 text-gray-400 hover:text-red-600 transition-colors"
                            >
                              <Trash2 className="w-4 h-4" />
                            </button>
                          </div>
                        </div>
                        
                        <div className="space-y-3">
                          <div className="flex items-center justify-between text-sm">
                            <span className="text-gray-600">Progresso</span>
                            <span className="font-medium text-gray-700">
                              {projeto.progressoAtual} / {projeto.progressoMeta}
                              {projeto.progressoTipo === 'porcentagem' && '%'}
                            </span>
                          </div>
                          <div className="w-full bg-gray-200 rounded-full h-3">
                            <div
                              className={`${cor.bg} h-3 rounded-full transition-all duration-500`}
                              style={{ width: `${progressoPercent}%` }}
                            />
                          </div>
                          <div className="flex items-center gap-3">
                            <input
                              type="range"
                              min="0"
                              max={projeto.progressoMeta}
                              value={projeto.progressoAtual}
                              onChange={(e) => atualizarProgresso(projeto.id, parseInt(e.target.value))}
                              className="flex-1"
                            />
                            <input
                              type="number"
                              min="0"
                              max={projeto.progressoMeta}
                              value={projeto.progressoAtual}
                              onChange={(e) => atualizarProgresso(projeto.id, parseInt(e.target.value) || 0)}
                              className="w-20 px-2 py-1 border border-gray-300 rounded-lg text-sm text-center"
                            />
                          </div>
                        </div>
                      </div>
                    );
                  })}
                </div>
              )}
            </div>
          )}

          {/* Marcos */}
          {activeTab === 'marcos' && (
            <div>
              <div className="flex justify-between items-center mb-6">
                <div>
                  <h2 className="text-2xl font-bold text-gray-800">Seus Marcos</h2>
                  <p className="text-gray-500 text-sm">Momentos que marcaram sua jornada</p>
                </div>
                <button
                  onClick={() => setShowAddModal(true)}
                  className="bg-cyan-500 text-white px-4 py-2 rounded-xl hover:bg-cyan-600 transition-colors flex items-center gap-2 shadow-sm"
                >
                  <Plus className="w-5 h-5" />
                  Novo Marco
                </button>
              </div>
              
              {marcos.length === 0 ? (
                <div className="text-center py-12 text-gray-400">
                  <Award className="w-16 h-16 mx-auto mb-4 opacity-50" />
                  <p className="text-lg mb-2">Nenhum marco ainda</p>
                  <p className="text-sm">Celebre suas conquistas e momentos especiais!</p>
                </div>
              ) : (
                <div className="grid gap-4">
                  {marcos.sort((a, b) => new Date(b.data) - new Date(a.data)).map(marco => {
                    const cor = getCor(marco.cor);
                    const diasDesde = calcularDiasDesdeMarco(marco.data);
                    return (
                      <div key={marco.id} className={`border ${cor.border} rounded-xl p-4 hover:shadow-md transition-all duration-300`}>
                        <div className="flex items-start justify-between">
                          <div className="flex gap-4 flex-1">
                            <div className={`w-12 h-12 ${cor.light} rounded-xl flex items-center justify-center flex-shrink-0`}>
                              <Award className={`w-6 h-6 ${cor.bg.replace('bg-', 'text-')}`} />
                            </div>
                            <div className="flex-1">
                              <h3 className="font-semibold text-lg text-gray-800 mb-1">{marco.titulo}</h3>
                              <p className="text-sm text-gray-500 mb-2">{marco.descricao}</p>
                              <div className="flex items-center gap-4 text-sm">
                                <span className="text-gray-600 font-medium">
                                  {new Date(marco.data).toLocaleDateString('pt-BR', { 
                                    day: 'numeric', 
                                    month: 'long', 
                                    year: 'numeric' 
                                  })}
                                </span>
                                <span className={`${cor.bg} text-white px-2 py-1 rounded-lg text-xs font-medium`}>
                                  Há {diasDesde} dia{diasDesde !== 1 ? 's' : ''}
                                </span>
                              </div>
                            </div>
                          </div>
                          <div className="flex gap-1">
                            <button
                              onClick={() => openEditModal(marco)}
                              className="p-2 text-gray-400 hover:text-gray-600 transition-colors"
                            >
                              <Edit2 className="w-4 h-4" />
                            </button>
                            <button
                              onClick={() => deleteItem(marco.id)}
                              className="p-2 text-gray-400 hover:text-red-600 transition-colors"
                            >
                              <Trash2 className="w-4 h-4" />
                            </button>
                          </div>
                        </div>
                      </div>
                    );
                  })}
                </div>
              )}
            </div>
          )}
        </div>

        {/* Modal */}
        {showAddModal && (
          <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
            <div className="bg-white rounded-2xl p-6 max-w-md w-full max-h-[90vh] overflow-y-auto">
              <div className="flex justify-between items-center mb-6">
                <h3 className="text-xl font-bold text-gray-800">
                  {editingItem ? 'Editar' : 'Novo'} {
                    activeTab === 'habitos' ? 'Hábito' :
                    activeTab === 'projetos' ? 'Projeto' : 'Marco'
                  }
                </h3>
                <button onClick={resetModal} className="text-gray-400 hover:text-gray-600 transition-colors">
                  <X className="w-6 h-6" />
                </button>
              </div>

              <div className="space-y-4">
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-2">Título *</label>
                  <input
                    type="text"
                    value={newItem.titulo}
                    onChange={(e) => setNewItem({ ...newItem, titulo: e.target.value })}
                    className="w-full px-4 py-3 border border-gray-300 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:border-transparent transition-all"
                    placeholder={
                      activeTab === 'habitos' ? 'Ex: Meditar pela manhã' :
                      activeTab === 'projetos' ? 'Ex: Aprender violão' :
                      'Ex: Aceitei Jesus'
                    }
                  />
                </div>

                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-2">Descrição</label>
                  <textarea
                    value={newItem.descricao}
                    onChange={(e) => setNewItem({ ...newItem, descricao: e.target.value })}
                    className="w-full px-4 py-3 border border-gray-300 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:border-transparent transition-all"
                    placeholder="Detalhes opcionais..."
                    rows="3"
                  />
                </div>

                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-2">Cor</label>
                  <div className="grid grid-cols-6 gap-2">
                    {cores.map(cor => (
                      <button
                        key={cor.nome}
                        onClick={() => setNewItem({ ...newItem, cor: cor.nome })}
                        className={`w-8 h-8 rounded-lg ${cor.bg} transition-transform ${
                          newItem.cor === cor.nome ? 'ring-2 ring-offset-2 ring-gray-400 scale-110' : ''
                        }`}
                      />
                    ))}
                  </div>
                </div>

                {activeTab === 'habitos' && (
                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-2">Frequência</label>
                    <select
                      value={newItem.frequencia}
                      onChange={(e) => setNewItem({ ...newItem, frequencia: e.target.value })}
                      className="w-full px-4 py-3 border border-gray-300 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:border-transparent"
                    >
                      <option value="diaria">Diária</option>
                      <option value="semanal">Semanal</option>
                      <option value="mensal">Mensal</option>
                    </select>
                  </div>
                )}

                {activeTab === 'projetos' && (
                  <div className="grid grid-cols-2 gap-4">
                    <div>
                      <label className="block text-sm font-medium text-gray-700 mb-2">Progresso Atual</label>
                      <input
                        type="number"
                        value={newItem.progressoAtual}
                        onChange={(e) => setNewItem({ ...newItem, progressoAtual: parseInt(e.target.value) || 0 })}
                        className="w-full px-4 py-3 border border-gray-300 rounded-xl focus:ring-2 focus:ring-teal-500 focus:border-transparent"
                      />
                    </div>
                    <div>
                      <label className="block text-sm font-medium text-gray-700 mb-2">Meta</label>
                      <input
                        type="number"
                        value={newItem.progressoMeta}
                        onChange={(e) => setNewItem({ ...newItem, progressoMeta: parseInt(e.target.value) || 100 })}
                        className="w-full px-4 py-3 border border-gray-300 rounded-xl focus:ring-2 focus:ring-teal-500 focus:border-transparent"
                      />
                    </div>
                  </div>
                )}

                {activeTab === 'marcos' && (
                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-2">Data</label>
                    <input
                      type="date"
                      value={newItem.data}
                      onChange={(e) => setNewItem({ ...newItem, data: e.target.value })}
                      className="w-full px-4 py-3 border border-gray-300 rounded-xl focus:ring-2 focus:ring-cyan-500 focus:border-transparent"
                    />
                  </div>
                )}

                <div className="flex gap-3 pt-4">
                  <button
                    onClick={resetModal}
                    className="flex-1 py-3 border border-gray-300 text-gray-700 rounded-xl font-medium hover:bg-gray-50 transition-colors"
                  >
                    Cancelar
                  </button>
                  <button
                    onClick={editingItem ? updateItem : addItem}
                    disabled={!newItem.titulo.trim()}
                    className={`flex-1 py-3 rounded-xl font-medium transition-colors ${
                      activeTab === 'habitos' ? 'bg-emerald-500 hover:bg-emerald-600' :
                      activeTab === 'projetos' ? 'bg-teal-500 hover:bg-teal-600' :
                      'bg-cyan-500 hover:bg-cyan-600'
                    } text-white disabled:opacity-50 disabled:cursor-not-allowed`}
                  >
                    {editingItem ? 'Atualizar' : 'Adicionar'}
                  </button>
                </div>
              </div>
            </div>
          </div>
        )}

        {/* Mensagem motivacional */}
        <div className="bg-gradient-to-r from-emerald-500 to-teal-500 rounded-2xl p-6 text-white text-center shadow-lg">
          <p className="text-lg font-medium mb-2">
            "O verdadeiro crescimento floresce não na pressa, mas na paciência consigo mesmo."
          </p>
          <p className="text-emerald-100 text-sm">
            Cada pequena ação conta. Você está progredindo! 🌱
          </p>
        </div>
      </div>
    </div>
  );
}
