<script setup>
import { useRoute } from 'vue-router'
import { games } from '@/data/games'

const route = useRoute()

function closeMenu(event) {
  event.currentTarget.blur()
}
</script>

<template>
  <header class="header">
    <RouterLink to="/" class="logo" aria-label="Mass Effect – accueil">
    <img src="/images/masseffect.svg" alt="Mass Effect N7" />
    </RouterLink>

    <nav aria-label="Navigation principale">
      <ul>
        <li class="has-menu">
          <button type="button" aria-haspopup="true" :class="{ active: route.path.startsWith('/jeux') }">
            Games
          </button>
          <ul class="menu">
            <li v-for="(game, i) in games" :key="game.id" :style="{ '--i': i }">
              <RouterLink :to="`/jeux/${game.id}`" @click="closeMenu">{{ game.title }}</RouterLink>
            </li>
          </ul>
        </li>
        <li>
          <RouterLink to="/bioware" active-class="active">BioWare</RouterLink>
        </li>
      </ul>
    </nav>
  </header>
</template>

<style scoped>
.header {
  position: sticky;
  top: 0;
  z-index: 20;
  height: var(--header-h);
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 clamp(1rem, 4vw, 3rem);
  background: rgba(4, 3, 17, 0.9);
  backdrop-filter: blur(6px);
}
.logo {
  display: flex;
  align-items: center;
  height: 100%;
}
.logo img { height: 1.8rem; }

.logo .n7 { font-size: 0.7rem; opacity: 0.85; }
.logo b { color: var(--red-bright); }

nav > ul { list-style: none; margin: 0; padding: 0; display: flex; gap: 1.8rem; align-items: center; }
nav a,
nav button {
  background: none;
  border: 0;
  cursor: pointer;
  font: 600 0.95rem var(--font-body);
  color: var(--text);
  text-decoration: none;
  padding: 0.4rem 0;
}
nav .active { color: var(--red-bright); }
nav a:hover, nav button:hover { color: var(--red-bright); }

.has-menu { position: relative; }
.menu {
  display: none;
  position: absolute;
  top: 100%;
  right: 0;
  min-width: 12rem;
  list-style: none;
  margin: 0;
  padding: 0.4rem 0;
}
.has-menu:hover .menu,
.has-menu:focus-within .menu { display: block; }

.menu li { padding-left: calc(var(--i) * 0.9rem); font-size: 0.85rem; }
.menu li a { display: inline-flex; align-items: center; gap: 0.5rem; }
.menu li a::before {
  content: '';
  width: 0.45rem;
  height: 0.45rem;
  background: var(--red-bright);
}

@media (max-width: 520px) {
  nav > ul { gap: 1rem; }
  .logo { font-size: 0.95rem; }
}
</style>