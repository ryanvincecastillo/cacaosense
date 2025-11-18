# 🍫 CacaoSense

**Supply Intelligence for Growth**

*Scale with confidence. Grow without breaking.*

---

## 📋 Overview

CacaoSense is an AI-powered supply chain intelligence platform designed for cacao processors in Davao, Philippines. It helps MSMEs like Cacao de Davao predict supply, optimize revenue, and scale their business with confidence.

### The Problem
Cacao processors can't produce what they can't source. Unpredictable harvests from smallholder farmers lead to:
- Production stoppages
- Missed orders and lost revenue
- Inability to commit to growth opportunities
- Strained farmer-processor relationships

### The Solution
CacaoSense provides:
- **Supply Forecasting** - Predict harvest volumes 7-30 days ahead
- **Revenue Optimization** - Recommend optimal production mix for maximum profit
- **Farmer Coordination** - Simple Messenger-based reporting for farmers
- **Growth Confidence** - Know when you can safely expand

---

## 🏗️ Project Structure

```
cacaosense/
├── README.md                    # This file
├── docs/                        # Documentation
│   ├── architecture.md          # System architecture details
│   ├── database-schema.md       # Database design
│   ├── api-reference.md         # API documentation
│   └── deployment.md            # Deployment guide
│
├── cacaosense-landing/          # Marketing & Landing Page
│   ├── index.html
│   ├── css/
│   │   └── styles.css
│   ├── js/
│   │   └── main.js
│   └── assets/
│       ├── images/
│       └── icons/
│
├── cacaosense-app/              # Main Web Application (Next.js)
│   ├── src/
│   │   ├── app/                 # Next.js App Router
│   │   ├── components/          # React components
│   │   ├── lib/                 # Utilities and helpers
│   │   ├── hooks/               # Custom React hooks
│   │   └── types/               # TypeScript types
│   ├── public/
│   ├── package.json
│   └── next.config.js
│
├── cacaosense-api/              # Backend API (FastAPI)
│   ├── app/
│   │   ├── main.py              # FastAPI entry point
│   │   ├── routers/             # API route handlers
│   │   ├── models/              # Pydantic models
│   │   ├── services/            # Business logic
│   │   ├── ml/                  # Machine learning models
│   │   └── db/                  # Database connections
│   ├── tests/
│   ├── requirements.txt
│   └── Dockerfile
│
├── cacaosense-bot/              # Messenger Chatbot
│   ├── app/
│   │   ├── main.py              # Webhook handler
│   │   ├── handlers/            # Message handlers
│   │   ├── nlp/                 # NLP processing
│   │   └── templates/           # Response templates
│   ├── requirements.txt
│   └── Dockerfile
│
├── cacaosense-ml/               # Machine Learning Pipeline
│   ├── notebooks/               # Jupyter notebooks for exploration
│   ├── models/                  # Trained model files
│   ├── src/
│   │   ├── train.py             # Model training scripts
│   │   ├── predict.py           # Prediction utilities
│   │   └── data/                # Data processing
│   └── requirements.txt
│
└── docker-compose.yml           # Local development orchestration
```

---

## 🛠️ Tech Stack

### Frontend (cacaosense-app)
- **Framework:** Next.js 14 (App Router)
- **Language:** TypeScript
- **Styling:** Tailwind CSS
- **UI Components:** shadcn/ui
- **Charts:** Recharts
- **State Management:** Zustand
- **Forms:** React Hook Form + Zod

### Backend (cacaosense-api)
- **Framework:** FastAPI (Python)
- **Database:** PostgreSQL (Supabase)
- **ORM:** SQLAlchemy
- **Authentication:** Supabase Auth
- **Task Queue:** Background tasks (FastAPI)

### Chatbot (cacaosense-bot)
- **Platform:** Facebook Messenger API
- **NLP:** OpenAI API (for natural Bisaya/English processing)
- **Webhook:** FastAPI

### Machine Learning (cacaosense-ml)
- **Forecasting:** Prophet / statsmodels
- **Optimization:** PuLP / OR-Tools
- **Framework:** scikit-learn
- **Data Processing:** pandas, numpy

### Infrastructure
- **Database:** Supabase (PostgreSQL + Auth + Storage)
- **Deployment:** Vercel (frontend), Railway (backend)
- **Weather API:** Open-Meteo

---

## 🎯 Features

### Phase 1: MVP (Hackathon)
- [ ] Farmer harvest reporting via Messenger
- [ ] Supply forecast dashboard
- [ ] Basic farmer management
- [ ] Alert system for supply dips
- [ ] Simple revenue recommendations

### Phase 2: Enhanced Intelligence
- [ ] Advanced ML forecasting models
- [ ] Revenue optimization engine
- [ ] Farmer reliability scoring
- [ ] Quality traceability
- [ ] Export readiness module

### Phase 3: Scale
- [ ] Multi-processor support (SaaS)
- [ ] Mobile app for processors
- [ ] Farmer mobile app
- [ ] API integrations
- [ ] Advanced analytics

---

## 📊 ROADMAP & TASK TRACKER

### Legend
- ⬜ Not Started
- 🟡 In Progress
- ✅ Completed
- 🔴 Blocked

---

### PHASE 1: FOUNDATION

#### 1.1 Project Setup
| Task | Status | Notes |
|------|--------|-------|
| Create project structure | ✅ | All folders initialized |
| Initialize README.md | ✅ | This file |
| Set up documentation folder | ✅ | docs/ created |
| Create architecture.md | ✅ | System architecture documented |
| Create database-schema.md | ✅ | Full schema with tables, views, functions |
| Create api-reference.md | ✅ | All endpoints documented |
| Create deployment.md | ✅ | Deployment guide for all services |

#### 1.2 Database & Backend Setup
| Task | Status | Notes |
|------|--------|-------|
| Set up Supabase project | ⬜ | |
| Design database schema | ⬜ | |
| Create database tables | ⬜ | |
| Initialize FastAPI project | ⬜ | |
| Set up SQLAlchemy models | ⬜ | |
| Create basic CRUD endpoints | ⬜ | |
| Set up authentication | ⬜ | |
| Test API endpoints | ⬜ | |

#### 1.3 Landing Page
| Task | Status | Notes |
|------|--------|-------|
| Create HTML structure | ✅ | index.html with all sections |
| Design hero section | ✅ | With dashboard preview animation |
| Create features section | ✅ | 6 feature cards with icons |
| Add pricing section | ✅ | 3 pricing tiers |
| Build contact form | ✅ | With validation and notifications |
| Mobile responsive design | ✅ | Breakpoints at 1024px, 768px, 480px |
| Add animations | ✅ | Scroll animations, hover effects |

---

### PHASE 2: FARMER INPUT SYSTEM

#### 2.1 Messenger Bot Setup
| Task | Status | Notes |
|------|--------|-------|
| Create Facebook App | ⬜ | |
| Set up Messenger webhook | ⬜ | |
| Configure page access token | ⬜ | |
| Create webhook handler | ⬜ | |
| Test basic message/response | ⬜ | |

#### 2.2 Conversation Flow
| Task | Status | Notes |
|------|--------|-------|
| Design conversation flow | ⬜ | |
| Implement farmer registration | ⬜ | |
| Create harvest report flow | ⬜ | |
| Add Bisaya language support | ⬜ | |
| Implement NLP for input parsing | ⬜ | |
| Store reports in database | ⬜ | |
| Send confirmation messages | ⬜ | |
| Add error handling | ⬜ | |

#### 2.3 Farmer Features
| Task | Status | Notes |
|------|--------|-------|
| Farmer profile management | ⬜ | |
| Report history viewing | ⬜ | |
| Weather alerts | ⬜ | |
| Price notifications | ⬜ | |

---

### PHASE 3: PROCESSOR DASHBOARD

#### 3.1 Next.js Setup
| Task | Status | Notes |
|------|--------|-------|
| Initialize Next.js project | ⬜ | |
| Configure Tailwind CSS | ⬜ | |
| Set up shadcn/ui | ⬜ | |
| Create project structure | ⬜ | |
| Set up authentication | ⬜ | |
| Create layout components | ⬜ | |

#### 3.2 Dashboard Views
| Task | Status | Notes |
|------|--------|-------|
| Dashboard home/overview | ⬜ | |
| Supply forecast view | ⬜ | |
| Farmer management view | ⬜ | |
| Orders management view | ⬜ | |
| Products management view | ⬜ | |
| Reports/analytics view | ⬜ | |
| Settings view | ⬜ | |

#### 3.3 Core Components
| Task | Status | Notes |
|------|--------|-------|
| Navigation/sidebar | ⬜ | |
| Supply forecast chart | ⬜ | |
| Farmer list table | ⬜ | |
| Farmer detail card | ⬜ | |
| Alert notification panel | ⬜ | |
| Order entry form | ⬜ | |
| Product management forms | ⬜ | |
| Revenue recommendation card | ⬜ | |

#### 3.4 Dashboard Features
| Task | Status | Notes |
|------|--------|-------|
| Real-time data updates | ⬜ | |
| Filter and search | ⬜ | |
| Export data to CSV | ⬜ | |
| Print reports | ⬜ | |
| Mobile responsive | ⬜ | |

---

### PHASE 4: AI/ML ENGINE

#### 4.1 Data Pipeline
| Task | Status | Notes |
|------|--------|-------|
| Set up data processing scripts | ⬜ | |
| Create synthetic training data | ⬜ | |
| Implement data validation | ⬜ | |
| Build feature engineering | ⬜ | |

#### 4.2 Harvest Prediction Model
| Task | Status | Notes |
|------|--------|-------|
| Explore data in notebooks | ⬜ | |
| Train Prophet model | ⬜ | |
| Evaluate model performance | ⬜ | |
| Create prediction API endpoint | ⬜ | |
| Integrate with dashboard | ⬜ | |

#### 4.3 Revenue Optimizer
| Task | Status | Notes |
|------|--------|-------|
| Define optimization constraints | ⬜ | |
| Implement optimization algorithm | ⬜ | |
| Create recommendation logic | ⬜ | |
| Build API endpoint | ⬜ | |
| Display in dashboard | ⬜ | |

#### 4.4 Farmer Scoring
| Task | Status | Notes |
|------|--------|-------|
| Define scoring criteria | ⬜ | |
| Implement scoring algorithm | ⬜ | |
| Create score history tracking | ⬜ | |
| Display scores in dashboard | ⬜ | |

---

### PHASE 5: INTEGRATION & POLISH

#### 5.1 System Integration
| Task | Status | Notes |
|------|--------|-------|
| Connect all services | ⬜ | |
| End-to-end testing | ⬜ | |
| Error handling review | ⬜ | |
| Performance optimization | ⬜ | |

#### 5.2 Demo Preparation
| Task | Status | Notes |
|------|--------|-------|
| Create demo data | ⬜ | |
| Prepare demo script | ⬜ | |
| Record backup video | ⬜ | |
| Test demo flow | ⬜ | |

#### 5.3 Pitch Materials
| Task | Status | Notes |
|------|--------|-------|
| Create pitch deck | ⬜ | |
| Prepare business model slides | ⬜ | |
| Write speaker notes | ⬜ | |
| Practice pitch timing | ⬜ | |

---

### PHASE 6: DEPLOYMENT

#### 6.1 Production Setup
| Task | Status | Notes |
|------|--------|-------|
| Set up Vercel project | ⬜ | |
| Set up Railway project | ⬜ | |
| Configure environment variables | ⬜ | |
| Set up domain (if applicable) | ⬜ | |

#### 6.2 Deployment
| Task | Status | Notes |
|------|--------|-------|
| Deploy landing page | ⬜ | |
| Deploy dashboard app | ⬜ | |
| Deploy API | ⬜ | |
| Deploy chatbot | ⬜ | |
| SSL/security check | ⬜ | |
| Final testing on production | ⬜ | |

---

## 🚀 Getting Started

### Prerequisites
- Node.js 18+
- Python 3.10+
- PostgreSQL (or Supabase account)
- Facebook Developer Account (for Messenger)

### Local Development

1. **Clone the repository**
   ```bash
   git clone https://github.com/ryanvincecastillo/cacaosense.git
   cd cacaosense
   ```

2. **Set up the backend**
   ```bash
   cd cacaosense-api
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   pip install -r requirements.txt
   cp .env.example .env
   # Edit .env with your credentials
   uvicorn app.main:app --reload
   ```

3. **Set up the frontend**
   ```bash
   cd cacaosense-app
   npm install
   cp .env.example .env.local
   # Edit .env.local with your credentials
   npm run dev
   ```

4. **Set up the chatbot**
   ```bash
   cd cacaosense-bot
   python -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   cp .env.example .env
   # Edit .env with your credentials
   uvicorn app.main:app --reload --port 8001
   ```

---

## 📁 Environment Variables

### cacaosense-api/.env
```
DATABASE_URL=postgresql://...
SUPABASE_URL=https://...
SUPABASE_KEY=...
OPENAI_API_KEY=...
```

### cacaosense-app/.env.local
```
NEXT_PUBLIC_SUPABASE_URL=https://...
NEXT_PUBLIC_SUPABASE_ANON_KEY=...
NEXT_PUBLIC_API_URL=http://localhost:8000
```

### cacaosense-bot/.env
```
VERIFY_TOKEN=...
PAGE_ACCESS_TOKEN=...
DATABASE_URL=postgresql://...
OPENAI_API_KEY=...
```

---

## 👥 Team

- **Ryan Vince Castillo** - Lead Developer
- [Add team members]

---

## 📄 License

This project is proprietary software developed for the ICT Davao AI Hackathon - MSME Edition.

---

## 🙏 Acknowledgments

- **Cacao de Davao** - Partner MSME
- **ICT Davao, Inc.** - Hackathon organizers
- **Davao cacao farming community**

---

## 📞 Contact

- Email: ryanvincecastillo@gmail.com
- GitHub: https://github.com/ryanvincecastillo

---

*Built with ❤️ for Davao's cacao industry*