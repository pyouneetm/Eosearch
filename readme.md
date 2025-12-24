Eosearch.js

A powerful, lightweight, and zero-dependency search engine for the browser. Eosearch combines fuzzy text search, geospatial querying, and advanced filtering into a single, easy-to-use asynchronous API.

Features

⚡ Async Architecture: Built with a non-blocking, worker-like architecture to keep your UI responsive during heavy searches.

🔍 Fuzzy Search: Uses Levenshtein distance to find matches even with typos.

🌍 Geospatial Search: Built-in searchNearby functionality to filter items by latitude/longitude radius.

🧠 Advanced Query Syntax: Supports logical operators (AND, OR, NOT), exact matches, and range filters (e.g., price>100).

🗣️ Phonetic Search: Optional Soundex algorithm to find words that sound similar (e.g., "Smith" finds "Smyth").

💾 Caching: Internal LRU caching for instant results on repeated queries.

🌐 I18n Ready: Unicode normalization and built-in localized placeholders for 17+ languages.

✨ UI Assistant: Includes EosearchAssistant to automatically handle input placeholders and query suggestions.

Installation

Browser

Include the script directly in your HTML file:

<script src="eosearch.js"></script>


Node.js / CommonJS

const { Eosearch } = require('./eosearch.js');


Quick Start

1. Initialize the Engine

Prepare your data and initialize Eosearch. You can specify which keys to search and assign weights to them.

const books = [
  { id: 1, title: "The Great Gatsby", author: "F. Scott Fitzgerald", year: 1925, price: 10.99 },
  { id: 2, title: "War and Peace", author: "Leo Tolstoy", year: 1869, price: 12.50 },
  { id: 3, title: "The Catcher in the Rye", author: "J.D. Salinger", year: 1951, price: 9.99 },
  { id: 4, title: "1984", author: "George Orwell", year: 1949, price: 8.99 }
];

const options = {
  keys: [
    { name: 'title', weight: 1.0 },
    { name: 'author', weight: 0.7 }
  ],
  threshold: 0.4, // Match sensitivity (0.0 = perfect match, 1.0 = match anything)
};

const search = new Eosearch(books, options);


2. Perform a Basic Search

search() returns a Promise that resolves with the results.

search.search('Orwell').then(results => {
  console.log(results);
  // Output: [{ item: { title: "1984", ... }, score: 0.1, ... }]
});


3. Stream Results (Real-time)

For large datasets, use searchStream to handle results as they are found, rather than waiting for the entire batch.

search.searchStream('Great', {
  onResult: (result) => {
    console.log('Found one:', result.item.title);
  },
  onComplete: (summary) => {
    console.log('Done! Total found:', summary.results.length);
  }
});


Query Syntax

Eosearch supports a rich query syntax out of the box:

Syntax

Description

Example

term

Fuzzy search for "term"

apple

"term"

Exact match required

"apple"

+term

AND operator (must include)

+iphone +black

!term

NOT operator (must not include)

apple !pie

field>val

Range filter (supports >, <, =, >=, <=)

price<=10 or year>1950

Example Complex Query:

// Find items containing "phone" but not "case", costing less than 500
search.search('phone !case price<500');


Geospatial Search

If your data includes latitude and longitude, you can search within a radius.

Data Requirements: Items must have latitude/longitude OR lat/lon properties.

const places = [
  { name: "Central Park", lat: 40.785091, lon: -73.968285 },
  { name: "Empire State Building", lat: 40.748817, lon: -73.985428 }
];

const geoSearch = new Eosearch(places);

// Find places within 5km of Times Square
geoSearch.searchNearby({
  latitude: 40.7580, 
  longitude: -73.9855, 
  radiusKm: 5
}, {
  onResult: (res) => console.log(`${res.item.name} is ${res.distance.toFixed(2)}km away`),
  onComplete: () => console.log('Search complete')
});


EosearchAssistant (UI Helper)

A helper class to create an interactive search experience with cycling localized placeholders ("Search for anything...", "¿Qué conocimiento buscas?...", etc.).

const input = document.getElementById('search-input');
const resultsContainer = document.getElementById('results-div');

// Initialize the assistant
const assistant = new EosearchAssistant(input, resultsContainer, {
  promptInterval: 3000 // Cycle placeholder text every 3 seconds
});

// To clean up later
// assistant.destroy();


Configuration Options

Option

Type

Default

Description

keys

Array

[]

Array of objects { name: 'field', weight: 1 }.

threshold

Number

0.6

Threshold for fuzzy matching (0.0 to 1.0). Lower is stricter.

usePhonetic

Boolean

false

Enable Soundex phonetic matching (e.g., "Jon" matches "John").

useCache

Boolean

true

Enable LRU caching for faster repeated queries.

cacheSize

Number

50

Number of queries to store in cache.

includeMatches

Boolean

false

Return match indices (useful for highlighting).

tokenizer

Function

internal

Custom function to split query strings into tokens.

Methods

search(query)

Returns a Promise resolving to an array of results.

add(item)

Adds a single item to the search index dynamically.

remove(id)

Removes an item from the index.
Note: This assumes you pass a predicate function or an ID matching item.id.

highlight(match, highlightFn)

Static helper to wrap matched text.

const html = Eosearch.highlight(result.matches[0], text => `<mark>${text}</mark>`);


exportIndex() / importIndex(data)

Persist the index to IndexedDB for offline capability or faster startup on subsequent visits.

await search.exportIndex();
// Later...
const restoredSearch = await Eosearch.fromIndex();


License

MIT
