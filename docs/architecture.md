# CacaoSense System Architecture

## Overview

CacaoSense is a multi-layered supply chain intelligence platform that connects smallholder cacao farmers with processors through AI-powered forecasting and optimization.

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        CACAOSENSE                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐       │
│  │   FARMER    │     │  PROCESSOR  │     │   MARKET    │       │
│  │   INPUT     │     │  DASHBOARD  │     │    DATA     │       │
│  │   LAYER     │     │   LAYER     │     │   LAYER     │       │
│  └──────┬──────┘     └──────┬──────┘     └──────┬──────┘       │
│         │                   │                   │               │
│         ▼                   ▼                   ▼               │
│  ┌─────────────────────────────────────────────────────┐       │
│  │              DATA AGGREGATION LAYER                 │       │
│  │         (API Gateway + Message Queue)               │       │
│  └──────────────────────┬──────────────────────────────┘       │
│                         │                                       │
│                         ▼                                       │
│  ┌─────────────────────────────────────────────────────┐       │
│  │                 AI/ML ENGINE                        │       │
│  │  ┌───────────┐ ┌───────────┐ ┌───────────────┐     │       │
│  │  │  Harvest  │ │  Demand   │ │   Revenue     │     │       │
│  │  │ Predictor │ │ Forecaster│ │  Optimizer    │     │       │
│  │  └───────────┘ └───────────┘ └───────────────┘     │       │
│  │  ┌───────────┐ ┌───────────┐                       │       │
│  │  │  Farmer   │ │   Waste   │                       │       │
│  │  │  Scorer   │ │ Predictor │                       │       │
│  │  └───────────┘ └───────────┘                       │       │
│  └──────────────────────┬──────────────────────────────┘       │
│                         │                                       │
│                         ▼                                       │
│  ┌─────────────────────────────────────────────────────┐       │
│  │              DECISION SUPPORT LAYER                 │       │
│  │     (Recommendations, Alerts, Insights)             │       │
│  └─────────────────────────────────────────────────────┘       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Component Details

### 1. Farmer Input Layer

**Purpose:** Collect harvest data from farmers with minimal friction

**Components:**
- Facebook Messenger Chatbot
- SMS Gateway (fallback)
- NLP Processing Engine

**Data Flow:**
1. Farmer sends message via Messenger
2. Webhook receives message
3. NLP extracts harvest information
4. Data validated and stored
5. Confirmation sent to farmer

**Key Features:**
- Bisaya/English language support
- Natural conversation flow
- Image upload for quality assessment (future)
- Voice message support (future)

---

### 2. Processor Dashboard Layer

**Purpose:** Provide actionable insights to cacao processors

**Components:**
- Next.js Web Application
- Real-time data updates via Supabase
- Interactive charts and visualizations

**Views:**
- **Dashboard Home** - Overview of key metrics
- **Supply Forecast** - 7/14/30-day predictions
- **Farmer Management** - Profiles, scores, history
- **Orders** - Order tracking and management
- **Products** - Product catalog and margins
- **Reports** - Analytics and exports
- **Settings** - Configuration and preferences

---

### 3. Market Data Layer

**Purpose:** Integrate external market intelligence

**Data Sources:**
- Weather API (Open-Meteo)
- Price inputs (manual initially)
- Seasonal patterns
- Historical trends

---

### 4. Data Aggregation Layer

**Purpose:** Central hub for all data processing

**Components:**
- FastAPI Backend
- PostgreSQL Database (Supabase)
- Background task processing

**Responsibilities:**
- API endpoint management
- Data validation
- Authentication/Authorization
- Rate limiting
- Logging and monitoring

---

### 5. AI/ML Engine

**Purpose:** Generate predictions and recommendations

**Models:**

#### Harvest Predictor
- **Input:** Farmer reports, weather, historical data
- **Output:** Volume predictions with confidence
- **Algorithm:** Prophet / LSTM

#### Demand Forecaster
- **Input:** Historical orders, seasonality
- **Output:** Expected demand by product
- **Algorithm:** XGBoost / Prophet

#### Revenue Optimizer
- **Input:** Supply, demand, prices, costs
- **Output:** Optimal production mix
- **Algorithm:** Linear Programming (PuLP)

#### Farmer Scorer
- **Input:** Delivery history, quality, communication
- **Output:** Reliability and quality scores
- **Algorithm:** Weighted scoring / ML classification

#### Waste Predictor
- **Input:** Inventory, supply, capacity
- **Output:** Waste risk alerts
- **Algorithm:** Threshold rules + regression

---

### 6. Decision Support Layer

**Purpose:** Transform ML outputs into actionable insights

**Outputs:**
- Supply forecast visualizations
- Revenue optimization recommendations
- Alert notifications
- Growth confidence scores
- Farmer development suggestions

---

## Data Flow Diagrams

### Farmer Report Flow

```
Farmer → Messenger → Webhook → NLP → Validate → Database → Dashboard
                                         ↓
                                   Confirmation → Farmer
```

### Prediction Flow

```
Database → Feature Engineering → ML Model → Prediction → Cache → API → Dashboard
              ↑
         Weather API
```

### Alert Flow

```
Prediction → Alert Rules → Notification Service → Push/Email/SMS → User
```

---

## Technology Stack

### Frontend
| Component | Technology |
|-----------|------------|
| Framework | Next.js 14 |
| Language | TypeScript |
| Styling | Tailwind CSS |
| Components | shadcn/ui |
| Charts | Recharts |
| State | Zustand |
| Forms | React Hook Form + Zod |

### Backend
| Component | Technology |
|-----------|------------|
| Framework | FastAPI |
| Language | Python 3.10+ |
| Database | PostgreSQL |
| ORM | SQLAlchemy |
| Auth | Supabase Auth |
| Tasks | FastAPI BackgroundTasks |

### AI/ML
| Component | Technology |
|-----------|------------|
| Forecasting | Prophet, statsmodels |
| Optimization | PuLP, OR-Tools |
| ML | scikit-learn |
| Data | pandas, numpy |

### Infrastructure
| Component | Technology |
|-----------|------------|
| Database | Supabase |
| Frontend Hosting | Vercel |
| Backend Hosting | Railway |
| Chatbot | Facebook Messenger API |

---

## Security Architecture

### Authentication
- Supabase Auth for processors
- Facebook authentication for farmers
- JWT tokens for API access

### Authorization
- Role-based access control (RBAC)
- Processor can only see their farmers
- API key management for integrations

### Data Protection
- HTTPS everywhere
- Database encryption at rest
- PII handling compliance
- Regular backups

---

## Scalability Considerations

### Current (MVP)
- Single processor
- Up to 50 farmers
- Basic ML models
- Manual price inputs

### Future (SaaS)
- Multi-tenant architecture
- Hundreds of processors
- Thousands of farmers
- Advanced ML pipeline
- Automated data feeds
- API integrations

---

## Integration Points

### External APIs
- **Facebook Messenger** - Farmer communication
- **Open-Meteo** - Weather data
- **Twilio/Semaphore** - SMS fallback

### Future Integrations
- Accounting software
- E-commerce platforms
- Government agricultural systems
- Banking/microfinance

---

## Monitoring & Observability

### Metrics
- API response times
- Prediction accuracy
- User engagement
- System health

### Logging
- Application logs
- Error tracking
- Audit trails

### Alerts
- System downtime
- Prediction failures
- Unusual patterns

---

## Deployment Architecture

```
                    ┌─────────────┐
                    │   Vercel    │
                    │  (Frontend) │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  Cloudflare │
                    │    (CDN)    │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
       ┌──────▼──────┐ ┌───▼────┐ ┌────▼─────┐
       │   Railway   │ │Supabase│ │ Facebook │
       │   (API)     │ │  (DB)  │ │   API    │
       └─────────────┘ └────────┘ └──────────┘
```

---

## Development Workflow

1. **Local Development** - Docker Compose for all services
2. **Testing** - Unit tests, integration tests
3. **Staging** - Preview deployments
4. **Production** - Blue-green deployments

---

## Future Architecture Enhancements

- [ ] Kubernetes for container orchestration
- [ ] Redis for caching
- [ ] Apache Kafka for event streaming
- [ ] MLflow for model management
- [ ] GraphQL API option
- [ ] Mobile apps (React Native)