// Root: three-little-birds/package.json
{
  "name": "three-little-birds",
  "private": true,
  "workspaces": ["apps/*", "packages/*"],
  "scripts": {
    "dev:web": "pnpm --filter web dev",
    "dev:admin": "pnpm --filter admin dev",
    "dev:mobile": "pnpm --filter mobile dev",
    "build": "turbo run build",
    "lint": "turbo run lint"
  },
  "devDependencies": {
    "turbo": "^1.10.0"
  }
}
// Root: turbo.json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "lint": {},
    "dev": {}
  }
}
// apps/web/package.json
{
  "name": "web",
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  },
  "dependencies": {
    "next": "13.4.10",
    "react": "18.2.0",
    "react-dom": "18.2.0",
    "@stripe/stripe-js": "^1.53.0",
    "firebase": "^10.0.0",
    "tailwindcss": "^3.4.0",
    "@stripe/react-stripe-js": "^1.11.0",
    "@headlessui/react": "^1.7.5",
    "@heroicons/react": "^2.0.18"
  }
}
// apps/web/tailwind.config.js
module.exports = {
  content: [
    "../../packages/ui/**/*.{js,ts,jsx,tsx}",
    "./pages/**/*.{js,ts,jsx,tsx}",
    "./components/**/*.{js,ts,jsx,tsx}"
  ],
  theme: {
    extend: {
      colors: {
        beige: '#f5f5dc',
        maroon: '#800000',
        navy: '#0b1c2c'
      },
      backgroundImage: {
        'nature-pattern': "url('/assets/backgrounds/nature.jpg')",
        'sheep-pasture': "url('/assets/backgrounds/sheep.jpg')"
      },
      fontFamily: {
        sans: ['ui-sans-serif', 'system-ui'],
        serif: ['ui-serif', 'Georgia'],
        mono: ['ui-monospace', 'SFMono-Regular']
      }
    }
  },
  plugins: []
};
// packages/firebase/index.js
import { initializeApp } from 'firebase/app';
import { getFirestore } from 'firebase/firestore';
const firebaseConfig = {
  apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
  authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
  storageBucket: process.env.NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
  appId: process.env.NEXT_PUBLIC_FIREBASE_APP_ID
};
const app = initializeApp(firebaseConfig);
export const db = getFirestore(app);
// packages/stripe/index.js
import { loadStripe } from '@stripe/stripe-js';
const stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY);
export const redirectToCheckout = async (sessionId) => {
  const stripe = await stripePromise;
  await stripe.redirectToCheckout({ sessionId });
};
// AWS Setup Notes:
// 1. Use AWS Amplify to host the web app (`apps/web`):
//    amplify init
//    amplify add hosting
//    amplify publish
//
// 2. Use AWS Lambda + API Gateway for functions:
//    deploy Firebase webhooks or subscription logic as serverless functions
//
// 3. Use Amazon S3 for asset storage (backgrounds, logos)
// 4. Store Firebase keys in Amplify environment variables
// 5. Use AWS Cognito or Firebase Auth for authentication
// 6. Configure Stripe + Klarna + Affirm + Afterpay + Sezzle for subscription billing options.
// Frontend Build Configuration:
// - Build Command: next build
// - Output Directory: .next
// Directory Structure (Monorepo):
// three-little-birds/      <-- amplify pull --appId d299lsp3i3kyou --envName staging
// ├── apps/                <-- mkdir -p apps/web apps/mobile apps/admin
// │   ├── web/             <-- import Amplify from 'aws-amplify';
import awsExports from '../../amplify/aws-exports'; // Adjust path if needed
Amplify.configure(awsExports);
// │   ├── mobile/          <-- import Amplify from 'aws-amplify';
import awsExports from '../../amplify/aws-exports'; // Adjust path
Amplify.configure(awsExports);
// │   └── admin/           <-- Optional admin dashboard
// ├── packages/
// │   ├── ui/              <-- Reusable UI components
// │   ├── firebase/        <-- Firebase config + utilities
// │   ├── stripe/          <-- Stripe subscription logic
// │   └── utils/           <-- Shared helpers (date, auth, etc.)
// ├── functions/           <-- Firebase Cloud Functions (Stripe webhooks, DB triggers)
// ├── public/              <-- Assets like backgrounds, logos
// ├── .gitignore
// ├── package.json         <-- Root dependencies & scripts
// ├── turbo.json or nx.json
// └── README.md
// Next Step: Create apps/mobile and apps/admin folders, populate them with boilerplate React Native and admin interfaces.