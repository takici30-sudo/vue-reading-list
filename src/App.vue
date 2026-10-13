<script setup>
import { computed, nextTick, ref, watch } from 'vue';

const storageKey = 'little-shelf.books.v1';
const books = ref([
  { id: 'sample-1', title: 'A notebook of small ideas', author: 'Sample author', read: false },
  { id: 'sample-2', title: 'The art of making things', author: 'Sample author', read: false },
  { id: 'sample-3', title: 'A slower kind of Sunday', author: 'Sample author', read: true },
]);
const title = ref('');
const author = ref('');
const query = ref('');
const filter = ref('all');
const message = ref('Sample books are shown on your first visit.');
const titleInput = ref(null);

try {
  const saved = localStorage.getItem(storageKey);
  if (saved !== null) {
    const parsed = JSON.parse(saved);
    if (!Array.isArray(parsed) || parsed.length > 500 || parsed.some(book =>
      !book || typeof book.id !== 'string' || typeof book.title !== 'string' ||
      !book.title.trim() || book.title.length > 160 || typeof book.author !== 'string' ||
      book.author.length > 100 || typeof book.read !== 'boolean'
    ) || new Set(parsed.map(book => book.id)).size !== parsed.length) {
      throw new Error('Invalid saved reading list');
    }
    books.value = parsed;
    message.value = 'Loaded your reading list from this browser.';
  }
} catch {
  message.value = 'Saved books could not be loaded. Sample books are shown; stored data has not been changed.';
}

const completed = computed(() => books.value.filter(book => book.read).length);
const visibleBooks = computed(() => {
  const search = query.value.trim().toLocaleLowerCase();
  return books.value.filter(book =>
    (filter.value === 'all' || (filter.value === 'read' ? book.read : !book.read)) &&
    (!search || `${book.title} ${book.author}`.toLocaleLowerCase().includes(search))
  );
});
const filters = [
  { value: 'all', label: 'All books' },
  { value: 'unread', label: 'To read' },
  { value: 'read', label: 'Finished' },
];

watch(books, value => {
  try {
    localStorage.setItem(storageKey, JSON.stringify(value));
    message.value = 'Reading list saved in this browser.';
  } catch {
    message.value = 'Local saving is unavailable or full. Changes are kept in this tab only.';
  }
}, { deep: true });

async function addBook() {
  const cleanTitle = title.value.trim();
  if (!cleanTitle) { message.value = 'Please enter a book title.'; titleInput.value?.focus(); return; }
  if (books.value.length >= 500) { message.value = 'This starter supports up to 500 books.'; return; }
  books.value.push({
    id: crypto.randomUUID(), title: cleanTitle, author: author.value.trim(), read: false,
  });
  title.value = ''; author.value = ''; query.value = ''; filter.value = 'all';
  await nextTick();
  titleInput.value?.focus();
}

function removeBook(id) {
  books.value = books.value.filter(book => book.id !== id);
  titleInput.value?.focus();
}
</script>

<template>
  <div class="page">
    <header><a class="brand" href="#main"><span aria-hidden="true">▥</span> little shelf</a><span class="edition">A Vue reading-list starter</span></header>
    <main id="main">
      <section class="intro" aria-labelledby="page-title">
        <div><p class="eyebrow">Make space for a good book</p><h1 id="page-title">Your next chapter<br>starts here.</h1><p class="lede">Keep a small shelf of things you want to read.<br>One title, one idea, one unhurried afternoon.</p></div>
        <aside class="reading-count" aria-label="Reading progress"><strong>{{ completed }}<span>/ {{ books.length }}</span></strong><p>books finished</p><div class="progress" role="progressbar" aria-label="Books finished" :aria-valuenow="completed" :aria-valuemax="books.length || 1" aria-valuemin="0"><div :style="{ width: `${books.length ? completed / books.length * 100 : 0}%` }"></div></div></aside>
      </section>
      <form class="add-form" @submit.prevent="addBook">
        <div><label for="book-title">Book title</label><input id="book-title" ref="titleInput" v-model="title" maxlength="160" placeholder="What would you like to read?" required></div>
        <div><label for="book-author">Author <span>(optional)</span></label><input id="book-author" v-model="author" maxlength="100" placeholder="Who wrote it?"></div>
        <button class="primary" type="submit">Add to shelf <span aria-hidden="true">+</span></button>
      </form>
      <p class="status" role="status" aria-live="polite">{{ message }}</p>
      <section aria-labelledby="shelf-title">
        <div class="shelf-heading"><h2 id="shelf-title">On your shelf</h2><label class="search"><span class="sr-only">Search title or author</span><input v-model="query" type="search" placeholder="Search your books"></label></div>
        <div class="filters" role="group" aria-label="Filter books"><button v-for="item in filters" :key="item.value" type="button" :aria-pressed="filter === item.value" @click="filter = item.value">{{ item.label }}</button></div>
        <ul v-if="visibleBooks.length" class="books">
          <li v-for="book in visibleBooks" :key="book.id" :class="{ finished: book.read }">
            <label class="book-check"><input v-model="book.read" type="checkbox" :aria-label="`Finished reading: ${book.title}`"><span class="book-spine" aria-hidden="true">{{ book.title.slice(0, 1).toUpperCase() }}</span></label>
            <div class="book-info"><h3>{{ book.title }}</h3><p>{{ book.author || 'Author not specified' }}</p></div>
            <span class="book-state">{{ book.read ? 'Finished' : 'To read' }}</span>
            <button class="remove" type="button" :aria-label="`Remove ${book.title}`" @click="removeBook(book.id)">Remove</button>
          </li>
        </ul>
        <div v-else class="empty"><h3>{{ books.length ? 'No matching books.' : 'Your shelf is ready.' }}</h3><p>{{ books.length ? 'Try another search or filter.' : 'Add a title above to get started.' }}</p></div>
      </section>
    </main>
    <footer><span>Saved on this browser · No account needed</span><a href="https://github.com/takici30-sudo/vue-reading-list">View source ↗</a></footer>
  </div>
</template>

<style>
:root { color-scheme: light; font-family: "Trebuchet MS", sans-serif; color: #332a22; background: #f4f0e7; --ink: #332a22; --muted: #776d61; --line: #d7cfbf; --accent: #71553a; }
* { box-sizing: border-box; }
body { margin: 0; }
button, input { font: inherit; }
button { cursor: pointer; }
a { color: inherit; text-underline-offset: 4px; }
button:focus-visible, input:focus-visible, a:focus-visible { outline: 3px solid #886437; outline-offset: 4px; }
.page { max-width: 1060px; margin: auto; padding: 0 36px; }
header, footer { display: flex; justify-content: space-between; align-items: center; gap: 20px; padding: 25px 0; }
header { border-bottom: 1px solid var(--line); }
.brand { font: bold 25px Georgia, serif; letter-spacing: -.8px; text-decoration: none; }
.brand span { color: var(--accent); margin-right: 10px; }
.edition, footer { font-size: 12px; color: var(--muted); }
.intro { display: grid; grid-template-columns: 1fr 240px; align-items: end; gap: 40px; padding: 58px 0 38px; }
.eyebrow { text-transform: uppercase; letter-spacing: .14em; color: var(--muted); font-size: 11px; }
h1 { font: normal clamp(44px, 6vw, 72px)/1.04 Georgia, serif; letter-spacing: -.055em; margin: 20px 0; }
.lede { color: var(--muted); font-size: 16px; line-height: 1.7; }
.reading-count { padding: 26px 0; border-top: 1px solid var(--ink); }
.reading-count strong { font: normal 58px Georgia, serif; }
.reading-count strong span { font-size: 24px; color: var(--muted); margin-left: 12px; }
.reading-count p { color: var(--muted); font-size: 12px; }
.progress { background: var(--line); height: 4px; margin-top: 22px; }
.progress div { height: 100%; background: var(--accent); }
.add-form { display: grid; grid-template-columns: 1.5fr 1fr auto; align-items: end; gap: 14px; padding: 24px; background: #eae3d5; border-radius: 8px; }
label { font-size: 12px; }
label span { color: var(--muted); }
input:not([type="checkbox"]) { display: block; width: 100%; min-width: 0; background: #fffdf8; border: 1px solid var(--line); border-radius: 5px; padding: 12px; margin-top: 8px; color: var(--ink); }
.primary { background: var(--ink); color: #fffdf8; border: 1px solid var(--ink); padding: 12px 20px; border-radius: 5px; white-space: nowrap; }
.primary span { margin-left: 12px; }
.status { min-height: 20px; font-size: 12px; color: var(--muted); margin: 14px 0 28px; }
.shelf-heading { display: flex; justify-content: space-between; align-items: center; gap: 20px; }
h2 { font: normal 28px Georgia, serif; letter-spacing: -.03em; }
.search { width: 230px; }
.filters { display: flex; gap: 8px; padding-bottom: 24px; }
.filters button { border: 1px solid var(--line); border-radius: 20px; padding: 7px 16px; background: transparent; color: var(--muted); font-size: 12px; }
.filters button[aria-pressed="true"] { background: var(--ink); border-color: var(--ink); color: #fffdf8; }
.books { list-style: none; padding: 0; margin: 0; border-top: 1px solid var(--line); }
.books li { display: flex; gap: 18px; align-items: center; padding: 20px 0; border-bottom: 1px solid var(--line); }
.book-check { display: flex; align-items: center; gap: 16px; }
.book-check input { accent-color: var(--accent); width: 17px; height: 17px; }
.book-spine { width: 40px; height: 53px; display: grid; place-items: center; background: #b8c1a6; color: #33432e; border-radius: 2px 5px 5px 2px; border-left: 5px solid #8d9e78; font: 22px Georgia, serif; }
.books li:nth-child(3n + 2) .book-spine { background: #dfc0a5; border-color: #ba9475; color: #694b34; }
.books li:nth-child(3n) .book-spine { background: #b0bccb; border-color: #8999b0; color: #334459; }
.book-info { flex: 1; min-width: 0; }
h3 { font-size: 16px; margin: 0 0 8px; overflow-wrap: anywhere; }
.book-info p { font-size: 12px; color: var(--muted); margin: 0; overflow-wrap: anywhere; }
.book-state { font-size: 11px; color: var(--muted); white-space: nowrap; }
.finished .book-state { color: #4c623c; }
.remove { background: transparent; color: var(--muted); border: 0; padding: 8px; font-size: 11px; }
.remove:hover { color: #9a352c; }
.empty { padding: 52px 20px; text-align: center; border: 1px dashed var(--line); border-radius: 8px; }
.empty p { color: var(--muted); font-size: 14px; }
footer { margin-top: 34px; padding-bottom: 30px; flex-wrap: wrap; }
.sr-only { position: absolute; width: 1px; height: 1px; padding: 0; margin: -1px; overflow: hidden; clip-path: inset(50%); white-space: nowrap; border: 0; }
@media (max-width: 720px) { .page { padding: 0 22px; } .intro { grid-template-columns: 1fr; padding-top: 38px; gap: 12px; } .reading-count { display: none; } .add-form { grid-template-columns: 1fr; } .edition { display: none; } .shelf-heading { flex-wrap: wrap; gap: 0; } .search { width: 100%; margin-bottom: 20px; } .book-state { display: none; } .books li { gap: 12px; } .book-check { gap: 10px; } }
</style>
