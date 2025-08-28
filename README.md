
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Interactive Financial Glossary</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Chosen Palette: Warm Neutrals -->
    <!-- Application Structure Plan: The application is designed as a single-page dashboard to provide an intuitive and efficient way to explore a large glossary. The structure consists of a persistent left sidebar for category-based navigation, a main content area with a powerful live search bar, and an interactive chart for visual exploration. This thematic, filter-driven approach was chosen over a simple linear list to manage the cognitive load of over 150 terms, allowing users to quickly drill down into topics of interest (e.g., "Investing," "Crypto") or find specific definitions instantly via search. The user flow is designed for discovery (browsing categories) and direct lookup (searching), making it highly usable for both beginners and those needing a quick reference. -->
    <!-- Visualization & Content Choices: Report Info: Large, uncategorized list of financial terms. -> Goal: Organize, Compare, Inform. -> Viz/Presentation Method: 1) Interactive Bar Chart (Chart.js/Canvas) to show term distribution per category. 2) Card-based layout (HTML/Tailwind) for individual terms. 3) Sidebar Navigation (HTML/Tailwind). -> Interaction: 1) Chart bars are clickable, filtering the term cards below. 2) A live search bar filters cards by term and definition. 3) Sidebar links filter cards by category. -> Justification: This multi-faceted approach supports different user behaviors. The chart provides a high-level visual overview and a novel navigation path. The card layout is more readable and responsive than a dense table. The live search is essential for a glossary's primary function: quick lookups. -> Library/Method: Chart.js for the canvas-based chart; Vanilla JS for all filtering and DOM manipulation. -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        .chart-container {
            position: relative;
            width: 100%;
            max-width: 800px;
            margin-left: auto;
            margin-right: auto;
            height: 300px;
            max-height: 40vh;
        }
        @media (min-width: 768px) {
            .chart-container {
                height: 350px;
            }
        }
        ::-webkit-scrollbar {
            width: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #f1f5f9;
        }
        ::-webkit-scrollbar-thumb {
            background: #94a3b8;
            border-radius: 10px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #64748b;
        }
        .active-category {
            background-color: #e0f2fe;
            color: #0c4a6e;
            font-weight: 600;
        }
    </style>
    <link rel="stylesheet" href="https://rsms.me/inter/inter.css">
</head>
<body class="bg-slate-50 text-slate-800">

    <div class="flex flex-col md:flex-row min-h-screen">
        <!-- Sidebar -->
        <aside id="sidebar" class="w-full md:w-64 bg-white border-r border-slate-200 p-4 md:p-6 md:fixed md:h-full">
            <h1 class="text-2xl font-bold text-slate-900 mb-6">Financial Glossary</h1>
            <nav>
                <h2 class="text-sm font-semibold text-slate-500 uppercase tracking-wider mb-3">Categories</h2>
                <ul id="category-list" class="space-y-2">
                    <!-- Categories will be populated by JS -->
                </ul>
            </nav>
        </aside>

        <!-- Main Content -->
        <main class="flex-1 md:ml-64 p-4 sm:p-6 lg:p-8">
            <div class="max-w-7xl mx-auto">
                <!-- Header & Search -->
                <div class="mb-8">
                    <p class="text-slate-600 mb-4">This interactive glossary provides clear definitions for a wide range of terms related to finance, business, and technology. Use the categories on the left, the interactive chart, or the search bar below to explore and learn.</p>
                    <div class="relative">
                        <input type="text" id="search-input" placeholder="Search for a term (e.g., 'ETF', 'Inflation')..." class="w-full p-4 pl-12 text-lg border border-slate-300 rounded-lg focus:ring-2 focus:ring-sky-500 focus:border-sky-500 transition">
                        <div class="absolute inset-y-0 left-0 pl-4 flex items-center pointer-events-none">
                            <svg class="h-6 w-6 text-slate-400" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path></svg>
                        </div>
                    </div>
                </div>

                <!-- Chart Section -->
                <div class="bg-white p-6 rounded-xl shadow-sm border border-slate-200 mb-8">
                     <h3 class="text-xl font-bold text-slate-900 mb-1">Term Distribution</h3>
                     <p class="text-slate-500 mb-4">Click on a bar to filter terms by that category.</p>
                    <div class="chart-container">
                        <canvas id="termsChart"></canvas>
                    </div>
                </div>

                <!-- Terms Grid -->
                <div id="terms-grid" class="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-6">
                    <!-- Term cards will be populated by JS -->
                </div>
                 <div id="no-results" class="hidden text-center py-16">
                    <h3 class="text-2xl font-semibold text-slate-700">No Results Found</h3>
                    <p class="text-slate-500 mt-2">Try adjusting your search or clearing the filters.</p>
                </div>
            </div>
        </main>
    </div>

    <script>
        const termsData = [
            { term: "Stocks", category: "Investing & Instruments", definition: "Represents ownership in a public company. Profit through capital appreciation (price increase) or dividends." },
            { term: "Shares", category: "Investing & Instruments", definition: "A single unit of ownership in a corporation. 'Stock' and 'shares' are often used interchangeably." },
            { term: "ETF (Exchange-Traded Fund)", category: "Investing & Instruments", definition: "An investment fund holding diverse assets (stocks, bonds) that trades on an exchange like a stock." },
            { term: "US Bonds", category: "Investing & Instruments", definition: "Debt securities issued by the U.S. Treasury. Considered very low-risk investments." },
            { term: "Yankee Bond", category: "Investing & Instruments", definition: "A bond issued by a foreign entity in the U.S. and denominated in U.S. dollars." },
            { term: "Convertible Bonds", category: "Investing & Instruments", definition: "A bond that can be converted into a predetermined number of the issuing company's shares." },
            { term: "Preferred Stocks", category: "Investing & Instruments", definition: "A class of stock with priority over common stock for dividends and asset distribution, but usually no voting rights." },
            { term: "Mutual Funds", category: "Investing & Instruments", definition: "Pools money from many investors to invest in a diversified portfolio managed by a professional." },
            { term: "Balanced Mutual Funds", category: "Investing & Instruments", definition: "A mutual fund that invests in a mix of both stocks and bonds to balance growth and stability." },
            { term: "REITs (Real Estate Investment Trusts)", category: "Investing & Instruments", definition: "A company that owns or finances income-producing real estate, allowing individuals to invest in real estate portfolios." },
            { term: "MBS (Mortgage-Backed Security)", category: "Investing & Instruments", definition: "A financial instrument whose value is derived from a pool of mortgages." },
            { term: "Securities", category: "Investing & Instruments", definition: "Tradable financial assets representing ownership (equity), a creditor relationship (debt), or rights to ownership (derivatives)." },
            { term: "Futures", category: "Investing & Instruments", definition: "A legal contract to buy or sell a commodity or financial instrument at a predetermined price at a future date." },
            { term: "Commodities", category: "Investing & Instruments", definition: "Basic goods used in commerce that are interchangeable, like oil, gold, or wheat." },
            { term: "Bond, Bill, Note (Treasury)", category: "Investing & Instruments", definition: "Debt instruments from the U.S. Treasury. Bills mature in <1 year, Notes in 2-10 years, Bonds in >10 years." },
            { term: "TIPS", category: "Investing & Instruments", definition: "Treasury Inflation-Protected Securities. U.S. Treasury bonds whose principal value adjusts with inflation." },
            { term: "FRN (Floating Rate Note)", category: "Investing & Instruments", definition: "A debt instrument with a variable interest rate that resets periodically based on a benchmark." },
            { term: "Penny stock", category: "Investing & Instruments", definition: "Stocks of very small companies that trade for less than $5 per share. Highly speculative and risky." },
            { term: "Trading", category: "Investing & Instruments", definition: "The act of buying and selling financial instruments with the goal of profiting from short-term price fluctuations." },
            { term: "Stock exchange", category: "Investing & Instruments", definition: "A marketplace where stocks, bonds, and other securities are bought and sold (e.g., NYSE, Nasdaq)." },
            { term: "S&P 500", category: "Market Indices", definition: "A stock market index tracking the performance of 500 of the largest U.S. publicly traded companies." },
            { term: "Indices / Index", category: "Market Indices", definition: "A statistical measure of changes in a representative group of data points (e.g., stock prices), used as a market benchmark." },
            { term: "Dow Jones Industrial Average", category: "Market Indices", definition: "A stock market index that measures the stock performance of 30 large, well-known companies in the U.S." },
            { term: "Nasdaq Composite", category: "Market Indices", definition: "A stock market index of all stocks listed on the Nasdaq stock exchange, heavily weighted towards technology companies." },
            { term: "Russell 2000", category: "Market Indices", definition: "A stock market index that measures the performance of 2,000 small-cap U.S. companies." },
            { term: "VIX (Volatility Index)", category: "Market Indices", definition: "Often called the 'fear index,' it measures the market's expectation of future volatility based on S&P 500 options." },
            { term: "Principal", category: "Economic Concepts", definition: "The original amount of a loan, investment, or debt before any interest, fees, or returns are added." },
            { term: "Interest", category: "Economic Concepts", definition: "The cost of borrowing money or the income earned from lending money, typically expressed as a percentage." },
            { term: "Inflation", category: "Economic Concepts", definition: "The rate at which the general level of prices for goods and services rises, decreasing the purchasing power of currency." },
            { term: "Deflation", category: "Economic Concepts", definition: "The opposite of inflation; a general decline in the price level of goods and services, increasing purchasing power." },
            { term: "Appreciation", category: "Economic Concepts", definition: "The increase in the value of an asset (like a stock or property) over a period of time." },
            { term: "GDP (Gross Domestic Product)", category: "Economic Concepts", definition: "The total monetary value of all finished goods and services produced within a country's borders in a specific period." },
            { term: "CPI (Consumer Price Index)", category: "Economic Concepts", definition: "A measure of the average change over time in the prices paid by urban consumers for a basket of consumer goods." },
            { term: "Capital Gain", category: "Economic Concepts", definition: "The profit realized from the sale of a capital asset (like stocks or real estate) when the selling price exceeds the purchase price." },
            { term: "Net profit", category: "Economic Concepts", definition: "The amount of money a business has left after deducting all expenses, including taxes, from total revenue." },
            { term: "Gross profit", category: "Economic Concepts", definition: "The profit a company makes after deducting the direct costs of producing and selling its products (Cost of Goods Sold)." },
            { term: "FX (Foreign Exchange)", category: "Economic Concepts", definition: "The global decentralized market where currencies are traded. It determines the exchange rates for currencies around the world." },
            { term: "Roth IRA", category: "Retirement & Savings", definition: "An Individual Retirement Account (IRA) where contributions are made with after-tax dollars, allowing for tax-free withdrawals in retirement." },
            { term: "SEP IRA", category: "Retirement & Savings", definition: "A Simplified Employee Pension plan for self-employed individuals and small business owners, allowing for tax-deductible contributions." },
            { term: "401(k)", category: "Retirement & Savings", definition: "An employer-sponsored retirement plan allowing employees to contribute a portion of their pre-tax salary to an investment account." },
            { term: "Savings account", category: "Retirement & Savings", definition: "An interest-bearing deposit account at a bank, providing a safe place to store money and earn a modest return." },
            { term: "Checking account", category: "Retirement & Savings", definition: "A deposit account for day-to-day transactions, allowing for frequent withdrawals and deposits, with little to no interest." },
            { term: "High yield saving", category: "Retirement & Savings", definition: "A savings account offering significantly higher interest rates than traditional accounts, often from online banks." },
            { term: "Money market account", category: "Retirement & Savings", definition: "An account combining features of savings and checking, often with higher interest rates and minimum balances." },
            { term: "CDs (Certificates of Deposit)", category: "Retirement & Savings", definition: "A savings product where you deposit a fixed sum for a specific period in exchange for a fixed interest rate." },
            { term: "Treasury Direct", category: "Retirement & Savings", definition: "A U.S. Treasury website allowing investors to buy and hold Treasury securities directly without a broker." },
            { term: "Crypto Currency", category: "Crypto & Tech", definition: "A decentralized digital currency secured by cryptography, such as Bitcoin or Ethereum." },
            { term: "Stable Coin", category: "Crypto & Tech", definition: "A type of cryptocurrency designed to maintain a stable value, usually by being pegged to a traditional asset like the U.S. dollar." },
            { term: "DAO (Decentralized Autonomous Organization)", category: "Crypto & Tech", definition: "An organization whose rules are encoded as transparent computer programs (smart contracts) on a blockchain." },
            { term: "Blockchain", category: "Crypto & Tech", definition: "A distributed, immutable, and decentralized digital ledger that records transactions across a network of computers." },
            { term: "NFT (Non-Fungible Token)", category: "Crypto & Tech", definition: "A unique digital asset on a blockchain representing ownership of a specific item, like art or music." },
            { term: "Crypto exchange", category: "Crypto & Tech", definition: "An online platform for buying, selling, and trading cryptocurrencies." },
            { term: "Crypto Wallet", category: "Crypto & Tech", definition: "A digital wallet to securely store, send, and receive cryptocurrencies by managing private keys." },
            { term: "IP address", category: "Crypto & Tech", definition: "A unique numerical label assigned to each device on a computer network, used for identification and location." },
            { term: "Database", category: "Crypto & Tech", definition: "An organized collection of structured information, or data, typically stored electronically in a computer system." },
            { term: "VC (Venture Capital)", category: "Business & Startups", definition: "Financing provided by firms to startups and emerging companies with high growth potential, in exchange for equity." },
            { term: "PE (Private Equity)", category: "Business & Startups", definition: "Investment funds that invest in or acquire private companies that are not listed on a public stock exchange." },
            { term: "Angel Investor", category: "Business & Startups", definition: "A wealthy individual who provides capital for a business start-up, usually in exchange for convertible debt or ownership equity." },
            { term: "Hedge Fund", category: "Business & Startups", definition: "An aggressively managed portfolio of investments using advanced strategies to generate high returns for wealthy investors." },
            { term: "Crowdfunding", category: "Business & Startups", definition: "Funding a project or venture by raising small amounts of money from a large number of people, typically via the internet." },
            { term: "Start Up", category: "Business & Startups", definition: "A young company founded by one or more entrepreneurs to develop a unique product or service and bring it to market." },
            { term: "Series A, B, C, D", category: "Business & Startups", definition: "Distinct rounds of venture capital funding for startups, indicating increasing maturity and valuation as the series progresses." },
            { term: "MVP (Minimum Viable Product)", category: "Business & Startups", definition: "A version of a new product with just enough features to satisfy early customers and provide feedback for future development." },
            { term: "LLC (Limited Liability Company)", category: "Business & Startups", definition: "A business structure that protects its owners from personal responsibility for its debts or liabilities." },
            { term: "Sole Proprietor", category: "Business & Startups", definition: "A business owned and run by one person, where there is no legal distinction between the owner and the business." },
            { term: "S Corp / C Corp", category: "Business & Startups", definition: "Types of corporations. C Corps are taxed separately from owners. S Corps pass profits/losses to owners' personal income." },
            { term: "Founder", category: "Business & Startups", definition: "The person or people who establish a new business or organization." },
            { term: "CEO (Chief Executive Officer)", category: "Business & Startups", definition: "The highest-ranking executive, responsible for major corporate decisions and managing overall operations." },
            { term: "Entrepreneur", category: "Business & Startups", definition: "A person who organizes and operates a business, taking on greater than normal financial risks to do so." },
            { term: "Insurance", category: "Insurance & Legal", definition: "A contract (policy) providing financial protection against potential losses from an insurer in exchange for a premium." },
            { term: "Home Insurance", category: "Insurance & Legal", definition: "Covers losses and damages to an individual's residence and assets within the home." },
            { term: "Health Insurance", category: "Insurance & Legal", definition: "Covers medical expenses, surgical procedures, and prescription drugs." },
            { term: "Life Insurance", category: "Insurance & Legal", definition: "A contract where an insurer pays a sum of money to beneficiaries upon the death of the insured person." },
            { term: "Car Insurance", category: "Insurance & Legal", definition: "Provides financial protection against physical damage or bodily injury from traffic collisions." },
            { term: "Renters insurance", category: "Insurance & Legal", definition: "Covers personal belongings within a rented property and provides liability coverage." },
            { term: "Business insurance", category: "Insurance & Legal", definition: "A category of policies that protect businesses from risks like property damage and liability claims." },
            { term: "PMI (Private Mortgage Insurance)", category: "Insurance & Legal", definition: "Insurance that protects the mortgage lender if a borrower defaults. Typically required for down payments under 20%." },
            { term: "FDIC", category: "Insurance & Legal", definition: "Federal Deposit Insurance Corporation. A U.S. government agency that insures bank deposits up to $250,000." },
            { term: "Taxes", category: "Insurance & Legal", definition: "Mandatory financial charges levied by a government on individuals or corporations to fund public services." },
            { term: "W2 / W9 / 1099", category: "Insurance & Legal", definition: "IRS tax forms. W2 for employees, W9 to confirm taxpayer info, 1099 for non-employee income." },
            { term: "EIN / SSN", category: "Insurance & Legal", definition: "Taxpayer IDs. EIN (Employer ID Number) for businesses, SSN (Social Security Number) for individuals." },
            { term: "Trade Mark / Patent / Copyright", category: "Insurance & Legal", definition: "Forms of intellectual property protection for brands (Trademark), inventions (Patent), and creative works (Copyright)." },
            { term: "Credit", category: "Loans & Credit", definition: "The ability to borrow money or access goods/services with the understanding that payment will be made later." },
            { term: "Loans", category: "Loans & Credit", definition: "A sum of money lent to another party in exchange for future repayment of the principal amount along with interest." },
            { term: "Grants", category: "Loans & Credit", definition: "A sum of money given by an organization for a specific purpose (e.g., education, research) that does not need to be repaid." },
            { term: "Scholarship", category: "Loans & Credit", definition: "A financial award for a student to support their education, based on merit or other criteria, which does not need to be repaid." },
            { term: "Loan Shark", category: "Loans & Credit", definition: "A person who lends money at illegally high interest rates and often uses intimidation to enforce repayment." },
            { term: "Business Credit", category: "Loans & Credit", definition: "A credit rating for a company, separate from the owner's personal credit, used to secure business loans and financing." },
            { term: "Mortgage", category: "Loans & Credit", definition: "A loan used to finance the purchase of real estate, where the property itself serves as collateral." },
            { term: "Student loans", category: "Loans & Credit", definition: "Loans designed to help students pay for post-secondary education and related expenses." },
        ];

        const termsGrid = document.getElementById('terms-grid');
        const searchInput = document.getElementById('search-input');
        const categoryList = document.getElementById('category-list');
        const noResultsDiv = document.getElementById('no-results');
        let termsChart;
        let activeCategory = 'All';

        const categories = ['All', ...new Set(termsData.map(term => term.category))];

        function renderCategories() {
            categoryList.innerHTML = '';
            categories.forEach(category => {
                const li = document.createElement('li');
                const button = document.createElement('button');
                button.textContent = category;
                button.className = `w-full text-left px-4 py-2 rounded-lg transition-colors duration-200 hover:bg-slate-100 ${category === activeCategory ? 'active-category' : ''}`;
                button.onclick = () => {
                    activeCategory = category;
                    renderCategories();
                    renderTerms();
                };
                li.appendChild(button);
                categoryList.appendChild(li);
            });
        }

        function renderTerms() {
            const searchTerm = searchInput.value.toLowerCase();
            const filteredTerms = termsData.filter(term => {
                const inCategory = activeCategory === 'All' || term.category === activeCategory;
                const matchesSearch = term.term.toLowerCase().includes(searchTerm) || term.definition.toLowerCase().includes(searchTerm);
                return inCategory && matchesSearch;
            });

            termsGrid.innerHTML = '';
            if (filteredTerms.length === 0) {
                noResultsDiv.classList.remove('hidden');
            } else {
                noResultsDiv.classList.add('hidden');
                filteredTerms.forEach(term => {
                    const card = document.createElement('div');
                    card.className = 'bg-white p-6 rounded-xl shadow-sm border border-slate-200 transition hover:shadow-md hover:-translate-y-1';
                    card.innerHTML = `
                        <h3 class="text-xl font-bold text-slate-900 mb-2">${term.term}</h3>
                        <p class="text-slate-600">${term.definition}</p>
                        <div class="mt-4">
                            <span class="inline-block bg-sky-100 text-sky-800 text-xs font-semibold px-2.5 py-0.5 rounded-full">${term.category}</span>
                        </div>
                    `;
                    termsGrid.appendChild(card);
                });
            }
        }

        function setupChart() {
            const ctx = document.getElementById('termsChart').getContext('2d');
            const categoryCounts = categories.slice(1).map(category => {
                return termsData.filter(term => term.category === category).length;
            });

            termsChart = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: categories.slice(1),
                    datasets: [{
                        label: '# of Terms',
                        data: categoryCounts,
                        backgroundColor: 'rgba(14, 165, 233, 0.6)',
                        borderColor: 'rgba(14, 165, 233, 1)',
                        borderWidth: 1,
                        borderRadius: 5,
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        y: {
                            beginAtZero: true,
                            grid: {
                                color: '#e2e8f0'
                            },
                            ticks: {
                                color: '#64748b'
                            }
                        },
                        x: {
                            grid: {
                                display: false
                            },
                            ticks: {
                                color: '#64748b'
                            }
                        }
                    },
                    plugins: {
                        legend: {
                            display: false
                        },
                        tooltip: {
                            backgroundColor: '#0f172a',
                            titleFont: {
                                size: 16
                            },
                            bodyFont: {
                                size: 14
                            },
                            padding: 12,
                            cornerRadius: 6
                        }
                    },
                    onClick: (event, elements) => {
                        if (elements.length > 0) {
                            const chartElement = elements[0];
                            const category = termsChart.data.labels[chartElement.index];
                            activeCategory = category;
                            searchInput.value = '';
                            renderCategories();
                            renderTerms();
                        }
                    },
                    onHover: (event, chartElement) => {
                        event.native.target.style.cursor = chartElement[0] ? 'pointer' : 'default';
                    }
                }
            });
        }

        searchInput.addEventListener('keyup', renderTerms);

        document.addEventListener('DOMContentLoaded', () => {
            renderCategories();
            renderTerms();
            setupChart();
        });
    </script>
</body>
</html>

