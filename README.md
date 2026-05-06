# 🌌 Weaver AI

> A full-stack AI-powered chat and image generation web application with a credit-based system and Stripe payments.

![Tech Stack](https://img.shields.io/badge/React-19-61DAFB?style=flat&logo=react)
![Node.js](https://img.shields.io/badge/Node.js-Express-339933?style=flat&logo=node.js)
![MongoDB](https://img.shields.io/badge/MongoDB-Mongoose-47A248?style=flat&logo=mongodb)
![Stripe](https://img.shields.io/badge/Payments-Stripe-635BFF?style=flat&logo=stripe)

---

## 📖 Table of Contents

- [About the Project](#about-the-project)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Database Schema](#database-schema)
- [API Endpoints](#api-endpoints)
- [Environment Variables](#environment-variables)
- [Getting Started](#getting-started)
- [How It Works](#how-it-works)
- [Deployment](#deployment)

---

## About the Project

Weaver AI is a full-stack web app where users can:
- Chat with **Google Gemini AI** for text responses
- Generate **AI images** from text prompts via ImageKit
- Share generated images to a **community gallery**
- Buy more **credits** via Stripe to keep using the app

Every new user gets **20 free credits**. Text generation costs **1 credit**, image generation costs **2 credits**.

---

## Features

- 🔐 **JWT Authentication** — Register & Login with hashed passwords
- 💬 **AI Text Chat** — Powered by Google Gemini 2.5 Flash
- 🖼️ **AI Image Generation** — Powered by ImageKit AI generation
- 🏛️ **Community Gallery** — Users can publish their AI images
- 💎 **Credit System** — Pay-as-you-go with Stripe Checkout
- 🌙 **Dark/Light Mode** — Theme persisted in localStorage
- 📱 **Responsive Design** — Mobile-friendly with collapsible sidebar
- 💳 **Stripe Webhooks** — Credits added only after payment confirmation

---

## Tech Stack

### Frontend
| Technology | Purpose |
|---|---|
| React 19 | UI Framework |
| React Router v7 | Client-side routing |
| Context API | Global state management |
| Axios | HTTP requests |
| Tailwind CSS v4 | Styling |
| react-markdown | Render AI markdown responses |
| Prism.js | Code syntax highlighting |
| react-hot-toast | Notifications |
| moment.js | Timestamp formatting |
| Vite | Build tool |

### Backend
| Technology | Purpose |
|---|---|
| Node.js + Express 5 | Server framework |
| Mongoose | MongoDB ODM |
| jsonwebtoken | JWT auth |
| bcryptjs | Password hashing |
| OpenAI SDK | Gemini API (via compat layer) |
| ImageKit SDK | Image upload & storage |
| Stripe SDK | Payments |
| Axios | ImageKit AI generation calls |
| dotenv | Environment variables |

---

## Project Structure

```
Weaver-AI/
├── client/                   # React frontend (Vite)
│   ├── public/
│   ├── src/
│   │   ├── assets/
│   │   │   ├── assets.js     # All SVG/image imports + dummy data
│   │   │   └── prism.css     # Code highlighting theme
│   │   ├── components/
│   │   │   ├── ChatBox.jsx   # Main chat interface
│   │   │   ├── Message.jsx   # Single message bubble
│   │   │   └── Sidebar.jsx   # Left sidebar with chat list
│   │   ├── context/
│   │   │   └── AppContext.jsx # Global state (user, chats, token, theme)
│   │   ├── pages/
│   │   │   ├── Login.jsx     # Login & Register form
│   │   │   ├── Credits.jsx   # Credit plans & Stripe purchase
│   │   │   ├── Community.jsx # Published AI images gallery
│   │   │   └── Loading.jsx   # Post-payment loading screen
│   │   ├── App.jsx           # Root component & routing
│   │   ├── main.jsx          # React entry point
│   │   └── index.css         # Global styles & animations
│   ├── .env
│   ├── vercel.json
│   └── package.json
│
└── server/                   # Node.js + Express backend
    ├── configs/
    │   ├── db.js             # MongoDB connection
    │   ├── imageKit.js       # ImageKit SDK instance
    │   └── openai.js         # OpenAI SDK → Gemini config
    ├── controllers/
    │   ├── userController.js   # Register, Login, GetUser, PublishedImages
    │   ├── chatController.js   # Create, Get, Delete chats
    │   ├── messageController.js # Text & Image AI generation
    │   ├── creditController.js  # Plans & Stripe checkout
    │   └── webhook.js          # Stripe webhook handler
    ├── middlewares/
    │   └── auth.js           # JWT verification middleware
    ├── models/
    │   ├── User.js           # User schema
    │   ├── Chat.js           # Chat + Messages schema
    │   └── Transaction.js    # Stripe transaction schema
    ├── routes/
    │   ├── userRoutes.js
    │   ├── chatRoutes.js
    │   ├── messageRoutes.js
    │   └── creditRoutes.js
    ├── server.js             # Express app entry point
    ├── vercel.json
    └── package.json
```

---

## Database Schema

### User
```js
{
  name:     String,   // required
  email:    String,   // required, unique
  password: String,   // bcrypt hashed (pre-save hook)
  credits:  Number    // default: 20
}
```

### Chat
```js
{
  userId:   String,   // ref to User
  userName: String,
  name:     String,   // "New Chat"
  messages: [
    {
      role:        String,   // "user" | "assistant"
      content:     String,   // text response or image URL
      isImage:     Boolean,
      isPublished: Boolean,  // true = visible in community
      timestamp:   Number
    }
  ]
}
// timestamps: true (createdAt, updatedAt auto-added)
```

### Transaction
```js
{
  userId:  ObjectId,  // ref to User
  planId:  String,    // "basic" | "pro" | "premium"
  amount:  Number,
  credits: Number,
  isPaid:  Boolean    // default: false, set true by Stripe webhook
}
```

---

## API Endpoints

### Auth — `/api/user`
| Method | Endpoint | Auth | Description |
|---|---|---|---|
| POST | `/api/user/register` | ❌ | Create account, returns JWT |
| POST | `/api/user/login` | ❌ | Login, returns JWT |
| GET | `/api/user/data` | ✅ | Get current user data |
| GET | `/api/user/published-images` | ❌ | Get all community images |

### Chats — `/api/chat`
| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/api/chat/create` | ✅ | Create new chat |
| GET | `/api/chat/get` | ✅ | Get all user chats |
| POST | `/api/chat/delete` | ✅ | Delete a chat by ID |

### Messages — `/api/message`
| Method | Endpoint | Auth | Cost | Description |
|---|---|---|---|---|
| POST | `/api/message/text` | ✅ | 1 credit | Send text prompt to Gemini |
| POST | `/api/message/image` | ✅ | 2 credits | Generate image from prompt |

### Credits — `/api/credit`
| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/api/credit/plan` | ❌ | Get available plans |
| POST | `/api/credit/purchase` | ✅ | Create Stripe checkout session |

### Stripe Webhook
| Method | Endpoint | Auth | Description |
|---|---|---|---|
| POST | `/api/stripe` | Stripe Sig | Handle payment success, add credits |

---

## Environment Variables

### `server/.env`
```env
MONGODB_URI=mongodb+srv://<user>:<pass>@cluster.mongodb.net
JWT_SECRET=your_jwt_secret

IMAGEKIT_PUBLIC_KEY=your_imagekit_public_key
IMAGEKIT_PRIVATE_KEY=your_imagekit_private_key
IMAGEKIT_URL_ENDPOINT=https://ik.imagekit.io/your_id

GEMINI_API_KEY=your_google_gemini_api_key

STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

PORT=3000
```

### `client/.env`
```env
VITE_SERVER_URL=http://localhost:3000
```

---

## Getting Started

### Prerequisites
- Node.js v18+
- MongoDB Atlas account (or local MongoDB)
- ImageKit account
- Google AI Studio API key (for Gemini)
- Stripe account

### 1. Clone the Repository
```bash
git clone https://github.com/your-username/weaver-ai.git
cd weaver-ai
```

### 2. Setup the Backend
```bash
cd server
npm install
# Create .env file and fill in all variables (see above)
npm run server   # starts with nodemon
```

### 3. Setup the Frontend
```bash
cd client
npm install
# Create .env file: VITE_SERVER_URL=http://localhost:3000
npm run dev
```

### 4. Setup Stripe Webhooks (for local dev)
```bash
# Install Stripe CLI and login
stripe listen --forward-to localhost:3000/api/stripe
```
Copy the webhook signing secret from the CLI output into `STRIPE_WEBHOOK_SECRET`.

---

## How It Works

### Authentication Flow
```
User submits Login form
  → POST /api/user/login
  → bcrypt.compare(password, hash)
  → JWT signed with user._id
  → Token stored in localStorage
  → AppContext fetches user data
  → App renders protected routes
```

### Text Generation Flow
```
User types prompt → selects "Text" mode → hits Send
  → POST /api/message/text  { chatId, prompt }
  → auth middleware verifies JWT → req.user attached
  → check user.credits >= 1
  → openai.chat.completions.create({ model: "gemini-2.5-flash" })
  → reply saved to Chat.messages in MongoDB
  → user.credits -= 1
  → reply returned to frontend → rendered via react-markdown
```

### Image Generation Flow
```
User types prompt → selects "Image" mode → hits Send
  → POST /api/message/image  { chatId, prompt, isPublished }
  → auth middleware verifies JWT
  → check user.credits >= 2
  → Build ImageKit AI URL: ENDPOINT/ik-genimg-prompt-{encodedPrompt}/...
  → axios.get(url, { responseType: "arraybuffer" })
  → Convert buffer to base64
  → imagekit.upload(base64, filename, folder)
  → Permanent image URL saved to Chat.messages
  → user.credits -= 2
  → URL returned to frontend → <img> rendered
```

### Payment Flow
```
User visits /credits → sees 3 plans
  → clicks "Buy Now"
  → POST /api/credit/purchase  { planId }
  → Transaction created (isPaid: false)
  → stripe.checkout.sessions.create(...)
  → Frontend redirected to Stripe Checkout page
  → User pays on Stripe
  → Stripe sends "payment_intent.succeeded" to /api/stripe webhook
  → Webhook verifies signature using STRIPE_WEBHOOK_SECRET
  → Transaction.isPaid = true
  → User.credits += plan.credits
  → User redirected to /loading page → after 8s fetchUser() called → credits updated in UI
```

---

## Deployment

Both `client` and `server` are deployed separately on **Vercel**.

### Server (`server/vercel.json`)
Routes all traffic to `server.js` using the `@vercel/node` runtime.

### Client (`client/vercel.json`)
All routes rewrite to `/index.html` to support React Router's client-side routing (SPA fallback).

---

## Credit Plans

| Plan | Price | Credits | Best For |
|---|---|---|---|
| Basic | $10 | 100 | Casual users |
| Pro | $20 | 500 | Regular users |
| Premium | $30 | 1000 | Power users |

---

## Key Design Decisions

- **OpenAI SDK → Gemini**: Used the OpenAI-compatible SDK with Google's `baseURL` endpoint. This means swapping AI providers in the future requires only changing the `baseURL` and `model` name.
- **Stripe Webhook over Redirect**: Credits are only added after the webhook fires (not on redirect), preventing users from getting credits without actually paying.
- **Optimistic UI**: The user's message appears in the chat instantly before the API responds, making the app feel fast.
- **express.raw() for Webhook**: The Stripe webhook route uses raw body parsing (not JSON) so Stripe can verify the payload signature correctly.
- **MongoDB Aggregation**: The community gallery uses `$unwind` + `$match` on the nested `messages` array to extract published images without loading entire chat documents.
