#!/usr/bin/env node

import { writeFileSync } from 'fs';

const PUBLIC_KEY = process.env.PUBLIC_KEY;
const MAX_DEPTH = process.env.MAX_DEPTH ? parseInt(process.env.MAX_DEPTH, 10) : Infinity;
const API_BASE = 'https://cloud-api.yandex.net/v1/disk/public/resources';

if (!PUBLIC_KEY) {
  console.error('ERROR: PUBLIC_KEY environment variable is required');
  process.exit(1);
}

/**
 * Sleep helper for retry backoff
 */
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

/**
 * Fetch with retry logic (max 3 attempts)
 */
async function fetchWithRetry(url, attempt = 1) {
  try {
    const response = await fetch(url);
    
    if (response.ok) {
      return response;
    }
    
    // Handle rate limiting and server errors
    if ((response.status === 429 || response.status >= 500) && attempt < 3) {
      const backoffMs = Math.pow(2, attempt) * 1000; // 2s, 4s, 8s
      console.warn(`HTTP ${response.status} - retrying in ${backoffMs}ms (attempt ${attempt}/3)`);
      await sleep(backoffMs);
      return fetchWithRetry(url, attempt + 1);
    }
    
    // Non-retryable error or max retries exceeded
    const errorText = await response.text();
    throw new Error(`HTTP ${response.status}: ${errorText}`);
    
  } catch (error) {
    if (attempt < 3 && error.message.includes('fetch failed')) {
      const backoffMs = Math.pow(2, attempt) * 1000;
      console.warn(`Network error - retrying in ${backoffMs}ms (attempt ${attempt}/3)`);
      await sleep(backoffMs);
      return fetchWithRetry(url, attempt + 1);
    }
    throw error;
  }
}

/**
 * Fetch a single page from Yandex Disk API
 */
async function fetchPage(publicKey, path, limit = 200, offset = 0) {
  const url = new URL(API_BASE);
  url.searchParams.set('public_key', publicKey);
  url.searchParams.set('path', path);
  url.searchParams.set('limit', limit.toString());
  url.searchParams.set('offset', offset.toString());
  
  console.log(`Fetching: ${path} (offset=${offset})`);
  
  const response = await fetchWithRetry(url.toString());
  return response.json();
}

/**
 * Recursively walk the folder tree
 */
async function walkTree(publicKey, path = '/', depth = 0) {
  if (depth > MAX_DEPTH) {
    console.log(`Max depth reached at: ${path}`);
    return [];
  }
  
  const results = [];
  let offset = 0;
  const limit = 200;
  let hasMore = true;
  
  while (hasMore) {
    const data = await fetchPage(publicKey, path, limit, offset);
    
    if (!data._embedded || !data._embedded.items) {
      hasMore = false;
      break;
    }
    
    const items = data._embedded.items;
    
    if (items.length === 0) {
      hasMore = false;
      break;
    }
    
    // Sort: directories first, then files
    items.sort((a, b) => {
      if (a.type === 'dir' && b.type !== 'dir') return -1;
      if (a.type !== 'dir' && b.type === 'dir') return 1;
      return a.name.localeCompare(b.name);
    });
    
    for (const item of items) {
      const itemPath = item.path.replace(/^disk:/, '');
      results.push({
        path: itemPath,
        type: item.type,
        name: item.name,
        size: item.size || 0,
        modified: item.modified || ''
      });
      
      // Recursively walk subdirectories
      if (item.type === 'dir') {
        const children = await walkTree(publicKey, itemPath, depth + 1);
        results.push(...children);
      }
    }
    
    offset += items.length;
    
    // Check if there are more items
    if (items.length < limit) {
      hasMore = false;
    }
  }
  
  return results;
}

/**
 * Convert results to CSV
 */
function toCSV(items) {
  const header = 'path,type,name,size,modified';
  const rows = items.map(item => {
    return [
      JSON.stringify(item.path),
      JSON.stringify(item.type),
      JSON.stringify(item.name),
      JSON.stringify(item.size),
      JSON.stringify(item.modified)
    ].join(',');
  });
  
  return [header, ...rows].join('\n');
}

/**
 * Main execution
 */
async function main() {
  try {
    console.log('Starting Yandex Disk scan...');
    console.log(`Public key: ${PUBLIC_KEY.substring(0, 30)}...`);
    console.log(`Max depth: ${MAX_DEPTH === Infinity ? 'unlimited' : MAX_DEPTH}`);
    
    const items = await walkTree(PUBLIC_KEY);
    
    console.log(`\nFound ${items.length} items`);
    
    const csv = toCSV(items);
    writeFileSync('tree.csv', csv, 'utf8');
    
    console.log('✓ tree.csv written successfully');
    
  } catch (error) {
    console.error('ERROR:', error.message);
    process.exit(1);
  }
}

main();
