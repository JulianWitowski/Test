// /api/ingest.js — Vercel Edge Function proxy to n8n
export const config = { runtime: 'edge' };

function corsHeaders(req) {
  const origin = req.headers.get('origin') || '*';
  return {
    'Access-Control-Allow-Origin': origin,
    'Access-Control-Allow-Methods': 'POST, OPTIONS',
    'Access-Control-Allow-Headers': 'content-type,x-proxy-key',
    'Vary': 'Origin',
    'Content-Type': 'text/plain; charset=utf-8'
  };
}

export default async function handler(req) {
  if (req.method === 'OPTIONS') {
    return new Response(null, { headers: corsHeaders(req) });
  }
  if (req.method !== 'POST') {
    return new Response('Only POST', { status: 405, headers: corsHeaders(req) });
  }

  // Optional shared key to prevent abuse
  const shared = process.env.SHARED_KEY;
  const provided = req.headers.get('x-proxy-key') || '';
  if (shared && provided !== shared) {
    return new Response('Unauthorized', { status: 401, headers: corsHeaders(req) });
  }

  const bodyText = await req.text(); // pass-through JSON
  const resp = await fetch(process.env.N8N_WEBHOOK_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: bodyText
  });

  const text = await resp.text();
  return new Response(text || 'ok', { status: resp.status, headers: corsHeaders(req) });
}
