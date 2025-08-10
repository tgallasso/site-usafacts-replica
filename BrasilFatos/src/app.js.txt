import React, { useState } from 'react';

// --- Ícones como Componentes SVG para melhor controle e performance ---
const SearchIcon = ({ className = 'w-6 h-6' }) => (
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}>
        <circle cx="11" cy="11" r="8"></circle>
        <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
    </svg>
);

const SparkleIcon = ({ className = 'w-5 h-5' }) => (
    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="currentColor" className={className}>
        <path d="M12 2l2.35 7.16h7.65l-6.18 4.48 2.35 7.16-6.17-4.48-6.17 4.48 2.35-7.16-6.18-4.48h7.65z"/>
    </svg>
);


// --- Componente Header ---
const Header = () => {
    const [isMenuOpen, setIsMenuOpen] = useState(false);
    const navItems = ['Economy', 'Crime', 'Government spending', 'Health', 'Immigration', 'About', 'More'];

    return (
        <header className="border-b border-gray-200 py-4 bg-white">
            <div className="max-w-7xl mx-auto px-6 flex items-center justify-between">
                <div className="text-3xl font-bold text-[#002d5a]">USAFacts</div>
                <nav className="hidden lg:flex items-center gap-6">
                    <ul className="flex gap-6">
                        {navItems.map(item => (
                            <li key={item}><a href="#" className="font-semibold text-gray-800 hover:text-[#0054a6] text-sm">{item.toUpperCase()}</a></li>
                        ))}
                    </ul>
                    <button aria-label="Search" className="text-gray-600 hover:text-[#0054a6]">
                        <SearchIcon />
                    </button>
                    <a href="#" className="bg-[#0054a6] text-white font-bold text-sm py-3 px-6 rounded-md transition-colors hover:bg-[#002d5a]">SUBSCRIBE</a>
                </nav>
                <button className="lg:hidden text-3xl text-[#002d5a]" onClick={() => setIsMenuOpen(!isMenuOpen)}>
                    ☰
                </button>
            </div>
            {/* Menu Mobile */}
            {isMenuOpen && (
                 <div className="lg:hidden mt-4 px-6 pb-4">
                    <ul className="flex flex-col gap-4">
                         {navItems.map(item => (
                            <li key={item}><a href="#" className="block py-2 font-semibold text-gray-800 hover:text-[#0054a6]">{item.toUpperCase()}</a></li>
                        ))}
                    </ul>
                    <a href="#" className="bg-[#0054a6] text-white font-bold py-3 px-5 rounded-md transition-colors hover:bg-[#002d5a] mt-4 block text-center">SUBSCRIBE</a>
                </div>
            )}
        </header>
    );
};

// --- Componente Hero com Barra de Pesquisa e Gemini ---
const Hero = () => {
    const [query, setQuery] = useState('');
    const [isLoading, setIsLoading] = useState(false);
    const [geminiResponse, setGeminiResponse] = useState('');
    const [error, setError] = useState('');

    const handleGeminiQuery = async (e) => {
        e.preventDefault();
        if (!query) return;

        setIsLoading(true);
        setGeminiResponse('');
        setError('');

        const prompt = `As a data analyst for USAFacts, provide a concise, data-driven answer to the following question, based on publicly available US data. Start your answer directly, without any preamble. Question: "${query}"`;
        
        const payload = {
            contents: [{
                role: "user",
                parts: [{ text: prompt }]
            }]
        };

        try {
            // NOTE: The API key is omitted here and should be handled securely.
            const apiKey = ""; 
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;
            
            const response = await fetch(apiUrl, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });

            if (!response.ok) {
                throw new Error(`API error: ${response.statusText}`);
            }

            const result = await response.json();
            
            if (result.candidates && result.candidates.length > 0 && result.candidates[0].content.parts.length > 0) {
                const text = result.candidates[0].content.parts[0].text;
                setGeminiResponse(text);
            } else {
                throw new Error("No content received from Gemini.");
            }

        } catch (err) {
            setError(err.message);
            console.error("Error calling Gemini API:", err);
        } finally {
            setIsLoading(false);
        }
    };

    return (
        <section className="text-center bg-white py-16 lg:py-24">
            <div className="max-w-6xl mx-auto px-6">
                <h1 className="text-4xl lg:text-5xl font-bold text-[#002d5a] max-w-4xl mx-auto leading-tight">
                    You've got questions. We've got answers straight from the source.
                </h1>
                <form onSubmit={handleGeminiQuery} className="mt-10 max-w-2xl mx-auto flex flex-col sm:flex-row gap-3">
                    <input 
                        type="search" 
                        value={query}
                        onChange={(e) => setQuery(e.target.value)}
                        placeholder="Ask a question about US data..." 
                        className="flex-grow p-4 border-2 border-gray-300 rounded-md focus:ring-2 focus:ring-[#0054a6] focus:border-[#0054a6] outline-none" 
                    />
                    <button 
                        type="submit" 
                        className="bg-[#002d5a] text-white font-bold py-4 px-6 rounded-md transition-colors hover:bg-[#0054a6] flex items-center justify-center gap-2 disabled:bg-gray-400"
                        disabled={isLoading}
                    >
                        <SparkleIcon />
                        <span>{isLoading ? 'Buscando...' : 'Perguntar com IA'}</span>
                    </button>
                </form>

                {/* Área de Resposta do Gemini */}
                <div className="mt-10 max-w-3xl mx-auto text-left">
                    {isLoading && <p className="text-gray-600">Buscando dados... Por favor, aguarde.</p>}
                    {error && <div className="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded-md" role="alert">{`Ocorreu um erro: ${error}`}</div>}
                    {geminiResponse && (
                        <div className="bg-gray-50 border border-gray-200 p-6 rounded-lg shadow-md animate-fade-in">
                            <h3 className="text-xl font-bold text-[#002d5a] mb-3 flex items-center gap-2">
                                <SparkleIcon className="text-[#0054a6]" />
                                Resposta Rápida
                            </h3>
                            <p className="text-gray-800 whitespace-pre-wrap">{geminiResponse}</p>
                             <button onClick={() => setGeminiResponse('')} className="text-sm text-gray-500 hover:text-black mt-4">Limpar resposta</button>
                        </div>
                    )}
                </div>
            </div>
        </section>
    );
};


// --- Componente de Conteúdo em Destaque ---
const FeaturedContent = () => (
    <section className="bg-white pb-16 lg:pb-24">
        <div className="max-w-7xl mx-auto px-6 grid lg:grid-cols-3 gap-8">
            {/* Coluna Principal (2/3) */}
            <div className="lg:col-span-2 group cursor-pointer">
                <div className="overflow-hidden rounded-lg">
                    <img 
                        src="https://placehold.co/800x450/002d5a/ffffff?text=Featured+Image" 
                        alt="What's in the One Big Beautiful Bill?" 
                        className="w-full h-full object-cover transition-transform duration-300 group-hover:scale-105"
                    />
                </div>
                <p className="text-[#0054a6] font-bold mt-4 text-sm">GUIDE</p>
                <h2 className="text-3xl font-bold text-[#002d5a] mt-2 group-hover:underline">What's in the One Big Beautiful Bill?</h2>
                <p className="text-gray-600 mt-2">The US government is projected to spend $6.9 trillion in fiscal year 2024. This guide explains how that money is spent.</p>
            </div>
            {/* Coluna Lateral (1/3) */}
            <div className="group cursor-pointer">
                 <div className="overflow-hidden rounded-lg">
                    <img 
                        src="https://placehold.co/400x225/e9ecef/343a40?text=Steve+Ballmer" 
                        alt="Just the Facts with Steve Ballmer" 
                        className="w-full h-full object-cover transition-transform duration-300 group-hover:scale-105"
                    />
                </div>
                <h3 className="text-xl font-bold text-[#002d5a] mt-4 group-hover:underline">Just the Facts with Steve Ballmer</h3>
                <p className="text-gray-600 mt-2">In our video series, USAFacts Founder Steve Ballmer shares the numbers behind the news.</p>
            </div>
        </div>
    </section>
);

// --- Componente de Perguntas em Alta ---
const TrendingQuestions = () => {
    const questions = [
        "How much does the US trade with other countries?",
        "How active is the 2025 hurricane season?",
        "How many people die from gun-related deaths per month?",
        "What is the current unemployment rate?"
    ];
    return (
        <section className="bg-gray-50 py-16 lg:py-20">
            <div className="max-w-7xl mx-auto px-6">
                <h2 className="text-3xl font-bold text-[#002d5a] mb-6">Trending and recently asked questions</h2>
                <div className="grid sm:grid-cols-2 gap-x-8 gap-y-4">
                    {questions.map(q => (
                        <a key={q} href="#" className="text-lg text-gray-800 hover:text-[#0054a6] hover:underline">{q}</a>
                    ))}
                </div>
            </div>
        </section>
    );
};

// --- Componente de Card para Relatórios ---
const ReportCard = ({ imageUrl, tag, title }) => (
    <div className="group cursor-pointer">
        <div className="overflow-hidden rounded-lg">
            <img src={imageUrl} alt={title} className="w-full h-auto object-cover transition-transform duration-300 group-hover:scale-105" />
        </div>
        <p className="text-[#0054a6] font-bold mt-4 text-sm">{tag}</p>
        <h3 className="text-xl font-bold text-[#002d5a] mt-1 group-hover:underline">{title}</h3>
    </div>
);

// --- Componente para a Seção "Sobre Nós" ---
const AboutUs = () => (
    <section className="bg-gray-50 text-center py-16 lg:py-20">
        <div className="max-w-6xl mx-auto px-6">
            <h2 className="text-3xl lg:text-4xl font-bold text-[#002d5a] mb-4">Learn more about us</h2>
            <p className="max-w-3xl mx-auto text-lg text-gray-700 mb-8">
                USAFacts is a non-partisan, not-for-profit civic initiative providing a data-driven portrait of the American population, our government’s finances, and government’s impact on society.
            </p>
            <a href="#" className="bg-transparent text-[#002d5a] font-bold py-3 px-7 rounded-md border-2 border-[#002d5a] transition-all hover:bg-[#002d5a] hover:text-white">
                About USAFacts
            </a>
        </div>
    </section>
);

// --- Componente para a Seção "Na Mídia" ---
const InTheNews = () => {
    const logos = ['Fast Company', 'Axios', 'Fox News', 'The New York Times', 'Forbes'];
    return (
        <section className="bg-white py-16 lg:py-20">
            <div className="max-w-6xl mx-auto px-6">
                <h2 className="text-3xl lg:text-4xl font-bold text-[#002d5a] text-center mb-12">In the News</h2>
                <div className="flex flex-wrap justify-center items-center gap-x-12 gap-y-8 text-gray-500">
                    {logos.map(logo => (
                        <span key={logo} className="text-2xl font-semibold">{logo}</span>
                    ))}
                </div>
            </div>
        </section>
    );
};

// --- Componente Footer ---
const Footer = () => {
    const footerLinks = {
        Browse: ['Crime', 'Defense & Security', 'Economy', 'Education', 'Environment', 'Health', 'Immigration', 'Population'],
        More: ['Guides & reports', 'Editorial guidelines', 'Data sources', 'FAQs', 'Contact us'],
        About: ['Our mission', 'Leadership', 'Careers', 'Press'],
    };

    return (
        <footer className="bg-[#002d5a] text-gray-300 pt-16 pb-8">
            <div className="max-w-7xl mx-auto px-6">
                <div className="grid lg:grid-cols-2 gap-12 mb-12">
                    <div>
                        <h3 className="text-white font-bold text-xl mb-4">Subscribe to our newsletter</h3>
                        <form className="flex flex-col sm:flex-row gap-3">
                            <input type="email" placeholder="Enter your email" className="flex-grow p-3 rounded-md border-0 text-gray-800" />
                            <button type="submit" className="bg-[#0054a6] text-white font-bold py-3 px-6 rounded-md hover:bg-blue-700">Sign Up</button>
                        </form>
                    </div>
                    <div className="grid grid-cols-2 sm:grid-cols-3 gap-8">
                        {Object.entries(footerLinks).map(([title, links]) => (
                            <div key={title}>
                                <h4 className="font-bold text-white mb-4">{title}</h4>
                                <ul>{links.map(link => <li key={link} className="mb-2"><a href="#" className="hover:text-white text-sm">{link}</a></li>)}</ul>
                            </div>
                        ))}
                    </div>
                </div>
                <div className="border-t border-gray-700 pt-8 flex flex-col sm:flex-row justify-between items-center gap-4 text-sm">
                    <p>© 2024 USAFacts. All rights reserved.</p>
                    <ul className="flex gap-6">
                        <li><a href="#" className="hover:text-white">Terms of service</a></li>
                        <li><a href="#" className="hover:text-white">Privacy policy</a></li>
                    </ul>
                </div>
            </div>
        </footer>
    );
};


// --- Componente Principal da Aplicação ---
export default function App() {
    const latestReportsData = [
        { imageUrl: "https://placehold.co/400x225/0054a6/ffffff?text=Report+1", tag: "INTERACTIVE", title: "How do taxes fund the federal government?" },
        { imageUrl: "https://placehold.co/400x225/343a40/ffffff?text=Report+2", tag: "REPORT", title: "2025 State of the Union in Numbers" },
        { imageUrl: "https://placehold.co/400x225/6c757d/ffffff?text=Report+3", tag: "REPORT", title: "2024 Government 10-K" },
    ];

    return (
        <div className="bg-white">
            <Header />
            <main>
                <Hero />
                <FeaturedContent />
                <TrendingQuestions />
                <section className="bg-white py-16 lg:py-24">
                    <div className="max-w-7xl mx-auto px-6">
                        <h2 className="text-3xl font-bold text-[#002d5a] mb-8">Latest projects</h2>
                        <div className="grid sm:grid-cols-2 lg:grid-cols-3 gap-8">
                            {latestReportsData.map((item, index) => <ReportCard key={index} {...item} />)}
                        </div>
                    </div>
                </section>
                <AboutUs />
                <InTheNews />
            </main>
            <Footer />
        </div>
    );
}
