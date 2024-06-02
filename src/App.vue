<template>
  <q-layout view="hHh lpR fFf">
    <q-header elevated class="bg-pink-9 text-white">
      <q-toolbar class="bg-soft-pink">
        <q-btn flat round dense icon="menu" @click="toggleLeftDrawer" />
        <q-toolbar-title>
          <q-avatar class="profile">
            <img src="https://i.pinimg.com/564x/23/c3/87/23c387869c33a8fd31e9c8eea1009797.jpg" alt="Logo" size="45px" />
          </q-avatar>
          Mita's Feedly
        </q-toolbar-title>
        <q-space />
        <q-input
          filled
          dense
          placeholder="Cari Artikel"
          prepend-icon="search"
          class="bg-white q-ma-none"
        />
      </q-toolbar>
    </q-header>

    <q-drawer v-model="leftDrawerOpen" side="left" class="bg-soft-pink" :overlay="false">
      <q-list>
        <q-item clickable v-ripple @click="navigateTo('Postingan')">
          <q-item-section>
            <q-btn flat style="color:black" label="Postingan" />
          </q-item-section>
        </q-item>
        <q-item clickable v-ripple @click="navigateTo('Tema')">
          <q-item-section>
            <q-btn flat style="color:black;" label="Tema" />
          </q-item-section>
        </q-item>
        <q-item clickable v-ripple @click="navigateTo('Daftar Bacaan')">
          <q-item-section>
            <q-btn flat style="color:black;" label="Daftar Bacaan" />
          </q-item-section>
        </q-item>
        <q-item clickable v-ripple @click="navigateTo('login')">
          <q-item-section>
            <q-btn style="background: #ff69b4; color: white" label="Login" />
          </q-item-section>
        </q-item>
      </q-list>
    </q-drawer>

    <q-page-container>
      <q-page class="container q-pa-md">
        <q-card class="q-pa-md q-mb-md rounded-borders">
          <q-card-section>
            <q-form @submit.prevent="save">
              <q-input
                v-model="form.title"
                label="Judul"
                outlined
                dense
                class="q-mb-md"
                :rules="[val => !!val || 'Judul diperlukan']"
              />
              <q-input
                v-model="form.content"
                label="Konten"
                type="textarea"
                outlined
                dense
                class="q-mb-md"
                :rules="[val => !!val || 'Konten diperlukan']"
              />
              <q-btn type="submit" label="Simpan" color="pink" class="q-mb-md rounded-borders" />
            </q-form>
          </q-card-section>
        </q-card>

        <q-card class="q-pa-md q-mb-md rounded-borders">
          <q-card-section>
            <q-list bordered padding>
              <q-item
                v-for="article in articles"
                :key="article.id"
                clickable
                class="q-mb-xs rounded-borders q-gutter-xs"
              >
                <q-item-section>
                  <q-item-label>{{ article.title }}</q-item-label>
                  <q-item-label caption>{{ article.content }}</q-item-label>
                </q-item-section>
                <q-item-section side>
                  <q-btn dense round color="pink" icon="edit" @click.stop="edit(article)" />
                  <q-btn dense round color="negative" icon="delete" @click.stop="remove(article.id)" />
                </q-item-section>
              </q-item>
            </q-list>
          </q-card-section>
        </q-card>

        <q-btn label="Muat Artikel" color="pink" @click="load" class="rounded-borders" />
      </q-page>
    </q-page-container>

    <q-footer class="footer bg-pink-9 text-white">
      <div class="footer-content">
        <p>Miftahul Jannah | 223510890</p>
      </div>
    </q-footer>
  </q-layout>
</template>

<script setup>
import { reactive, ref, onMounted } from 'vue';
import axios from 'axios';

// JSONBin API details
const JSONBIN_API_KEY = '$2a$10$XVTGLAfQa.7V85WP/L9GtObAB8BdXzLtaGmmqvq7LvynjKwzvLuPe';
const BIN_ID = '6659f4f5acd3cb34a850b41a';

const leftDrawerOpen = ref(false);

const form = reactive({
  id: null,
  title: '',
  content: '',
});

const articles = ref([]);

async function load() {
  try {
    const response = await axios.get(`https://api.jsonbin.io/v3/b/${BIN_ID}/latest`, {
      headers: {
        'X-Master-Key': JSONBIN_API_KEY,
      },
    });
    articles.value = response.data.record.articles || [];
  } catch (error) {
    console.error('Error loading articles', error);
  }
}

async function save() {
  try {
    let updatedArticles;
    if (form.id) {
      // Update the existing article
      updatedArticles = articles.value.map((article) =>
        article.id === form.id ? { ...form } : article
      );
    } else {
      // Create a new article
      const newArticle = { ...form, id: Date.now() };
      updatedArticles = [...articles.value, newArticle];
    }
    await axios.put(
      `https://api.jsonbin.io/v3/b/${BIN_ID}`,
      { articles: updatedArticles },
      {
        headers: {
          'Content-Type': 'application/json',
          'X-Master-Key': JSONBIN_API_KEY,
        },
      }
    );
    form.id = null;
    form.title = '';
    form.content = '';
    await load();
  } catch (error) {
    console.error('Error saving article', error);
  }
}

function edit(article) {
  form.id = article.id;
  form.title = article.title;
  form.content = article.content;
}

async function remove(id) {
  try {
    const updatedArticles = articles.value.filter((article) => article.id !== id);
    await axios.put(
      `https://api.jsonbin.io/v3/b/${BIN_ID}`,
      { articles: updatedArticles },
      {
        headers: {
          'Content-Type': 'application/json',
          'X-Master-Key': JSONBIN_API_KEY,
        },
      }
    );
    await load();
  } catch (error) {
    console.error('Error deleting article', error);
  }
}

function toggleLeftDrawer() {
  leftDrawerOpen.value = !leftDrawerOpen.value;
}

function navigateTo(page) {
  console.log(`Navigating to ${page}`);
}

onMounted(load);
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: auto;
}

.footer {
  background-color: #FFB6C1;
  color: white;
  width: 100%;
  font-size: medium;
  padding: 10px;
  text-align: center;
  position: relative;
  bottom: 0;
}

.footer-content {
  margin: auto;
}

.profile {
  border-radius: 50%;
  overflow: hidden;
}

.q-card {
  margin-bottom: 20px;
}

.bg-pink-9 {
  background-color: #FFB6C1 !important;
}

.bg-soft-pink {
  background-color: #FFB6C1 !important;
}

.rounded-borders {
  border-radius: 15px;
}

.q-btn {
  margin-left: 5px;
  margin-right: 5px;
}

.q-page-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.q-page {
  flex: 1;
}
</style>
