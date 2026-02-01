<template>
  <header>
    <div class="topbar">
      <nav aria-label="Navegación">
        <a href="#p1" data-nav="p1" :class="{ active: activePage === 'p1' }"
          >Inicio</a
        >
        <a href="#p3" data-nav="p3" :class="{ active: activePage === 'p3' }"
          >Detalles</a
        >
        <a href="#p4" data-nav="p4" :class="{ active: activePage === 'p4' }"
          >Ubicación</a
        >
      </nav>
    </div>
  </header>

  <main ref="scrollerRef" id="scroller" aria-label="Invitación scrolleable">
    <!-- PÁGINA 1 -->
    <section class="page" id="p1" data-page>
      <div class="diag" aria-hidden="true">
        <img class="tlbr" :src="diagonal1" alt="" />
        <img class="trbl" :src="diagonal2" alt="" />
      </div>

      <article class="card">
        <div class="card-inner">
          <div class="grid">
            <div>
              <div class="typewriter">
                <h1 ref="typewriterRef" class="tw"></h1>
              </div>

              <div class="image-center">
                <img id="prometidos" :src="prometidos" alt="Decoración" />
              </div>

              <div class="typewriter">
                <h2 ref="typewriterNameRef" class="cursive"></h2>
              </div>

              <div class="panel" aria-label="Cuenta regresiva">
                <div class="statline">
                  <div class="stat">
                    <b>{{ d }}</b
                    ><span>Días</span>
                  </div>
                  <div class="stat">
                    <b>{{ h }}</b
                    ><span>Horas</span>
                  </div>
                  <div class="stat">
                    <b>{{ m }}</b
                    ><span>Min</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </article>
    </section>

    <!-- PÁGINA 3 (stack) -->
    <section class="page stack" id="p3" data-page>
      <div class="stack-wrap">
        <article class="card">
          <div class="card-inner" style="text-align: center">
            <h2 ref="verseRef" class="fade-verse"></h2>
          </div>
        </article>

        <article class="card">
          <div class="card-inner">
            <h2 style="text-align: center">
              Con la bendición de nuestros padres
            </h2>
            <div class="divider"></div>

            <div
              class="grid"
              style="grid-template-columns: 1fr 1fr; text-align: center"
            >
              <div class="panel">
                <p class="miniTitle">Padres de la Novia</p>
                <p style="margin: 4px 0 0">José Cazarin Linarez</p>
                <p style="margin: 4px 0 0">&</p>
                <p style="margin: 4px 0 0">Candelaria Parada Domínguez</p>
              </div>

              <div class="panel">
                <p class="miniTitle">Padres del Novio</p>
                <p style="margin: 4px 0 0">Nicolás Nieves Heredia</p>
                <p style="margin: 4px 0 0">&</p>
                <p style="margin: 4px 0 0">María Isabel Santos Prieto</p>
              </div>
            </div>

            <div class="divider"></div>
            <p style="text-align: center; font-style: italic">
              Nos han guiado con amor, hoy nos acompañan a comenzar nuestra
              propia historia.
            </p>
          </div>
        </article>
      </div>
    </section>

    <!-- PÁGINA 4 -->
    <section class="page stack" id="p4" data-page>
      <div class="stack-wrap">
        <article class="card">
          <div class="card-inner invite4">
            <h2 class="invite-title">Ubicación</h2>
            <p class="invite-sub">
              Acompáñanos a celebrar el inicio de nuestra vida juntos
            </p>

            <div class="invite-divider"></div>

            <div class="invite-dateRow" aria-label="Fecha y hora">
              <div class="invite-side">
                <div class="invite-line"></div>
                <div class="invite-small">SÁBADO</div>
                <div class="invite-line"></div>
              </div>

              <div class="invite-center">
                <div class="invite-month">FEBRERO</div>
                <div class="invite-day">28</div>
                <div class="invite-year">2026</div>
              </div>

              <div class="invite-side">
                <div class="invite-line"></div>
                <div class="invite-small">5:00 PM</div>
                <div class="invite-line"></div>
              </div>
            </div>

            <div class="invite-divider"></div>

            <div class="invite-details">
              <div class="invite-pill">
                <span class="k">Lugar</span>
                <span class="v">{{ address }}</span>
              </div>
            </div>
          </div>
        </article>
      </div>
    </section>
  </main>

  <div class="fab" aria-label="Acciones rápidas">
    <a class="chip" href="#p1" title="Volver arriba">Arriba</a>
  </div>

  <div class="toast" ref="toastRef" role="status" aria-live="polite"></div>
</template>

<script setup>
import { onMounted, onBeforeUnmount, ref } from "vue";

// ✅ Imágenes (ponlas en src/assets/img/)
import diagonal1 from "@/assets/img/diagonal-1.png";
import diagonal2 from "@/assets/img/diagonal-2.png";
import prometidos from "@/assets/img/foto-prometidos-1.png";
import teEsperamos from "@/assets/img/Te-esperamos.png";

// ===== Config =====
const eventStartISO = "2026-02-28T17:00:00-06:00";
const address =
  "Calle Prolongación Miguel Alemán SN, Colonia El Cerrito, Hueyapan de Ocampo, Ver.";

// ===== Refs DOM =====
const scrollerRef = ref(null);
const typewriterRef = ref(null);
const typewriterNameRef = ref(null);
const verseRef = ref(null);
const toastRef = ref(null);

// ===== Estado =====
const activePage = ref("p1");
const d = ref("—");
const h = ref("—");
const m = ref("—");

let countdownTimer = null;
let io = null;

// ===== Toast =====
const showToast = (msg) => {
  const el = toastRef.value;
  if (!el) return;
  el.textContent = msg;
  el.classList.add("show");
  clearTimeout(showToast._t);
  showToast._t = setTimeout(() => el.classList.remove("show"), 2200);
};

// ===== Typewriter =====
const typeText = "Nuestra boda";
const typeName = "Rocío & Nicolás";

const runTypewriter = () => {
  const target = typewriterRef.value;
  const target2 = typewriterNameRef.value;
  if (!target || !target2) return;

  target.innerHTML = "";
  target2.textContent = "";

  let i = 0;
  let i2 = 0;

  const type = () => {
    if (i < typeText.length) {
      const span = document.createElement("span");
      span.textContent = typeText[i] === " " ? "\u00A0" : typeText[i];
      span.style.animationDelay = `${i * 0.04}s`;
      target.appendChild(span);
      i++;
      setTimeout(type, 60);
    } else {
      setTimeout(type2Fn, 300);
    }
  };

  const type2Fn = () => {
    if (i2 < typeName.length) {
      target2.textContent += typeName.charAt(i2++);
      setTimeout(type2Fn, 50);
    }
  };

  type();
};

// ===== Verse (fade letter) =====
const verseText =
  "El amor es paciente, es bondadoso. El amor nunca deja de ser.\n— 1 Corintios 13:4,8";

const typeVerseWithFade = (text, element) => {
  element.innerHTML = "";
  let delay = 0;

  const lines = text.split("\n");
  lines.forEach((line, lineIdx) => {
    const words = line.split(" ");

    words.forEach((word, wIdx) => {
      const w = document.createElement("span");
      w.className = "word";

      [...word].forEach((ch) => {
        const s = document.createElement("span");
        s.className = "letter";
        s.textContent = ch;
        s.style.animationDelay = `${delay}s`;
        delay += 0.03;
        w.appendChild(s);
      });

      element.appendChild(w);
      if (wIdx < words.length - 1)
        element.appendChild(document.createTextNode(" "));
    });

    if (lineIdx < lines.length - 1)
      element.appendChild(document.createElement("br"));
  });
};

// ===== Nav Observer =====
const setupNavObserver = () => {
  const root = scrollerRef.value;
  if (!root) return;

  const pages = [...document.querySelectorAll("[data-page]")];

  io = new IntersectionObserver(
    (entries) => {
      const visible = entries
        .filter((e) => e.isIntersecting)
        .sort((a, b) => b.intersectionRatio - a.intersectionRatio)[0];

      if (!visible) return;

      activePage.value = visible.target.id;

      if (
        visible.target.id === "p3" &&
        verseRef.value &&
        !verseRef.value.dataset.loaded
      ) {
        typeVerseWithFade(verseText, verseRef.value);
        verseRef.value.dataset.loaded = "true";
      }
    },
    { root, threshold: [0.55, 0.65, 0.75] },
  );

  pages.forEach((p) => io.observe(p));
};

// ===== Countdown =====
const formatCountdown = () => {
  const end = new Date(eventStartISO);
  const now = new Date();
  let diff = end - now;

  if (diff <= 0) {
    d.value = "0";
    h.value = "0";
    m.value = "0";
    return;
  }

  const totalMinutes = Math.floor(diff / 60000);
  const days = Math.floor(totalMinutes / (60 * 24));
  const hours = Math.floor((totalMinutes % (60 * 24)) / 60);
  const minutes = totalMinutes % 60;

  d.value = String(days);
  h.value = String(hours).padStart(2, "0");
  m.value = String(minutes).padStart(2, "0");
};

// ===== Lifecycle =====
onMounted(() => {
  runTypewriter();
  setupNavObserver();

  formatCountdown();
  countdownTimer = setInterval(formatCountdown, 1000 * 30);
});

onBeforeUnmount(() => {
  if (io) io.disconnect();
  if (countdownTimer) clearInterval(countdownTimer);
});
</script>

<style scoped>
/* ============ Header / Nav ============ */
header {
  position: fixed;
  inset: 0 0 auto 0;
  z-index: 50;
  padding: 12px 14px;
  backdrop-filter: blur(14px);
  background: linear-gradient(180deg, rgba(0, 0, 0, 0.55), rgba(0, 0, 0, 0.1));
  border-bottom: 1px;
}

.topbar {
  max-width: var(--maxw);
  margin: 0 auto;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
}

nav {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  justify-content: flex-end;
}

nav a {
  text-decoration: none;
  color: rgba(255, 255, 255, 0.8);
  border: 1px solid rgba(255, 255, 255, 0.14);
  background: rgba(255, 255, 255, 0.06);
  padding: 8px 10px;
  border-radius: 999px;
  font-size: 12px;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  transition: 0.2s ease;
}

nav a:hover {
  transform: translateY(-1px);
  background: rgba(255, 255, 255, 0.1);
  border-color: rgba(255, 255, 255, 0.22);
}

nav a.active {
  color: rgba(0, 0, 0, 0.88);
  background: linear-gradient(
    90deg,
    rgba(217, 178, 110, 0.95),
    rgba(217, 178, 110, 0.55)
  );
  border-color: rgba(217, 178, 110, 0.55);
}

/* ============ App / Pages ============ */
#app {
  height: var(--h);
  overflow-y: auto;
  scroll-snap-type: y mandatory;
  scroll-behavior: smooth;
}

#scroller {
  height: 100dvh; /* o 100vh */
  overflow-y: auto;
  scroll-snap-type: y mandatory;
  scroll-behavior: smooth;

  /* para que el header fijo no tape el inicio al hacer click en anchors */
  scroll-padding-top: 72px; /* ajusta a tu header real */
}
.page {
  scroll-snap-align: start;
  padding: calc(36% + 16px) var(--pad) var(--pad);
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: var(--h);
}

/* página con varios cards apilados */
.page.stack {
  align-items: flex-start;
  justify-content: flex-start;
}

/* contenedor que apila */
.stack-wrap {
  width: min(100%, var(--maxw));
  display: flex;
  flex-direction: column;
  gap: 18px;
}

/* Ajustes específicos por página (si de verdad los quieres) */
#p1 {
  padding-bottom: 20%;
}

#p3 {
  padding-top: 28%;
}

#p4 {
  padding-top: 25%;
}

/* ============ Card ============ */
.card {
  width: min(100%, var(--maxw));
  border-radius: var(--radius);
  background: linear-gradient(
    180deg,
    rgba(255, 255, 255, 0.1),
    rgba(255, 255, 255, 0.06)
  );
  border: 1px solid rgba(255, 255, 255, 0.14);
  box-shadow: var(--shadow);
  overflow: hidden;
  position: relative;
  z-index: 2;
}

.card::before {
  content: "";
  position: absolute;
  inset: -2px;
  background:
    radial-gradient(
      600px 400px at 15% 0%,
      rgba(217, 178, 110, 0.22),
      transparent 55%
    ),
    radial-gradient(
      700px 500px at 100% 50%,
      rgba(126, 224, 210, 0.18),
      transparent 60%
    );
  pointer-events: none;
  opacity: 0.85;
}

.card-inner {
  position: relative;
  z-index: 2;
  padding: clamp(18px, 3.2vw, 34px);
}

/* ============ Diagonales (fondo en p1) ============ */
.diag {
  position: absolute;
  inset: 0;
  pointer-events: none;
  z-index: 0;
}

.diag img {
  position: absolute;
  width: 71%;
  max-width: none;
  height: auto;
  filter: blur(1px);
}

#prometidos {
  display: block;
  margin: 0 auto;
  max-width: 320px;
  width: 60vw;
  height: auto;
  border-radius: 18px;
}

#Te-esperamos {
  display: block;
  margin: 0 auto;
  width: 40%;
  height: auto;
  border-radius: 18px;
}

.diag .tlbr {
  top: 6%;
  left: 0;
}

.diag .trbl {
  top: 61%;
  right: 0;
}

/* ============ Typography / Layout ============ */
h2 {
  font-family: "Playfair Display", serif;
  margin: 14px 0 8px;
  font-size: clamp(28px, 4.2vw, 56px);
  line-height: 1.02;
  letter-spacing: -0.02em;
}

p {
  margin: 0 0 12px;
  color: var(--muted);
  font-size: clamp(14px, 1.8vw, 18px);
  line-height: 1.55;
}

.cursive {
  font-family: "Great Vibes", cursive;
  font-weight: normal;
  /* esta fuente solo tiene un peso */
  letter-spacing: 0.02em;
  text-align: center;
}

.grid {
  display: grid;
  grid-template-columns: 1.15fr 0.85fr;
  gap: 18px;
  align-items: stretch;
}

#typewriter {
  font-family: "Playfair Display", serif;
  font-size: clamp(34px, 5vw, 64px);
  line-height: 1.02;
  letter-spacing: -0.02em;
  margin: 0 0 10px;
  text-align: center;
  font-weight: 800;
}

#typewriterName {
  font-family: "Playfair Display", serif;
  font-size: clamp(34px, 5vw, 64px);
  line-height: 1.02;
  letter-spacing: -0.02em;
  margin: 0 0 10px;
  text-align: center;
  font-weight: 800;
}

#typewriter span {
  color: var(--accent);
  text-shadow: 0 10px 30px rgba(0, 0, 0, 0.25);
}

.panel {
  border-radius: 18px;
  border: 1px solid rgba(255, 255, 255, 0.12);
  background: rgba(0, 0, 0, 0.18);
  padding: 16px;
}

.divider {
  height: 1px;
  background: rgba(255, 255, 255, 0.14);
  margin: 16px 0;
}

/* ============ Countdown stats ============ */
.statline {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
  margin-top: 10px;
}

.stat {
  text-align: center;
}

.stat b {
  display: block;
  font-size: clamp(22px, 3.2vw, 34px);
  color: rgba(3, 3, 3, 0.95);
}

.stat span {
  display: block;
  margin-top: 4px;
  font-size: 12px;
  letter-spacing: 0.14em;
  text-transform: uppercase;
  color: rgba(1, 1, 1, 0.72);
}

/* ============ Verse animation ============ */
.fade-verse {
  text-align: center;
  line-height: 1.3;
  font-size: clamp(22px, 3.4vw, 46px);
  margin: 0;
  max-width: 32ch;
  margin-inline: auto;
  font-family: "Playfair Display", serif;
  font-style: italic;
  color: #2f5d9f;
}

.fade-verse .word {
  display: inline-block;
  white-space: nowrap;
}

.fade-verse .letter {
  opacity: 0;
  display: inline-block;
  transform: translateY(10px);
  animation: fadeLetter 0.6s forwards;
}

@keyframes fadeLetter {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* ============ Invite (p4) ============ */
.invite4 {
  text-align: center;
}

.invite-title {
  margin-top: 0;
}

.invite-sub {
  max-width: 48ch;
  margin: 0 auto 8px;
}

.invite-divider {
  height: 1px;
  background: rgba(255, 255, 255, 0.16);
  margin: 16px auto;
  max-width: 560px;
}

.invite-dateRow {
  display: grid;
  grid-template-columns: 1fr auto 1fr;
  gap: 14px;
  align-items: center;
  max-width: 720px;
  margin: 0 auto;
}

.invite-side {
  display: grid;
  grid-template-columns: 1fr;
  gap: 10px;
  align-items: center;
}

.invite-line {
  height: 1px;
  background: rgba(217, 178, 110, 0.55);
  opacity: 0.9;
}

.invite-small {
  letter-spacing: 0.12em;
  text-transform: uppercase;
  font-size: 12px;
  color: rgba(0, 0, 0, 0.72);
  font-weight: 650;
}

.invite-center {
  padding: 6px 10px;
  border-radius: 18px;
  border: 1px solid rgba(217, 178, 110, 0.35);
  background: rgba(255, 255, 255, 0.1);
  min-width: 170px;
}

.invite-month {
  letter-spacing: 0.14em;
  text-transform: uppercase;
  font-size: 14px;
  color: rgba(0, 0, 0, 0.72);
  font-weight: 700;
}

.invite-day {
  font-size: clamp(44px, 6.4vw, 72px);
  line-height: 1;
  margin: 4px 0;
  color: rgba(0, 0, 0, 0.9);
  font-weight: 800;
}

.invite-year {
  letter-spacing: 0.12em;
  text-transform: uppercase;
  font-size: 12px;
  color: rgba(0, 0, 0, 0.7);
  font-weight: 700;
}

.invite-details {
  display: grid;
  gap: 10px;
  max-width: 720px;
  margin: 0 auto;
}

.invite-pill {
  display: flex;
  justify-content: space-between;
  gap: 12px;
  padding: 12px 14px;
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.14);
  background: rgba(0, 0, 0, 0.1);
}

.invite-pill .k {
  letter-spacing: 0.12em;
  text-transform: uppercase;
  font-size: 12px;
  color: rgba(0, 0, 0, 0.65);
  font-weight: 750;
}

.invite-pill .v {
  color: rgba(0, 0, 0, 0.88);
  font-weight: 650;
}

/* ============ Typewriter spans ============ */
.tw {
  font-family: "Great Vibes", cursive;
  font-weight: normal;
  /* esta fuente solo tiene un peso */
  letter-spacing: 0.02em;
  display: block;
  text-align: center;
  color: #d9b26e;
  font-style: italic;
}

.tw span {
  opacity: 0;
  display: inline-block;
  transform: translateY(10px);
  animation: twIn 0.55s forwards;
}

@keyframes twIn {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* ============ Floating UI ============ */
.toast {
  position: fixed;
  inset: auto 16px 16px auto;
  z-index: 60;
  max-width: 360px;
  padding: 12px 14px;
  border-radius: 14px;
  border: 1px solid rgba(255, 255, 255, 0.16);
  background: rgba(0, 0, 0, 0.55);
  backdrop-filter: blur(14px);
  color: rgba(255, 255, 255, 0.92);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.45);
  transform: translateY(20px);
  opacity: 0;
  pointer-events: none;
  transition: 0.25s ease;
}

.toast.show {
  transform: translateY(0);
  opacity: 1;
}

.fab {
  position: fixed;
  inset: auto 16px 16px auto;
  z-index: 55;
  display: flex;
  flex-direction: column;
  gap: 10px;
  align-items: flex-end;
}

.chip {
  padding: 10px 12px;
  border-radius: 999px;
  border: 1px solid rgba(255, 255, 255, 0.16);
  background: rgba(255, 255, 255, 0.1);
  color: rgba(255, 255, 255, 0.92);
  text-decoration: none;
  font-size: 13px;
  display: inline-flex;
  gap: 10px;
  align-items: center;
  transition: 0.2s ease;
  backdrop-filter: blur(12px);
}

.chip:hover {
  transform: translateY(-1px);
  background: rgba(255, 255, 255, 0.14);
}

/* ============ Accessibility ============ */
:focus-visible {
  outline: 2px solid rgba(217, 178, 110, 0.75);
  outline-offset: 3px;
  border-radius: 12px;
}

/* ============ Responsive ============ */
@media (max-width: 860px) {
  .grid {
    grid-template-columns: 1fr;
  }

  nav {
    justify-content: center;
  }

  header {
    padding-bottom: 14px;
  }

  .card .diag img {
    opacity: 0.14;
  }
}

@media (min-width: 861px) {
  .page {
    padding-top: calc(22% + 16px);
  }

  .page.stack {
    align-items: center;
  }

  #p1 {
    padding-bottom: 20%;
  }

  #p3 {
    padding-top: 20%;
  }

  #p4 {
    padding-top: 18%;
  }

  .stack-wrap {
    margin: 0 auto;
    gap: 24px;
  }
}

/* ============ Reduced motion ============ */
@media (prefers-reduced-motion: reduce) {
  #app {
    scroll-behavior: auto;
  }

  nav a,
  .btn,
  .chip {
    transition: none;
  }
}
</style>
