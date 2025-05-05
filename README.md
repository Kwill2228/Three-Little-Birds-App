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
