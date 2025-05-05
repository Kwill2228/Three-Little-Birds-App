// App Entry Point - index.js or App.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { StripeProvider } from '@stripe/stripe-react-native';
import { GestureHandlerRootView } from 'react-native-gesture-handler';
import { TailwindProvider } from 'tailwind-rn';
import utilities from './tailwind.json';

import Marketplace from './screens/Marketplace';
import Meditation from './screens/Meditation';
import Dashboard from './screens/Dashboard';
import ProductDetail from './screens/ProductDetail';

const Stack = createNativeStackNavigator();

export default function App() {
  return (
    <GestureHandlerRootView style={{ flex: 1 }}>
      <StripeProvider publishableKey="your-publishable-key">
        <TailwindProvider utilities={utilities}>
          <NavigationContainer>
            <Stack.Navigator initialRouteName="Marketplace">
              <Stack.Screen name="Marketplace" component={Marketplace} />
              <Stack.Screen name="Meditation" component={Meditation} />
              <Stack.Screen name="Dashboard" component={Dashboard} />
              <Stack.Screen name="ProductDetail" component={ProductDetail} />
            </Stack.Navigator>
          </NavigationContainer>
        </TailwindProvider>
      </StripeProvider>
    </GestureHandlerRootView>
  );
}

// Firebase Setup - firebase.js
import { initializeApp } from 'firebase/app';
import { getFirestore } from 'firebase/firestore';

const firebaseConfig = {
  apiKey: 'your-api-key',
  authDomain: 'your-auth-domain',
  projectId: 'your-project-id',
  storageBucket: 'your-storage-bucket',
  messagingSenderId: 'your-messaging-sender-id',
  appId: 'your-app-id'
};

const app = initializeApp(firebaseConfig);
export const db = getFirestore(app);

// Firestore Operations - services/firestoreService.js
import { db } from '../firebase';
import {
  collection, getDocs, addDoc, updateDoc, doc
} from 'firebase/firestore';

export const fetchProducts = async () => {
  const snapshot = await getDocs(collection(db, 'products'));
  return snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
};

export const addProduct = async (product) => {
  await addDoc(collection(db, 'products'), product);
};

export const updateProduct = async (id, updatedProduct) => {
  const productRef = doc(db, 'products', id);
  await updateDoc(productRef, updatedProduct);
};

export const fetchScriptures = async () => {
  const snapshot = await getDocs(collection(db, 'scriptures'));
  return snapshot.docs.map(doc => doc.data());
};

export const addManufacturer = async (manufacturer) => {
  await addDoc(collection(db, 'manufacturers'), manufacturer);
};

export const fetchManufacturers = async () => {
  const snapshot = await getDocs(collection(db, 'manufacturers'));
  return snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
};

// Stripe Integration - services/stripeService.js
import { loadStripe } from '@stripe/stripe-js';
const stripePromise = loadStripe('your-publishable-key');

export const createCheckoutSession = async (userId) => {
  const res = await fetch('https://your-backend.com/create-checkout-session', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ userId })
  });
  const session = await res.json();
  const stripe = await stripePromise;
  await stripe.redirectToCheckout({ sessionId: session.id });
};

// Mobile Screens
// Example: screens/Marketplace.js
import React, { useEffect, useState } from 'react';
import { View, Text, FlatList, Image, TouchableOpacity } from 'react-native';
import { fetchProducts } from '../services/firestoreService';

const Marketplace = () => {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    const loadProducts = async () => {
      const data = await fetchProducts();
      setProducts(data);
    };
    loadProducts();
  }, []);

  const renderItem = ({ item }) => (
    <View className="p-4 bg-white rounded-2xl shadow mb-4">
      <Image source={{ uri: item.imageUrl }} className="w-full h-48 rounded-xl" />
      <Text className="text-lg font-bold mt-2">{item.title}</Text>
      <Text className="text-sm">${item.price}</Text>
    </View>
  );

  return (
    <FlatList
      data={products}
      keyExtractor={(item) => item.id}
      renderItem={renderItem}
      contentContainerStyle={{ padding: 16 }}
    />
  );
};

export default Marketplace;

// Note: Similar component structure can be followed for Meditation.js, Dashboard.js, ProductDetail.js

// Styling
// tailwind.config.js (for Expo w/ tailwind-rn)
module.exports = {
  theme: {
    extend: {
      colors: {
        beige: '#f5f5dc',
        maroon: '#800000',
        navy: '#0b1c2c'
      }
    }
  },
  content: [
    './App.{js,jsx,ts,tsx}',
    './screens/**/*.{js,jsx,ts,tsx}'
  ]
};

// App Build Instructions
// - Web: use Next.js or React + Vite
// - Mobile: use Expo CLI or EAS Build
// - Output: generate .apk for Android and .ipa for iOS
// - Host web on Firebase Hosting or Vercel
// - Submit mobile builds to Google Play Store and Apple App Store

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
    "tailwindcss": "^3.4.0"
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

// Directory Structure (Monorepo):
// three-little-birds/      <-- ✅ Monorepo root
// ├── apps/
// │   ├── web/             <-- React web frontend
// │   ├── mobile/          <-- React Native or Expo mobile app
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

