import React, { useState, useEffect } from 'react';
import { 
  Menu, X, Instagram, Phone, MapPin, 
  Ruler, HardHat, FileCheck, Gem, 
  ArrowRight, CheckCircle, Mail, 
  Lock, LogOut, Plus, Trash2, Save, Edit3, Image as ImageIcon, Link as LinkIcon
} from 'lucide-react';

// Importação dos módulos do Firebase
import { initializeApp } from 'firebase/app';
import { 
  getFirestore, collection, doc, onSnapshot, 
  updateDoc, setDoc, addDoc, deleteDoc 
} from 'firebase/firestore';
import { 
  getAuth, signInAnonymously, 
  signInWithCustomToken, onAuthStateChanged 
} from 'firebase/auth';

// --- CONFIGURAÇÃO DO FIREBASE ---

// 1. Suas credenciais reais
const userFirebaseConfig = {
  apiKey: "AIzaSyBBlolaRJ5Ihi09I6Hc0FvV82o6ZtWDr-w",
  authDomain: "studiodea-859b4.firebaseapp.com",
  projectId: "studiodea-859b4",
  storageBucket: "studiodea-859b4.firebasestorage.app",
  messagingSenderId: "141547468660",
  appId: "1:141547468660:web:06c1b0dfd91ed698301e21",
  measurementId: "G-4541JCEYRL"
};

// 2. Lógica de seleção de ambiente
const isPreviewEnv = typeof __firebase_config !== 'undefined';
const firebaseConfig = isPreviewEnv ? JSON.parse(__firebase_config) : userFirebaseConfig;

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);

// Se estiver no preview, usa ID dinâmico. Em produção, usa 'prod'.
const appId = typeof __app_id !== 'undefined' ? __app_id : 'studiodea-prod';

// Coleção base para dados públicos
// Caminho: artifacts -> {appId} -> public -> data (Documento base)
const PUBLIC_DATA_PATH = ['artifacts', appId, 'public', 'data'];

const App = () => {
  // --- ESTADOS DE DADOS (CONTEÚDO DO SITE) ---
  const [siteData, setSiteData] = useState(null); 
  const [projects, setProjects] = useState([]);
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  // --- ESTADOS DE CONTROLE UI ---
  const [isMenuOpen, setIsMenuOpen] = useState(false);
  const [isAdmin, setIsAdmin] = useState(false);
  const [showLoginModal, setShowLoginModal] = useState(false);
  const [loginUser, setLoginUser] = useState("");
  const [loginPass, setLoginPass] = useState("");
  const [loginError, setLoginError] = useState("");
  
  // Edição
  const [editingProject, setEditingProject] = useState(null);
  const [projectForm, setProjectForm] = useState({ title: "", category: "", image: "" });
  const [showProjectForm, setShowProjectForm] = useState(false);
  const [isEditingText, setIsEditingText] = useState(false);
  const [tempSiteData, setTempSiteData] = useState({});

  // --- DADOS PADRÃO ---
  const defaultSiteData = {
    heroImage: "https://images.unsplash.com/photo-1600607687939-ce8a6c25118c?ixlib=rb-1.2.1&auto=format&fit=crop&w=1920&q=80",
    aboutImage: "https://images.unsplash.com/photo-1556912173-3db996ea0622?ixlib=rb-1.2.1&auto=format&fit=crop&w=800&q=80",
    heroTitle: "Engenharia, Arquitetura e Design Integrados",
    heroSubtitle: "Da regularização à especificação de acabamentos de alto padrão. Transformamos espaços com técnica apurada e estética refinada em Sete Lagoas e região.",
    btnHeroPrimary: "Solicitar Orçamento",
    btnHeroSecondary: "Nossos Serviços",
    aboutLabel: "Quem Somos",
    aboutTitle: "Sobre o Studio Dea",
    aboutText: "Localizado no bairro Manoa em Sete Lagoas, o Studio Dea nasce da união entre a precisão da engenharia e a sensibilidade da arquitetura.",
    aboutBoxTitle: "Expertise Diferenciada",
    aboutBoxText: "Atuação direta como Especificadores de Alto Padrão na Ideale Acabamentos, garantindo a melhor curadoria de materiais para sua obra.",
    contactPhone: "(31) 99350-4513",
    contactEmail: "flaviohenrique.eng@gmail.com",
    contactAddress: "Rua Tapajós, 285 D - Bairro Manoa, Sete Lagoas - MG",
    whatsappLink: "https://wa.me/5531993504513",
    instagramLink: "https://instagram.com/studiodea7l",
    regTitle: "Sua obra está irregular?",
    regText: "Evite multas e desvalorização do seu imóvel. Atuamos com despachante técnico para regularização de obras em Sete Lagoas e região.",
    btnReg: "Falar com Especialista"
  };

  const services = [
    { icon: <Ruler className="w-10 h-10 text-orange-500" />, title: "Arquitetura e Design", description: "Projetos arquitetônicos residenciais e comerciais com foco em funcionalidade e estética de alto padrão." },
    { icon: <HardHat className="w-10 h-10 text-orange-500" />, title: "Engenharia e Obras", description: "Execução e acompanhamento técnico de obras, garantindo segurança, prazo e fidelidade ao projeto." },
    { icon: <FileCheck className="w-10 h-10 text-orange-500" />, title: "Regularização e Despachante", description: "Resolvemos a burocracia. Regularização de obras, habite-se, processos na prefeitura e averbações." },
    { icon: <Gem className="w-10 h-10 text-orange-500" />, title: "Especificação de Alto Padrão", description: "Curadoria especializada em acabamentos. Expertise adquirida como especificador na Ideale Acabamentos." }
  ];

  // --- EFEITOS DO FIREBASE ---
  useEffect(() => {
    const initAuth = async () => {
      if (isPreviewEnv && typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
        try {
          await signInWithCustomToken(auth, __initial_auth_token);
        } catch (err) {
          console.error("Erro auth token preview:", err);
          await signInAnonymously(auth);
        }
      } else {
        try {
          await signInAnonymously(auth);
        } catch (err) {
          console.error("Erro login anônimo:", err);
        }
      }
    };
    initAuth();
    const unsubscribe = onAuthStateChanged(auth, setUser);
    return () => unsubscribe();
  }, []);

  useEffect(() => {
    if (!user) return;

    // artifacts/appId/public/data/content/main
    const contentRef = doc(db, ...PUBLIC_DATA_PATH, 'content', 'main');
    
    const unsubContent = onSnapshot(contentRef, (docSnap) => {
      if (docSnap.exists()) {
        const data = docSnap.data();
        setSiteData(data);
        if (!isEditingText) setTempSiteData(data);
      } else {
        setDoc(contentRef, defaultSiteData);
        setSiteData(defaultSiteData);
        setTempSiteData(defaultSiteData);
      }
      setLoading(false);
    }, (error) => {
      console.error("Erro ao ler conteúdo:", error);
      if (!siteData) {
        setSiteData(defaultSiteData);
        setLoading(false);
      }
    });

    // artifacts/appId/public/data/projects (coleção)
    const projectsRef = collection(db, ...PUBLIC_DATA_PATH, 'projects');
    
    const unsubProjects = onSnapshot(projectsRef, (snapshot) => {
      const projs = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      projs.sort((a, b) => (b.createdAt || 0) - (a.createdAt || 0));
      setProjects(projs);
    }, (error) => console.error("Erro ao ler projetos:", error));

    return () => {
      unsubContent();
      unsubProjects();
    };
  }, [user]);

  // --- FUNÇÕES ADMIN ---
  const handleLogin = (e) => {
    e.preventDefault();
    if (loginUser === "admin" && loginPass === "admin123") {
      setIsAdmin(true);
      setShowLoginModal(false);
      setLoginUser("");
      setLoginPass("");
      setLoginError("");
    } else {
      setLoginError("Usuário ou senha incorretos.");
    }
  };

  const handleLogout = () => {
    setIsAdmin(false);
    setIsEditingText(false);
    setShowProjectForm(false);
    setEditingProject(null);
  };

  const handleSaveTexts = async () => {
    if (!user) return;
    try {
      const contentRef = doc(db, ...PUBLIC_DATA_PATH, 'content', 'main');
      await updateDoc(contentRef, tempSiteData);
      setIsEditingText(false);
      alert("Conteúdo salvo no banco de dados!");
    } catch (e) {
      alert("Erro ao salvar: " + e.message);
    }
  };

  const handleChange = (field, value) => {
    setTempSiteData({ ...tempSiteData, [field]: value });
  };

  const openProjectForm = (project = null) => {
    if (project) {
      setEditingProject(project);
      setProjectForm(project);
    } else {
      setEditingProject(null);
      setProjectForm({ title: "", category: "", image: "" });
    }
    setShowProjectForm(true);
  };

  const handleSaveProject = async () => {
    if (!projectForm.title || !projectForm.image || !user) return;

    try {
      if (editingProject) {
        const projRef = doc(db, ...PUBLIC_DATA_PATH, 'projects', editingProject.id);
        await updateDoc(projRef, { ...projectForm });
      } else {
        const projectsRef = collection(db, ...PUBLIC_DATA_PATH, 'projects');
        await addDoc(projectsRef, { ...projectForm, createdAt: Date.now() });
      }
      setShowProjectForm(false);
    } catch (e) {
      alert("Erro ao salvar projeto: " + e.message);
    }
  };

  const handleDeleteProject = async (id) => {
    if (window.confirm("Tem certeza que deseja excluir este projeto permanentemente?")) {
      try {
        await deleteDoc(doc(db, ...PUBLIC_DATA_PATH, 'projects', id));
      } catch (e) {
        alert("Erro ao excluir: " + e.message);
      }
    }
  };

  const toggleMenu = () => setIsMenuOpen(!isMenuOpen);

  if (loading || !siteData) {
    return (
      <div className="h-screen flex items-center justify-center bg-gray-900 text-white">
        <div className="animate-pulse flex flex-col items-center">
          <div className="h-12 w-12 border-4 border-orange-500 border-t-transparent rounded-full animate-spin mb-4"></div>
          <p>Carregando Studio Dea...</p>
        </div>
      </div>
    );
  }

  return (
    <div className="font-sans text-gray-800 bg-gray-50 pb-16">
      
      {/* BARRA ADMIN */}
      {isAdmin && (
        <div className="fixed top-0 w-full bg-gray-900 text-white z-[60] shadow-lg border-b-4 border-orange-500">
          <div className="max-w-7xl mx-auto px-4 h-16 flex justify-between items-center">
            <div className="flex items-center gap-2">
              <Lock className="text-orange-500" size={20} />
              <span className="font-bold hidden sm:inline">Modo Administrador (Online)</span>
            </div>
            <div className="flex gap-2 sm:gap-4">
               {!isEditingText ? (
                 <button onClick={() => { setIsEditingText(true); setTempSiteData(siteData); }} className="flex items-center gap-2 bg-blue-600 hover:bg-blue-700 px-3 py-1 rounded text-sm transition">
                   <Edit3 size={16} /> <span className="hidden sm:inline">Editar Textos</span>
                 </button>
               ) : (
                 <>
                   <button onClick={handleSaveTexts} className="flex items-center gap-2 bg-green-600 hover:bg-green-700 px-3 py-1 rounded text-sm transition">
                     <Save size={16} /> <span className="hidden sm:inline">Salvar no Banco</span>
                   </button>
                   <button onClick={() => { setIsEditingText(false); setTempSiteData(siteData); }} className="flex items-center gap-2 bg-red-600 hover:bg-red-700 px-3 py-1 rounded text-sm transition">
                     <X size={16} /> <span className="hidden sm:inline">Cancelar</span>
                   </button>
                 </>
               )}
               <button onClick={handleLogout} className="flex items-center gap-2 bg-gray-700 hover:bg-gray-600 px-3 py-1 rounded text-sm transition">
                 <LogOut size={16} /> <span className="hidden sm:inline">Sair</span>
               </button>
            </div>
          </div>
        </div>
      )}

      {/* LOGIN MODAL */}
      {showLoginModal && (
        <div className="fixed inset-0 bg-black z-[70] flex items-center justify-center p-4">
          <div className="bg-white rounded-lg p-8 max-w-sm w-full shadow-2xl">
            <div className="flex justify-between items-center mb-6">
              <h3 className="text-xl font-bold text-gray-900">Acesso Restrito</h3>
              <button onClick={() => setShowLoginModal(false)}><X size={24} className="text-gray-500 hover:text-red-500" /></button>
            </div>
            <form onSubmit={handleLogin} className="space-y-4">
              <input 
                type="text" 
                value={loginUser} 
                onChange={(e) => setLoginUser(e.target.value)} 
                className="w-full border rounded p-2 focus:ring-2 focus:ring-orange-500 outline-none" 
                placeholder="Usuário" 
              />
              <input 
                type="password" 
                value={loginPass} 
                onChange={(e) => setLoginPass(e.target.value)} 
                className="w-full border rounded p-2 focus:ring-2 focus:ring-orange-500 outline-none" 
                placeholder="Senha" 
              />
              {loginError && <p className="text-red-500 text-sm">{loginError}</p>}
              <button type="submit" className="w-full bg-orange-500 text-white py-2 rounded font-bold hover:bg-orange-600">Entrar</button>
            </form>
          </div>
        </div>
      )}

      {/* NAV */}
      <nav className={`bg-white shadow-md fixed w-full z-40 transition-all ${isAdmin ? 'top-16' : 'top-0'}`}>
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="flex justify-between h-24">
            <div className="flex items-center">
              {/* LOGO: Tenta carregar o arquivo, se falhar mostra texto */}
              <img 
                src="logo.png" 
                alt="Studio Dea Logo" 
                className="h-16 w-auto object-contain hidden md:block" 
                onError={(e) => {
                  e.target.style.display = 'none';
                  e.target.nextSibling.style.display = 'flex';
                }}
              />
              {/* Fallback para texto se a imagem não carregar */}
              <div className="hidden md:flex items-center" style={{display: 'none'}}>
                <span className="text-2xl font-black text-orange-500 tracking-tighter">STUDIO DEA</span>
              </div>
              <span className="md:hidden text-2xl font-black text-orange-500 tracking-tighter">STUDIO DEA</span>
            </div>
            
            <div className="hidden md:flex items-center space-x-8">
              <a href="#home" className="text-gray-600 hover:text-orange-500 font-medium">Início</a>
              <a href="#about" className="text-gray-600 hover:text-orange-500 font-medium">Sobre</a>
              <a href="#services" className="text-gray-600 hover:text-orange-500 font-medium">Serviços</a>
              <a href="#portfolio" className="text-gray-600 hover:text-orange-500 font-medium">Portfólio</a>
              <a href="#contact" className="bg-orange-500 text-white px-6 py-2 rounded-md hover:bg-orange-600 font-bold shadow-md">Fale Conosco</a>
            </div>
            <div className="md:hidden flex items-center">
              <button onClick={toggleMenu} className="text-gray-600"><Menu size={28} /></button>
            </div>
          </div>
        </div>
        {isMenuOpen && (
          <div className="md:hidden bg-white border-t p-4 space-y-2">
            <a href="#home" onClick={toggleMenu} className="block text-gray-600">Início</a>
            <a href="#about" onClick={toggleMenu} className="block text-gray-600">Sobre</a>
            <a href="#services" onClick={toggleMenu} className="block text-gray-600">Serviços</a>
            <a href="#portfolio" onClick={toggleMenu} className="block text-gray-600">Portfólio</a>
            <a href="#contact" onClick={toggleMenu} className="block text-orange-500 font-bold">Fale Conosco</a>
          </div>
        )}
      </nav>

      {/* HERO SECTION */}
      <section id="home" className={`relative h-screen flex items-center justify-center bg-gray-900 ${isAdmin ? 'mt-16' : ''}`}>
        <div className="absolute inset-0 z-0">
          <img src={isEditingText ? tempSiteData.heroImage : siteData.heroImage} alt="Background" className="w-full h-full object-cover opacity-20" />
          <div className="absolute inset-0 bg-gradient-to-b from-gray-900/80 to-gray-900/95"></div>
        </div>
        
        {isEditingText && (
          <div className="absolute top-4 right-4 z-20 bg-white p-2 rounded shadow-lg max-w-sm w-full">
            <label className="text-xs font-bold text-gray-500 flex items-center gap-1 mb-1"><ImageIcon size={12}/> Imagem de Fundo (URL)</label>
            <input 
              type="text" 
              value={tempSiteData.heroImage} 
              onChange={(e) => handleChange('heroImage', e.target.value)}
              className="w-full text-xs p-1 border rounded text-gray-800"
            />
          </div>
        )}

        <div className="relative z-10 text-center px-4 max-w-4xl w-full">
          {isEditingText ? (
            <textarea value={tempSiteData.heroTitle} onChange={(e) => handleChange('heroTitle', e.target.value)} className="w-full p-2 text-gray-900 text-3xl font-bold rounded mb-4 text-center bg-white/90" rows="2" />
          ) : (
            <h1 className="text-4xl md:text-6xl font-bold text-white mb-6 leading-tight">{siteData.heroTitle}</h1>
          )}

          {isEditingText ? (
            <textarea value={tempSiteData.heroSubtitle} onChange={(e) => handleChange('heroSubtitle', e.target.value)} className="w-full p-2 text-gray-900 text-lg rounded mb-8 text-center bg-white/90" rows="3" />
          ) : (
            <p className="text-xl text-gray-300 mb-8 max-w-2xl mx-auto">{siteData.heroSubtitle}</p>
          )}

          <div className="flex flex-col sm:flex-row gap-4 justify-center">
            {isEditingText ? (
               <input value={tempSiteData.btnHeroPrimary} onChange={(e) => handleChange('btnHeroPrimary', e.target.value)} className="p-2 rounded text-black font-bold text-center" />
            ) : (
              <a href="#contact" className="px-8 py-3 bg-orange-500 text-white font-bold rounded-md hover:bg-orange-600 transition flex items-center justify-center gap-2 shadow-lg">
                {siteData.btnHeroPrimary} <ArrowRight size={20} />
              </a>
            )}

            {isEditingText ? (
               <input value={tempSiteData.btnHeroSecondary} onChange={(e) => handleChange('btnHeroSecondary', e.target.value)} className="p-2 rounded text-black font-bold text-center" />
            ) : (
              <a href="#services" className="px-8 py-3 border-2 border-white text-white font-semibold rounded-md hover:bg-white/10 transition">
                {siteData.btnHeroSecondary}
              </a>
            )}
          </div>
        </div>
      </section>

      {/* ABOUT SECTION */}
      <section id="about" className="py-20 bg-white">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="grid md:grid-cols-2 gap-12 items-center">
            <div className="relative">
              {isEditingText && (
                <div className="absolute -top-12 left-0 z-20 bg-white p-2 rounded shadow border border-orange-200 w-full mb-2">
                  <label className="text-xs font-bold text-gray-500 flex items-center gap-1"><ImageIcon size={12}/> Imagem Sobre (URL)</label>
                  <input type="text" value={tempSiteData.aboutImage} onChange={(e) => handleChange('aboutImage', e.target.value)} className="w-full text-xs p-1 border rounded text-gray-800" />
                </div>
              )}
              <div className="absolute -top-4 -left-4 w-24 h-24 bg-orange-100 rounded-full z-0"></div>
              <img src={isEditingText ? tempSiteData.aboutImage : siteData.aboutImage} alt="Sobre" className="relative z-10 rounded-lg shadow-xl w-full h-auto object-cover" />
            </div>
            <div>
              <div className="flex items-center gap-2 mb-4">
                <div className="h-1 w-12 bg-orange-500"></div>
                {isEditingText ? (
                  <input value={tempSiteData.aboutLabel} onChange={(e) => handleChange('aboutLabel', e.target.value)} className="text-orange-500 font-bold uppercase tracking-wider text-sm border-b border-orange-300 focus:outline-none" />
                ) : (
                  <span className="text-orange-500 font-bold uppercase tracking-wider text-sm">{siteData.aboutLabel}</span>
                )}
              </div>
              
              {isEditingText ? (
                 <input value={tempSiteData.aboutTitle} onChange={(e) => handleChange('aboutTitle', e.target.value)} className="w-full text-3xl font-bold text-gray-900 mb-6 p-2 border rounded" />
              ) : (
                <h2 className="text-3xl font-bold text-gray-900 mb-6">{siteData.aboutTitle}</h2>
              )}
              
              {isEditingText ? (
                <textarea value={tempSiteData.aboutText} onChange={(e) => handleChange('aboutText', e.target.value)} className="w-full p-2 border border-gray-300 rounded mb-6 text-gray-600" rows="4" />
              ) : (
                <p className="text-lg text-gray-600 mb-6">{siteData.aboutText}</p>
              )}
              
              <div className="bg-orange-50 p-6 rounded-lg border-l-4 border-orange-500">
                {isEditingText ? (
                  <input 
                    value={tempSiteData.aboutBoxTitle} 
                    onChange={(e) => handleChange('aboutBoxTitle', e.target.value)} 
                    className="w-full font-bold text-xl mb-2 text-gray-900 bg-transparent border-b border-orange-300"
                  />
                ) : (
                  <h3 className="font-bold text-xl mb-2 text-gray-900">{siteData.aboutBoxTitle}</h3>
                )}
                
                {isEditingText ? (
                  <textarea 
                    value={tempSiteData.aboutBoxText} 
                    onChange={(e) => handleChange('aboutBoxText', e.target.value)} 
                    className="w-full text-gray-700 bg-transparent border border-orange-200 p-1 rounded"
                    rows="3"
                  />
                ) : (
                  <p className="text-gray-700">{siteData.aboutBoxText}</p>
                )}
              </div>
            </div>
          </div>
        </div>
      </section>

      {/* SERVICES */}
      <section id="services" className="py-20 bg-gray-50">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="text-center mb-16">
            <span className="text-orange-500 font-bold uppercase tracking-wider text-sm">O que fazemos</span>
            <h2 className="text-3xl font-bold text-gray-900 mt-2">Soluções Completas</h2>
          </div>
          <div className="grid md:grid-cols-2 lg:grid-cols-4 gap-8">
            {services.map((service, index) => (
              <div key={index} className="bg-white p-8 rounded-lg shadow-sm hover:shadow-xl transition duration-300 border-b-4 border-transparent hover:border-orange-500">
                <div className="mb-4">{service.icon}</div>
                <h3 className="text-xl font-bold mb-3 text-gray-900">{service.title}</h3>
                <p className="text-gray-600 text-sm leading-relaxed">{service.description}</p>
              </div>
            ))}
          </div>
        </div>
      </section>

      {/* REGULARIZATION SECTION */}
      <section className="py-16 bg-gray-900 text-white relative overflow-hidden">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 relative z-10">
          <div className="grid md:grid-cols-2 gap-8 items-center">
            <div>
              {isEditingText ? (
                <input value={tempSiteData.regTitle} onChange={(e) => handleChange('regTitle', e.target.value)} className="w-full text-3xl font-bold mb-4 text-black p-2 rounded" />
              ) : (
                <h2 className="text-3xl font-bold mb-4">{siteData.regTitle}</h2>
              )}

              {isEditingText ? (
                 <textarea value={tempSiteData.regText} onChange={(e) => handleChange('regText', e.target.value)} className="w-full text-lg text-black mb-8 p-2 rounded" rows="3" />
              ) : (
                <p className="text-lg text-gray-300 mb-8">{siteData.regText}</p>
              )}

              <ul className="space-y-3 mb-8">
                <li className="flex items-center gap-3"><CheckCircle className="text-orange-500" size={20} /><span>Aprovação de Projetos na Prefeitura</span></li>
                <li className="flex items-center gap-3"><CheckCircle className="text-orange-500" size={20} /><span>Emissão de Habite-se</span></li>
              </ul>

              {isEditingText ? (
                <div className="flex flex-col gap-2">
                  <label className="text-xs text-gray-400">Texto do Botão:</label>
                  <input value={tempSiteData.btnReg} onChange={(e) => handleChange('btnReg', e.target.value)} className="p-2 rounded text-black font-bold" />
                  <label className="text-xs text-gray-400">Link WhatsApp:</label>
                  <input value={tempSiteData.whatsappLink} onChange={(e) => handleChange('whatsappLink', e.target.value)} className="p-2 rounded text-black text-xs" />
                </div>
              ) : (
                <a href={siteData.whatsappLink} target="_blank" rel="noopener noreferrer" className="inline-block bg-orange-500 text-white px-6 py-3 rounded-md font-bold hover:bg-orange-600 transition shadow-lg">
                  {siteData.btnReg}
                </a>
              )}
            </div>
            <div className="hidden md:block">
               <div className="bg-white/5 p-8 rounded-2xl border border-white/10 text-center backdrop-blur-sm">
                 <FileCheck size={120} className="mx-auto text-orange-500 opacity-80" />
               </div>
            </div>
          </div>
        </div>
      </section>

      {/* PORTFOLIO / POSTAGENS */}
      <section id="portfolio" className="py-20 bg-white">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="flex flex-col items-center mb-12">
             <span className="text-orange-500 font-bold uppercase tracking-wider text-sm">Nosso Trabalho</span>
             <h2 className="text-3xl font-bold text-gray-900 mt-2">Projetos e Postagens</h2>
             
             {isAdmin && !showProjectForm && (
               <button onClick={() => openProjectForm()} className="mt-6 flex items-center gap-2 bg-orange-500 text-white px-4 py-2 rounded-full font-bold hover:bg-orange-600 transition shadow-md">
                 <Plus size={20} /> Adicionar Nova Postagem
               </button>
             )}

             {isAdmin && showProjectForm && (
               <div className="mt-8 bg-gray-100 p-6 rounded-lg w-full max-w-lg border-2 border-orange-500 animate-fade-in shadow-xl">
                 <h4 className="font-bold text-lg mb-4 text-gray-900">{editingProject ? "Editar Postagem" : "Nova Postagem"}</h4>
                 <div className="space-y-4">
                    <div>
                      <label className="block text-xs font-bold text-gray-500 mb-1">Título</label>
                      <input type="text" className="w-full p-2 border rounded" value={projectForm.title} onChange={(e) => setProjectForm({...projectForm, title: e.target.value})} />
                    </div>
                    <div>
                      <label className="block text-xs font-bold text-gray-500 mb-1">Categoria (ex: Obra, Dica, Interiores)</label>
                      <input type="text" className="w-full p-2 border rounded" value={projectForm.category} onChange={(e) => setProjectForm({...projectForm, category: e.target.value})} />
                    </div>
                    <div>
                      <label className="block text-xs font-bold text-gray-500 mb-1">Link da Imagem (URL)</label>
                      <input type="text" placeholder="https://..." className="w-full p-2 border rounded" value={projectForm.image} onChange={(e) => setProjectForm({...projectForm, image: e.target.value})} />
                      {projectForm.image && <img src={projectForm.image} alt="Preview" className="h-20 w-auto mt-2 rounded object-cover border" />}
                    </div>
                    <div className="flex gap-2 justify-end pt-4">
                      <button onClick={() => setShowProjectForm(false)} className="px-4 py-2 text-gray-600 hover:text-gray-900 font-medium">Cancelar</button>
                      <button onClick={handleSaveProject} className="px-6 py-2 bg-green-600 text-white rounded hover:bg-green-700 font-bold">
                        {editingProject ? "Salvar" : "Publicar"}
                      </button>
                    </div>
                 </div>
               </div>
             )}
          </div>

          <div className="grid md:grid-cols-3 gap-8">
            {projects.map((project) => (
              <div key={project.id} className="relative group bg-white rounded-lg shadow-lg overflow-hidden border border-gray-100">
                <div className="h-64 overflow-hidden relative">
                  <img src={project.image} alt={project.title} className="w-full h-full object-cover transform group-hover:scale-110 transition duration-700" />
                  <div className="absolute inset-0 bg-black/40 opacity-0 group-hover:opacity-100 transition duration-300"></div>
                </div>
                <div className="p-4">
                   <span className="text-xs text-orange-500 font-bold uppercase tracking-wider">{project.category}</span>
                   <h3 className="text-xl font-bold text-gray-900 mt-1">{project.title}</h3>
                </div>

                {isAdmin && (
                  <div className="absolute top-2 right-2 flex gap-2 z-20">
                    <button onClick={() => openProjectForm(project)} className="bg-blue-600 text-white p-2 rounded-full shadow-lg hover:bg-blue-700 transition"><Edit3 size={14} /></button>
                    <button onClick={() => handleDeleteProject(project.id)} className="bg-red-600 text-white p-2 rounded-full shadow-lg hover:bg-red-700 transition"><Trash2 size={14} /></button>
                  </div>
                )}
              </div>
            ))}
          </div>
        </div>
      </section>

      {/* FOOTER */}
      <section id="contact" className="bg-gray-800 text-gray-300 py-16 border-t-4 border-orange-500">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="grid md:grid-cols-3 gap-12">
            <div>
              <h3 className="text-2xl font-black text-white mb-6 tracking-tight">STUDIO DEA</h3>
              <p className="mb-6 text-sm text-gray-400">Engenharia, Arquitetura e Design com excelência.</p>
              <div className="space-y-4">
                <div className="flex items-start gap-3">
                  <MapPin className="text-white mt-1" size={18} />
                  {isEditingText ? <input value={tempSiteData.contactAddress} onChange={(e) => handleChange('contactAddress', e.target.value)} className="w-full text-black p-1 rounded text-xs" /> : <p className="text-sm">{siteData.contactAddress}</p>}
                </div>
                <div className="flex items-center gap-3">
                  <Phone className="text-white" size={18} />
                  {isEditingText ? <input value={tempSiteData.contactPhone} onChange={(e) => handleChange('contactPhone', e.target.value)} className="w-full text-black p-1 rounded text-xs" /> : <p>{siteData.contactPhone}</p>}
                </div>
                <div className="flex items-center gap-3">
                  <Mail className="text-white" size={18} />
                  {isEditingText ? <input value={tempSiteData.contactEmail} onChange={(e) => handleChange('contactEmail', e.target.value)} className="w-full text-black p-1 rounded text-xs" /> : <p>{siteData.contactEmail}</p>}
                </div>
              </div>
            </div>
            <div>
              <h4 className="text-lg font-bold text-white mb-6">Links Rápidos</h4>
              <ul className="space-y-2 text-sm">
                <li><a href="#home" className="hover:text-orange-500">Início</a></li>
                <li><a href="#about" className="hover:text-orange-500">Sobre Nós</a></li>
                <li><a href="#services" className="hover:text-orange-500">Regularização</a></li>
              </ul>
            </div>
            <div>
              <h4 className="text-lg font-bold text-white mb-6">Redes Sociais</h4>
              <div className="flex gap-4 mb-6">
                {isEditingText ? (
                   <input value={tempSiteData.instagramLink} onChange={(e) => handleChange('instagramLink', e.target.value)} className="w-full text-black p-1 rounded text-xs" placeholder="Link Instagram" />
                ) : (
                  <a href={siteData.instagramLink} target="_blank" rel="noreferrer" className="bg-gray-700 p-3 rounded-full hover:bg-orange-500 text-white transition"><Instagram size={24} /></a>
                )}
              </div>
              <div className="pt-6 border-t border-gray-700 flex justify-between items-center">
                 <p className="text-xs">© 2024 Studio Dea.</p>
                 {!isAdmin && <button onClick={() => setShowLoginModal(true)} className="flex items-center gap-1 text-xs text-gray-500 hover:text-orange-500"><Lock size={12}/> Admin</button>}
              </div>
            </div>
          </div>
        </div>
      </section>
    </div>
  );
};

export default App;