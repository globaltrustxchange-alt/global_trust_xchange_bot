/* GTX — Global TrustXchange React + Tailwind single-file app (App.jsx)

Features implemented:

Header with GTX logo placeholder

Main navigation tiles (Rates, Currencies, FAQs, Help, About)

'Mini inline buttons' sticky bottom bar: WhatsApp, Catalog, Telegram, X, Submit Card

Rates panel that fetches from an API endpoint (/api/rates) with graceful fallback to sample data

Submit Card modal form that posts to /api/submit (adjust backend URL as needed)

FAQ accordion, Help and About sections

Mobile-first responsive layout styled with Tailwind (no colors locked; you can change easily)


How to use:

1. Add this file to a React project (e.g. create-react-app). Ensure Tailwind is configured.


2. Replace API endpoints (/api/rates, /api/submit) with your own server endpoints (or proxy to Google Sheets via server-side code).


3. Swap the GTX logo (placeholder) with your real SVG or image.



This file is intentionally self-contained and focuses on UI/UX and flow. Backend integration (Google Sheets & Telegram forwarding) should run server-side and expose the two endpoints mentioned above. */

import React, { useEffect, useState } from "react";

export default function App() { const [rates, setRates] = useState(null); const [loadingRates, setLoadingRates] = useState(true); const [activeView, setActiveView] = useState("home"); const [showSubmit, setShowSubmit] = useState(false); const [submitState, setSubmitState] = useState({ loading: false, ok: null, error: null });

// Form state const [form, setForm] = useState({ card: "", value: "", country: "", code: "", whatsapp: "" });

useEffect(() => { fetchRates(); }, []);

async function fetchRates() { setLoadingRates(true); try { const res = await fetch("/api/rates"); if (!res.ok) throw new Error("Network"); const data = await res.json(); setRates(data); } catch (e) { // fallback sample rates setRates(sampleRates()); } setLoadingRates(false); }

function sampleRates() { return [ { card: "UK Steam Physical", usd_value: "100", rate: 1080 }, { card: "UK Steam ecode", usd_value: "100", rate: 1050 }, { card: "Swiss Steam (>$100)", usd_value: "100", rate: 975 }, { card: "US Nordstrom Physical", usd_value: "100", rate: 850 }, { card: "Razer Gold", usd_value: "100", rate: 910 }, ]; }

function top8Text() { if (!rates) return ""; const gift = rates.filter(r => !["BTC", "USDT", "ETH"].includes((r.card || "").toUpperCase())); const top = gift.slice(0, 8); return top.map((r) => ${r.card} $${r.usd_value} : ₦${Math.round(r.usd_value * r.rate).toLocaleString()}).join('\n'); }

function handleChange(e) { setForm(prev => ({ ...prev, [e.target.name]: e.target.value })); }

async function handleSubmit(e) { e.preventDefault(); setSubmitState({ loading: true, ok: null, error: null }); try { const payload = { ...form, timestamp: new Date().toISOString() }; const res = await fetch("/api/submit", { method: "POST", headers: { "Content-Type": "application/json" }, body: JSON.stringify(payload), }); if (!res.ok) throw new Error("Failed to submit"); const json = await res.json(); setSubmitState({ loading: false, ok: true, error: null }); setForm({ card: "", value: "", country: "", code: "", whatsapp: "" }); } catch (err) { setSubmitState({ loading: false, ok: false, error: err.message || "Error" }); } }

return ( <div className="min-h-screen bg-gray-50 text-gray-900"> {/* Header */} <header className="bg-white shadow-sm sticky top-0 z-30"> <div className="max-w-4xl mx-auto px-4 py-4 flex items-center justify-between"> <div className="flex items-center space-x-3"> <div className="w-12 h-12 bg-gradient-to-tr from-blue-600 to-cyan-400 rounded-full flex items-center justify-center text-white font-bold">GTX</div> <div> <div className="font-semibold">Global TrustXchange</div> <div className="text-xs text-gray-500">Fast • Trusted • Secure</div> </div> </div> <div className="hidden sm:flex items-center space-x-2"> <button onClick={() => { setActiveView('rates'); }} className="px-3 py-2 text-sm rounded hover:bg-gray-100">Rates</button> <button onClick={() => { setActiveView('currencies'); }} className="px-3 py-2 text-sm rounded hover:bg-gray-100">Currencies</button> <button onClick={() => { setActiveView('faq'); }} className="px-3 py-2 text-sm rounded hover:bg-gray-100">FAQs</button> <button onClick={() => { setActiveView('help'); }} className="px-3 py-2 text-sm rounded hover:bg-gray-100">Help</button> <button onClick={() => { setActiveView('about'); }} className="px-3 py-2 text-sm rounded hover:bg-gray-100">About</button> </div> <div className="sm:hidden"> <button onClick={() => setShowSubmit(true)} className="px-3 py-2 bg-blue-600 text-white rounded">Submit</button> </div> </div> </header>

{/* Main content */}
  <main className="max-w-4xl mx-auto px-4 py-6">
    <section className="bg-white rounded-lg p-5 shadow-sm mb-6">
      <div className="flex items-start gap-4">
        <div className="flex-1">
          <h1 className="text-xl font-semibold">Welcome to Global TrustXchange</h1>
          <p className="mt-2 text-sm text-gray-600">Your trusted partner for gift card & crypto trading. Below are our top gift card rates. Use the buttons to navigate.</p>
        </div>
        <div className="w-36 text-right hidden sm:block">
          <div className="text-xs text-gray-500">Top 8 Gift Cards</div>
          <pre className="text-sm bg-gray-50 rounded p-2 mt-2 text-right whitespace-pre-wrap">{top8Text()}</pre>
        </div>
      </div>

      {/* Mini navigation tiles that appear inside the welcome card */}
      <div className="mt-4 grid grid-cols-2 sm:grid-cols-5 gap-2">
        <button onClick={() => setActiveView('rates')} className="p-2 text-center rounded border hover:shadow-sm">📈 Rates</button>
        <button onClick={() => setActiveView('currencies')} className="p-2 text-center rounded border hover:shadow-sm">💰 Currencies</button>
        <button onClick={() => setActiveView('faq')} className="p-2 text-center rounded border hover:shadow-sm">📌 FAQs</button>
        <button onClick={() => setActiveView('help')} className="p-2 text-center rounded border hover:shadow-sm">❓ Help</button>
        <button onClick={() => setActiveView('about')} className="p-2 text-center rounded border hover:shadow-sm">ℹ️ About</button>
      </div>
    </section>

    {/* Views */}
    {activeView === 'home' && (
      <section className="grid grid-cols-1 gap-4">
        <Card title="Latest Rates">
          {loadingRates ? <div>Loading rates…</div> : <RatesList rates={rates} />}
        </Card>
        <Card title="How to submit">
          <p className="text-sm text-gray-700">Click <strong>Submit Card Details</strong> (bottom bar) or use the form to send: Card Type, USD Value, Card Code, and WhatsApp number. Admin will contact you.</p>
        </Card>
        <Card title="Why choose GTX">
          <ul className="list-disc pl-5 text-sm text-gray-700">
            <li>Fast payouts</li>
            <li>Competitive rates</li>
            <li>Secure & reliable</li>
          </ul>
        </Card>
      </section>
    )}

    {activeView === 'rates' && (
      <section>
        <h2 className="text-lg font-semibold mb-3">All Rates</h2>
        <div className="bg-white rounded shadow p-4">
          {loadingRates ? <div>Loading rates…</div> : <RatesList rates={rates} />}
        </div>
      </section>
    )}

    {activeView === 'currencies' && (
      <section>
        <h2 className="text-lg font-semibold mb-3">Currencies & Offers</h2>
        <div className="bg-white rounded shadow p-4 text-sm">
          <p>We support gift cards and crypto (BTC, ETH, USDT). For crypto rates contact support.</p>
        </div>
      </section>
    )}

    {activeView === 'faq' && (
      <section>
        <h2 className="text-lg font-semibold mb-3">FAQs</h2>
        <div className="space-y-2">
          <FaqItem q="How to sell gift cards?" a="Send your card details using the submit form or contact us on WhatsApp. We verify and pay instantly." />
          <FaqItem q="How to buy/cashout crypto?" a="We support BTC, ETH, and USDT. Contact us for payment methods and verification." />
          <FaqItem q="Payment options?" a="Bank Transfer, USDT (TRC20), BTC — other options on request." />
        </div>
      </section>
    )}

    {activeView === 'help' && (
      <section>
        <h2 className="text-lg font-semibold mb-3">Help & Support</h2>
        <div className="bg-white rounded shadow p-4 text-sm">
          <p>Contact support on WhatsApp: <a className="text-blue-600" href="https://wa.me/2349162846184">+2349162846184</a></p>
          <p className="mt-2">Telegram: <a className="text-blue-600" href="https://t.me/globaltrustxchange">@globaltrustxchange</a></p>
        </div>
      </section>
    )}

    {activeView === 'about' && (
      <section>
        <h2 className="text-lg font-semibold mb-3">About GTX</h2>
        <div className="bg-white rounded shadow p-4 text-sm">
          <p>GTX — Global TrustXchange is a trusted platform for gift card and crypto trading. Fast payouts, secure verification, and excellent support.</p>
        </div>
      </section>
    )}

  </main>

  {/* Sticky mini inline keyboard (bottom) */}
  <div className="fixed left-0 right-0 bottom-4 z-40 flex justify-center">
    <div className="bg-white shadow-lg rounded-full px-3 py-2 flex items-center gap-2">
      <a className="flex flex-col items-center text-xs" href="https://wa.me/2349162846184?text=Hello">
        <div className="w-10 h-10 rounded-full bg-green-500 flex items-center justify-center text-white">💬</div>
        <div className="mt-1">WhatsApp</div>
      </a>
      <a className="flex flex-col items-center text-xs" href="https://wa.me/c/2349162846184">
        <div className="w-10 h-10 rounded-full bg-gray-800 flex items-center justify-center text-white">🛍</div>
        <div className="mt-1">Catalog</div>
      </a>
      <a className="flex flex-col items-center text-xs" href="https://t.me/globaltrustxchange">
        <div className="w-10 h-10 rounded-full bg-blue-500 flex items-center justify-center text-white">📲</div>
        <div className="mt-1">Telegram</div>
      </a>
      <a className="flex flex-col items-center text-xs" href="https://x.com/GlobalTrustX_">
        <div className="w-10 h-10 rounded-full bg-slate-700 flex items-center justify-center text-white">❌</div>
        <div className="mt-1">X</div>
      </a>
      <button onClick={() => setShowSubmit(true)} className="flex flex-col items-center text-xs">
        <div className="w-10 h-10 rounded-full bg-indigo-600 flex items-center justify-center text-white">📤</div>
        <div className="mt-1">Submit</div>
      </button>
    </div>
  </div>

  {/* Submit Modal */}
  {showSubmit && (
    <div className="fixed inset-0 z-50 flex items-center justify-center bg-black/40">
      <div className="bg-white rounded-lg w-full max-w-md p-4">
        <div className="flex items-center justify-between">
          <h3 className="font-semibold">Submit Card Details</h3>
          <button onClick={() => setShowSubmit(false)} className="text-gray-500">✖</button>
        </div>
        <form className="mt-3 space-y-3" onSubmit={handleSubmit}>
          <div>
            <label className="text-xs text-gray-600">Card Type</label>
            <input name="card" value={form.card} onChange={handleChange} required className="w-full border rounded p-2 text-sm" placeholder="e.g. iTunes, Steam" />
          </div>
          <div>
            <label className="text-xs text-gray-600">USD Value</label>
            <input name="value" value={form.value} onChange={handleChange} required type="number" className="w-full border rounded p-2 text-sm" placeholder="e.g. 100" />
          </div>
          <div>
            <label className="text-xs text-gray-600">Country / Notes</label>
            <input name="country" value={form.country} onChange={handleChange} className="w-full border rounded p-2 text-sm" placeholder="Optional - e.g. UK" />
          </div>
          <div>
            <label className="text-xs text-gray-600">Card Code / Info</label>
            <input name="code" value={form.code} onChange={handleChange} className="w-full border rounded p-2 text-sm" placeholder="Code or additional info" />
          </div>
          <div>
            <label className="text-xs text-gray-600">WhatsApp Number</label>
            <input name="whatsapp" value={form.whatsapp} onChange={handleChange} className="w-full border rounded p-2 text-sm" placeholder="e.g. +2347055..." />
          </div>
          <div className="flex items-center gap-2 mt-2">
            <button disabled={submitState.loading} className="px-4 py-2 bg-indigo-600 text-white rounded">{submitState.loading ? 'Submitting...' : 'Submit'}</button>
            <button type="button" onClick={() => setShowSubmit(false)} className="px-3 py-2 border rounded">Cancel</button>
          </div>
          {submitState.ok && <div className="text-sm text-green-600">Submitted successfully — admin will contact you.</div>}
          {submitState.ok === false && <div className="text-sm text-red-600">Error: {submitState.error}</div>}
        </form>
      </div>
    </div>
  )}

</div>

); }

function Card({ title, children }){ return ( <div className="bg-white rounded shadow p-4"> {title && <h3 className="font-semibold mb-2">{title}</h3>} <div>{children}</div> </div> ); }

function RatesList({ rates }){ if(!rates || rates.length === 0) return <div className="text-sm text-gray-500">No rates available.</div>; return ( <div className="grid grid-cols-1 sm:grid-cols-2 gap-2"> {rates.map((r, i) => ( <div key={i} className="border rounded p-2 flex items-center justify-between"> <div> <div className="font-medium text-sm">{r.card}</div> <div className="text-xs text-gray-500">${r.usd_value} each</div> </div> <div className="text-right"> <div className="font-semibold">₦{Math.round(r.usd_value * r.rate).toLocaleString()}</div> <div className="text-xs text-gray-500">@ ₦{r.rate.toLocaleString()} / $</div> </div> </div> ))} </div> ); }

function FaqItem({ q, a }){ const [open, setOpen] = useState(false); return ( <div className="bg-white rounded shadow-sm p-3"> <button onClick={() => setOpen(o => !o)} className="w-full text-left flex items-center justify-between"> <div className="font-medium">{q}</div> <div className="text-gray-400">{open ? '−' : '+'}</div> </button> {open && <div className="mt-2 text-sm text-gray-700">{a}</div>} </div> ); }
