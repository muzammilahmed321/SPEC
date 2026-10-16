# 🛒 DukaanMind AI

> **Give every small shop the decision-making tools of a large retailer.**

DukaanMind AI is a full-stack AI copilot for neighborhood shops (kiryana / grocery stores). It tracks sales and inventory, forecasts next-week demand, warns about expiring stock, and recommends what to reorder. Shopkeepers can also ask questions about their own business in plain language and get answers grounded in their real data.

---

## 🎯 The Problem

Small retailers usually manage stock, sales, expiry dates and supplier orders using notebooks or memory. This leads to:

- **Stockouts**: popular items run out and sales are lost.
- **Expiry waste**: products expire unsold.
- **Over-ordering**: cash gets locked in slow-moving stock.
- **No visibility**: no easy way to know what sells, what doesn't, and what to buy next.

Large retailers solve this with expensive software. Small shops have no affordable equivalent.

## 💡 The Solution

A simple web app where a shop owner records products and sales, and the system turns that data into **clear, reviewable actions**:

| Insight | Action |
|---|---|
| "Milk will sell ~42 units next week" | Suggested reorder quantity |
| "These 6 items expire within 7 days" | Expiry alert |
| "Stock for sugar covers only 2 days" | Low-stock alert |
| "Sales of eggs dropped sharply" | Anomaly alert |
| "Which products sold least this month?" | Answer from the shop's own data |

---

## ✨ Features

### Core (Full-Stack)
- 🔐 **Authentication**: secure signup / login using JWT
- 🛡️ **Authorization**: role-based access (Owner, Staff)
- 📦 **Product & inventory management**: CRUD with stock batches and expiry dates
- 💰 **Sales recording**: transactions automatically update inventory
- 🏭 **Supplier management**: supplier details and lead times
- 📊 **Dashboard**: today's sales, estimated profit, low-stock and expiring-soon alerts

### AI Features
- 📈 **Demand forecasting**: predicts next-week sales per product from sales history
- 🔁 **Smart reorder suggestions**: combines forecast, current stock and supplier lead time
- 💬 **"Ask Your Shop" assistant**: natural-language questions answered from the database using fixed, validated, read-only query tools (no free-form SQL)
- 🚨 **Anomaly detection**: flags unusual sales drops or spikes

### Roles & Permissions

| Action | Owner | Staff |
|---|:---:|:---:|
| View dashboard & stock | ✅ | ✅ |
| Record sales | ✅ | ✅ |
| Add / edit / delete products | ✅ | ❌ |
| Manage suppliers | ✅ | ❌ |
| View forecasts & reorder suggestions | ✅ | ❌ |
| Approve reorder proposals | ✅ | ❌ |
| Manage staff accounts | ✅ | ❌ |

---

## 🧠 How the AI Works

### 1. Demand Forecasting
Uses historical daily sales per product to predict the next 7 days. Starts with a moving average / simple regression (scikit-learn) and shows the **forecast, the assumptions, and a confidence range**. When a product has too little history, the app says so instead of guessing.

### 2. Reorder Recommendation
```
suggested_order = forecast_demand_over_lead_time + safety_stock - current_stock
```
Recommendations are **proposals**: the owner reviews and approves them. Nothing is ordered automatically.

### 3. Ask Your Shop (Grounded Q&A)
The LLM does **not** write SQL or invent facts. Instead it chooses from a small set of safe, read-only tools:

- `top_selling_products(period)`
- `slowest_selling_products(period)`
- `low_stock_items()`
- `expiring_items(days)`
- `sales_summary(period)`

The backend runs the tool against PostgreSQL and the LLM explains the returned data, so every answer is traceable to real records.

### 4. Anomaly Detection
Flags days where a product's sales deviate strongly from its recent average (z-score / rolling statistics).

---

## 🏗️ Architecture

```
┌──────────────┐      ┌───────────────────────┐      ┌──────────────┐
│   React UI   │ ───▶ │  Django REST Backend  │ ───▶ │  PostgreSQL  │
│ Dashboard,   │      │ Auth, validation,     │      │ Shops, sales,│
│ forms, charts│ ◀─── │ business rules,       │ ◀─── │ stock, models│
│ & AI chat    │      │ AI orchestration      │      └──────────────┘
└──────────────┘      └──────────┬────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │  AI Layer               │
                    │  • scikit-learn models  │
                    │  • LLM (safe tools)     │
                    └─────────────────────────┘
```

## 🧰 Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React, Tailwind CSS, Recharts |
| Backend | Django, Django REST Framework, SimpleJWT |
| Database | PostgreSQL |
| AI / ML | scikit-learn, pandas, LLM API |
| Deployment | Seeded demo data for a stable demo |

> The stack can be adapted to what the hackathon environment supports.

---

## 🗄️ Database Design

| Entity | Purpose |
|---|---|
| **User** | Login account with role (Owner / Staff) |
| **Shop** | Shop profile; separates one shop's data from another's |
| **Product** | Name, SKU, category, cost price, selling price, reorder threshold |
| **InventoryBatch** | Quantity, received date, expiry date per product batch |
| **Sale / SaleItem** | Transactions and quantities sold |
| **Supplier** | Contact details and lead time |
| **PurchaseOrder** | Reorder proposals and their status (proposed / approved / received) |
| **Forecast** | Predicted demand, model version, assumptions, date generated |

---

## 🔌 API Overview

| Method | Endpoint | Description | Access |
|---|---|---|---|
| POST | `/api/auth/register/` | Create account | Public |
| POST | `/api/auth/login/` | Get JWT tokens | Public |
| GET/POST | `/api/products/` | List / create products | Owner (write), Staff (read) |
| GET/POST | `/api/sales/` | List / record sales | Owner, Staff |
| GET | `/api/inventory/alerts/` | Low-stock & expiry alerts | Owner, Staff |
| GET/POST | `/api/suppliers/` | Manage suppliers | Owner |
| GET | `/api/forecast/<product_id>/` | Next-week forecast | Owner |
| GET | `/api/reorder/suggestions/` | Reorder proposals | Owner |
| POST | `/api/reorder/<id>/approve/` | Approve a proposal | Owner |
| POST | `/api/assistant/ask/` | Ask a business question | Owner |

---

## 🚀 Getting Started

### Prerequisites
- Python 3.10+
- Node.js 18+
- PostgreSQL 14+

### Backend
```bash
cd backend
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt

# configure environment
cp .env.example .env            # set DB credentials, SECRET_KEY, LLM_API_KEY

python manage.py migrate
python manage.py seed_demo_data # loads sample shop, products and sales history
python manage.py runserver
```

### Frontend
```bash
cd frontend
npm install
npm run dev
```

Open the app at `http://localhost:5173` and the API at `http://localhost:8000`.

### Demo Accounts (seeded)
| Role | Email | Password |
|---|---|---|
| Owner | owner@dukaanmind.demo | `change-me` |
| Staff | staff@dukaanmind.demo | `change-me` |

---

## 🎬 Demo Flow (90 seconds)

1. **Problem**: a shopkeeper loses money to stockouts and expired goods.
2. **Dashboard**: sales, profit, low-stock and expiry alerts.
3. **AI forecast**: next-week demand for a product, with the data behind it.
4. **Ask Your Shop**: "Which products sold least this month?" → grounded answer.
5. **Action**: review and approve a reorder suggestion.
6. **Impact**: metrics to track in a real pilot.

## 📏 Success Metrics (for a real pilot)

- Fewer stockouts per week
- Lower expiry losses
- Forecast error (MAPE) per product
- Share of reorder suggestions the owner accepts

> ⚠️ **Note:** All data in this demo is **seeded sample data**. Savings and forecast figures shown are illustrative, not measured real-world results.

---

## 🔒 Security & Responsible AI

- Passwords hashed; JWT-based authentication
- Role-based permissions enforced on the backend
- Each shop can only access its own data
- The LLM has **no direct database access**; it only calls validated, read-only tools
- Forecasts display assumptions and uncertainty
- AI recommendations require human approval

## 🗺️ Roadmap

- [x] Auth, roles, products, sales, inventory
- [x] Dashboard and alerts
- [x] Demand forecast and reorder suggestions
- [x] Ask Your Shop assistant
- [ ] Roman Urdu / Urdu question support
- [ ] Barcode scanning for faster sales entry
- [ ] WhatsApp reorder messages to suppliers
- [ ] Multi-shop support and subscription plans

## 👥 Team

| Name | Role |
|---|---|
| _Your name_ | _Full-stack / AI_ |

## 📄 License

MIT
