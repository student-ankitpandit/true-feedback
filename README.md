# True Feedback

**True Feedback** is an anonymous messaging platform built with Next.js — every user gets a personal, shareable link where anyone can send them honest, anonymous feedback. Think of it as a lightweight, self-hosted "Qooh.me"-style app: no sign-in is required to *send* a message, only to *receive* and manage them.

🔗 Live site: [truefeedback.in](https://truefeedback.in/)

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [How It Works](#how-it-works)
- [API Reference](#api-reference)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Environment Variables](#environment-variables)
  - [Running Locally](#running-locally)
- [Deployment](#deployment)
- [License](#license)

## Features

- **Anonymous messaging** — Anyone with a user's public link (`/u/[username]`) can send them a message without creating an account.
- **AI-suggested prompts** — An OpenAI-powered endpoint suggests three open-ended, friendly conversation starters to help senders who are stuck on what to write.
- **Secure authentication** — Credential-based auth (email/username + password) via NextAuth.js, with bcrypt password hashing and JWT sessions.
- **Email verification** — New accounts receive a 6-digit verification code by email (via Resend + React Email templates) that expires after 1 hour.
- **Accept/pause messages toggle** — Users can turn message-receiving on or off at any time from their dashboard.
- **Message dashboard** — View all received messages, sorted newest-first, with the ability to delete any message.
- **Route protection middleware** — Signed-in users are redirected away from auth pages; unauthenticated users are redirected away from the dashboard.
- **Responsive UI** — Built with Tailwind CSS and shadcn/ui (Radix primitives) components, including an autoplaying carousel showcasing example messages on the landing page.

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | [Next.js 14](https://nextjs.org/) (App Router) |
| Language | TypeScript |
| Styling / UI | Tailwind CSS, shadcn/ui, Radix UI primitives, Lucide icons |
| Database | MongoDB with [Mongoose](https://mongoosejs.com/) |
| Auth | [NextAuth.js](https://next-auth.js.org/) (Credentials provider, JWT strategy) |
| Forms & Validation | React Hook Form + [Zod](https://zod.dev/) |
| Email | [Resend](https://resend.com/) + [React Email](https://react.email/) components |
| AI | OpenAI API (streaming completions via the Vercel AI SDK) |
| Password Hashing | bcryptjs |

## Project Structure

```
true-feedback/
├── emails/
│   └── VerificationEmail.tsx      # React Email template for the verification code
├── public/                        # Static assets
├── src/
│   ├── app/
│   │   ├── (app)/                 # Authenticated app shell
│   │   │   ├── dashboard/         # Message dashboard (view/delete/accept toggle)
│   │   │   └── page.tsx           # Landing page with message carousel
│   │   ├── (auth)/
│   │   │   ├── sign-in/
│   │   │   ├── sign-up/
│   │   │   └── verify/[username]/ # Email verification code entry
│   │   ├── api/
│   │   │   ├── accept-messages/   # GET/POST toggle for message acceptance
│   │   │   ├── auth/[...nextauth]/# NextAuth route + authOptions
│   │   │   ├── check-username-unique/
│   │   │   ├── delete-message/[messageid]/
│   │   │   ├── get-messages/
│   │   │   ├── send-message/
│   │   │   ├── sign-up/
│   │   │   ├── suggest-messages/  # OpenAI-powered prompt suggestions
│   │   │   └── verifycode/
│   │   └── u/[username]/          # Public page for sending someone a message
│   ├── components/                # Navbar, MessageCard, shadcn/ui components
│   ├── context/                   # React context providers
│   ├── helpers/                   # sendVerificationEmail, etc.
│   ├── hooks/                     # Custom hooks
│   ├── lib/                       # dbConnect, resend client, utils
│   ├── model/                     # Mongoose User/Message schema
│   ├── schemas/                   # Zod schemas (sign-up, sign-in, message, etc.)
│   ├── types/                     # Shared TypeScript types
│   ├── messages.json              # Sample messages for the landing page carousel
│   └── middleware.ts              # Route protection logic
├── .env.sample
├── next.config.mjs
├── tailwind.config.ts
└── package.json
```

## How It Works

1. **Sign up** — A user registers with a username, email, and password. A hashed password is stored, and a 6-digit verification code (valid for 1 hour) is emailed to them.
2. **Verify** — The user enters the code at `/verify/[username]` to activate their account.
3. **Sign in** — Verified users log in via NextAuth's credentials provider; a JWT session is issued containing their `_id`, `username`, `isVerified`, and `isAcceptingMessages` status.
4. **Share the link** — Each user's public feedback page lives at `/u/[username]`.
5. **Receive feedback** — Anyone (no login required) can visit that link and submit an anonymous message, optionally using AI-generated conversation starters. Messages are only accepted if the recipient hasn't paused their inbox.
6. **Manage messages** — From `/dashboard`, the user can view all messages (newest first), delete unwanted ones, and toggle whether they're currently accepting new messages.

## API Reference

All routes live under `src/app/api` and return JSON in the shape `{ success: boolean, message?: string, ... }`.

| Method | Route | Auth required | Description |
|---|---|---|---|
| `POST` | `/api/sign-up` | No | Register a new user and trigger a verification email |
| `POST` | `/api/verifycode` | No | Verify a user's email using their 6-digit code |
| `GET` | `/api/check-username-unique` | No | Check whether a username is available |
| `POST`/`GET` | `/api/accept-messages` | Yes | Update / fetch a user's "accepting messages" status |
| `POST` | `/api/send-message` | No | Send an anonymous message to a given username |
| `GET` | `/api/get-messages` | Yes | Fetch the signed-in user's messages, sorted newest-first |
| `DELETE` | `/api/delete-message/[messageid]` | Yes | Delete a specific message by ID |
| `POST` | `/api/suggest-messages` | No | Stream 3 AI-generated, open-ended message prompts |
| `*` | `/api/auth/[...nextauth]` | — | NextAuth.js authentication handler |

## Getting Started

### Prerequisites

- Node.js 18+
- A MongoDB database (e.g. [MongoDB Atlas](https://www.mongodb.com/atlas))
- A [Resend](https://resend.com/) API key for sending emails
- An [OpenAI](https://platform.openai.com/) API key for the message-suggestion feature

### Installation

```bash
git clone https://github.com/student-ankitpandit/true-feedback.git
cd true-feedback
npm install
```

### Environment Variables

Copy `.env.sample` to `.env` and fill in the values:

```bash
cp .env.sample .env
```

```env
MONGO_URI=""            # Your MongoDB connection string
RESEND_API_KEY=""       # Resend API key for verification emails
NEXTAUTH_SECRET_KEY=""  # Secret used to sign NextAuth JWTs
OPENAI_API_KEY=""       # OpenAI API key for AI-suggested messages
```

### Running Locally

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
# or
bun dev
```

Open [http://localhost:3000](http://localhost:3000) to see the app. The page auto-updates as you edit files under `src/app`.

Other available scripts:

```bash
npm run build   # Production build
npm run start   # Start the production server
npm run lint    # Run ESLint
```

## Deployment

The easiest way to deploy this app is via the [Vercel Platform](https://vercel.com/new), the creators of Next.js. Make sure to add the same environment variables (`MONGO_URI`, `RESEND_API_KEY`, `NEXTAUTH_SECRET_KEY`, `OPENAI_API_KEY`) in your Vercel project settings.

See the [Next.js deployment documentation](https://nextjs.org/docs/deployment) for more details.

## License

No license file is currently included in this repository. Consider adding one (e.g. MIT) if you intend for others to reuse this code.
