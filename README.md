<!doctype html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Family Board</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Fredoka:wght@500;600&family=Inter:wght@400;500;600&display=swap" rel="stylesheet">

<!-- Firebase (compat build — simplest to use without a bundler).
     Check firebase.google.com/docs/web/setup for the current version number
     and update it here if this one has aged out. -->
<script src="https://www.gstatic.com/firebasejs/10.13.2/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.13.2/firebase-firestore-compat.js"></script>

<style>
  :root {
    --bg: #26302B;
    --panel: #2F3B34;
    --ink: #F1ECDD;
    --ink-dim: #C9C2AE;
    --rule: rgba(241, 236, 221, 0.14);
    --accent: #E0A45C;
    --radius: 10px;
    --font-display: 'Fredoka', system-ui, sans-serif;
    --font-body: 'Inter', system-ui, sans-serif;
  }

  * { box-sizing: border-box; }

  html, body {
    margin: 0;
    height: 100%;
    background: var(--bg);
    color: var(--ink);
    font-family: var(--font-body);
    background-image: repeating-linear-gradient(0deg, transparent, transparent 39px, var(--rule) 40px);
  }

  body {
    padding: clamp(16px, 3vw, 32px);
    display: flex;
    flex-direction: column;
    gap: 20px;
    min-height: 100vh;
  }

  .board-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    flex-wrap: wrap;
    gap: 12px;
    border-bottom: 2px dashed var(--rule);
    padding-bottom: 14px;
  }

  #monthTitle {
    font-family: var(--font-display);
    font-weight: 600;
    font-size: clamp(24px, 3.4vw, 40px);
    margin: 0;
    letter-spacing: 0.01em;
  }

  .legend { display: flex; gap: 14px; flex-wrap: wrap; }
  .legend-item { display: flex; align-items: center; gap: 6px; font-size: 14px; color: var(--ink-dim); }
  .dot { width: 12px; height: 12px; border-radius: 50%; display: inline-block; }

  .ghost-btn {
    background: transparent;
    border: 1px solid var(--rule);
    color: var(--ink);
    padding: 8px 16px;
    border-radius: 999px;
    font-family: var(--font-body);
    font-size: 14px;
    cursor: pointer;
  }
  .ghost-btn:hover { border-color: var(--accent); color: var(--accent); }

  .week-grid {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    gap: 10px;
    flex: 1;
  }

  .day-col {
    background: var(--panel);
    border-radius: var(--radius);
    padding: 12px 10px;
    min-height: 220px;
    display: flex;
    flex-direction: column;
    gap: 8px;
  }
  .day-col.today { outline: 2px solid var(--accent); outline-offset: -2px; }

  .day-header {
    display: flex;
    flex-direction: column;
    border-bottom: 1px dashed var(--rule);
    padding-bottom: 6px;
    margin-bottom: 2px;
  }
  .dow { font-size: 12px; text-transform: uppercase; letter-spacing: 0.08em; color: var(--ink-dim); }
  .dnum { font-family: var(--font-display); font-size: 18px; font-weight: 600; }

  .no-events { color: var(--ink-dim); font-size: 13px; padding-top: 6px; }

  .event-chip {
    border-left: 3px solid transparent;
    border-radius: 6px;
    padding: 6px 8px;
    font-size: 13px;
    line-height: 1.3;
  }
  .event-chip.sample { opacity: 0.7; }
  .event-chip.shared { border-left-style: dashed; border-left-width: 2px; }
  .chip-dots { display: flex; gap: 4px; margin-bottom: 3px; }
  .chip-dot { width: 8px; height: 8px; border-radius: 50%; display: inline-block; }
  .chip-time { display: block; font-size: 11px; color: var(--ink-dim); }
  .chip-title { display: block; font-weight: 500; }

  .chores-panel {
    background: var(--panel);
    border-radius: var(--radius);
    padding: 18px 20px;
  }
  .chores-panel h2 { font-family: var(--font-display); margin: 0 0 12px; font-size: 20px; }

  .add-chore-row { display: flex; gap: 8px; margin-bottom: 14px; flex-wrap: wrap; }
  .add-chore-row input[type=text] {
    flex: 1; min-width: 160px; background: var(--bg); border: 1px solid var(--rule); color: var(--ink);
    padding: 8px 12px; border-radius: 8px; font-family: var(--font-body); font-size: 14px;
  }
  .add-chore-row select {
    background: var(--bg); color: var(--ink); border: 1px solid var(--rule); border-radius: 8px; padding: 8px;
  }
  .add-chore-row button {
    background: var(--accent); border: none; color: #26302B; font-weight: 600;
    padding: 8px 18px; border-radius: 8px; cursor: pointer;
  }

  .chore-list { list-style: none; margin: 0; padding: 0; display: flex; flex-direction: column; gap: 6px; }
  .chore-row { display: flex; align-items: center; gap: 10px; padding: 6px 4px; border-bottom: 1px dashed var(--rule); }
  .chore-row.done .chore-text { text-decoration: line-through; color: var(--ink-dim); }
  .chore-text { font-size: 15px; }
  .status-note { color: var(--ink-dim); font-size: 13px; margin: 10px 0 0; }

  @media (max-width: 900px) {
    .week-grid { grid-template-columns: repeat(auto-fit, minmax(120px, 1fr)); }
  }
  @media (max-width: 600px) {
    .week-grid { grid-template-columns: repeat(2, 1fr); }
  }
</style>
</head>
<body>

  <header class="board-header">
    <h1 id="monthTitle">Loading…</h1>
    <div class="legend" id="legend"></div>
    <button id="refreshBtn" class="ghost-btn" title="Refresh calendars">Refresh</button>
  </header>

  <main class="week-grid" id="weekGrid"></main>

  <section class="chores-panel">
    <h2>Chores</h2>
    <form id="addChoreForm" class="add-chore-row">
      <input id="newChoreText" type="text" placeholder="Add a chore…" autocomplete="off" />
      <select id="newChoreOwner"></select>
      <button type="submit">Add</button>
    </form>
    <ul id="choreList" class="chore-list"></ul>
    <p id="choresStatus" class="status-note"></p>
  </section>

<script>
  // ============================= CONFIG =============================
  // PEOPLE: everyone who can be assigned to events, with a display color.
  // "email" is optional — set it if you want this person to be addable to
  // individual events beyond a feed's default people (see FEEDS below).
  const CONFIG = {
    people: [
      { name: "Mom",   color: "#7FA5C4", email: "clwendt22@gmail.com" },
      { name: "Dad",   color: "#C97FA0", email: "jon.pedersen42@gmail.com" },
      { name: "Margot", color: "#7FBF8E", email: "" },
      { name: "Etta", color: "#E0A45C", email: "" },
    ],

    // FEEDS: each calendar you're pulling from. "defaultPeople" is who an
    // event on this feed is assumed to involve. If an event also invites
    // someone else (matched by the email you set above in `people`), that
    // person's color is ADDED to the event — you don't lose the default.
    feeds: [
      { name: "Mom", feedUrl: "", defaultPeople: ["Mom"] },
      { name: "Dad",
        feedUrl: "https://delicate-mountain-ff10.jon-pedersen42.workers.dev/?feed=" +
                 encodeURIComponent("https://p147-caldav.icloud.com/published/2/Mjc4NTM4MTU1Mjc4NTM4Mdjl2n5aQFRPNZBDYMWKqn3SDyhLYkvD_tQtBYx53egvnczEQPPUcMHvQKL68evRQP6IfOv_nLUgg-DJIQRV-eY"),
        defaultPeople: ["Dad"] },
      { name: "Margot",
        feedUrl: "https://delicate-mountain-ff10.jon-pedersen42.workers.dev/?feed=" +
                 encodeURIComponent("https://p147-caldav.icloud.com/published/2/Mjc4NTM4MTU1Mjc4NTM4Mdjl2n5aQFRPNZBDYMWKqn2SM5ytRqAqRzRTuLNPssW_dm9H4VCXNT27PZ1g7v6NOF7BvjhbJHp0abk9kQxJWtQ"),
        defaultPeople: ["Margot"] },
      { name: "Etta",
        feedUrl: "https://delicate-mountain-ff10.jon-pedersen42.workers.dev/?feed=" +
                 encodeURIComponent("https://p147-caldav.icloud.com/published/2/Mjc4NTM4MTU1Mjc4NTM4Mdjl2n5aQFRPNZBDYMWKqn36yrzzT1aWCFaJnYZuIkzOndEYHgDtiyw0e5xW1dmSl3RbR8bDRmP_0jWGH49Vyis"),
        defaultPeople: ["Etta"] },
    ],

    refreshIntervalMinutes: 15,
    firebase: {
      apiKey: "AIzaSyALDITAFOXMCk7VF12m18B3aFiow5wLjSY",
      authDomain: "family-planner-a4802.firebaseapp.com",
      projectId: "family-planner-a4802",
      storageBucket: "family-planner-a4802.firebasestorage.app",
      messagingSenderId: "917963378699",
      appId: "1:917963378699:web:0ef7297b24a3345cd8b70e"
    }
  };
  // ====================================================================

  const DAY_MS = 24 * 60 * 60 * 1000;

  function startOfDay(d) { const x = new Date(d); x.setHours(0, 0, 0, 0); return x; }
  function fmtDay(d) { return d.toLocaleDateString(undefined, { weekday: 'short' }); }
  function fmtDate(d) { return d.toLocaleDateString(undefined, { month: 'short', day: 'numeric' }); }
  function fmtTime(d) { return d.toLocaleTimeString(undefined, { hour: 'numeric', minute: '2-digit' }); }
  function escapeHTML(s) { const d = document.createElement('div'); d.textContent = s; return d.innerHTML; }

  function hexToRGBA(hex, alpha) {
    const h = hex.replace('#', '');
    const full = h.length === 3 ? h.split('').map(c => c + c).join('') : h;
    const n = parseInt(full, 16);
    const r = (n >> 16) & 255, g = (n >> 8) & 255, b = n & 255;
    return `rgba(${r}, ${g}, ${b}, ${alpha})`;
  }

  // ---------------------- minimal ICS parser ----------------------
  // Handles single, non-recurring VEVENTs (SUMMARY/DTSTART/DTEND), plus
  // ATTENDEE/ORGANIZER email addresses so multi-person events can be
  // detected. Recurring events (RRULE) are NOT expanded — swap in a full
  // library like ical.js later if you need those.
  function parseICS(icsText) {
    const rawLines = icsText.split(/\r\n|\n|\r/);
    const lines = [];
    for (const line of rawLines) {
      if ((line.startsWith(' ') || line.startsWith('\t')) && lines.length) {
        lines[lines.length - 1] += line.slice(1);
      } else {
        lines.push(line);
      }
    }

    const events = [];
    let cur = null;
    for (const line of lines) {
      if (line === 'BEGIN:VEVENT') { cur = {}; continue; }
      if (line === 'END:VEVENT') { if (cur && cur.start) events.push(cur); cur = null; continue; }
      if (!cur) continue;
      const idx = line.indexOf(':');
      if (idx === -1) continue;
      const key = line.slice(0, idx).split(';')[0];
      const value = line.slice(idx + 1);
      if (key === 'SUMMARY') cur.summary = value.replace(/\\,/g, ',').replace(/\\n/gi, ' ');
      if (key === 'DTSTART') { cur.start = parseICSDate(value); cur.allDay = /^\d{8}$/.test(value); }
      if (key === 'DTEND') cur.end = parseICSDate(value);
      if (key === 'ATTENDEE' || key === 'ORGANIZER') {
        const m = value.match(/mailto:([^;\s]+)/i);
        if (m) { (cur.attendees = cur.attendees || []).push(m[1].trim().toLowerCase()); }
      }
    }

    return events.map(e => ({
      summary: e.summary || '(untitled)',
      start: e.start,
      end: e.end || e.start,
      allDay: !!e.allDay,
      attendees: e.attendees || []
    }));
  }

  function parseICSDate(value) {
    const m = value.match(/^(\d{4})(\d{2})(\d{2})(T(\d{2})(\d{2})(\d{2})Z?)?$/);
    if (!m) return null;
    const [, y, mo, d, , h = '0', mi = '0', s = '0'] = m;
    if (value.endsWith('Z')) return new Date(Date.UTC(+y, +mo - 1, +d, +h, +mi, +s));
    return new Date(+y, +mo - 1, +d, +h, +mi, +s);
  }

  // Who a parsed event should show as assigned: the feed's default people,
  // plus anyone else matched by email in the event's attendee list.
  function resolveMembers(feed, attendeeEmails) {
    const defaults = (feed.defaultPeople || [])
      .map(name => CONFIG.people.find(p => p.name === name))
      .filter(Boolean);
    const matched = CONFIG.people.filter(p =>
      p.email && attendeeEmails.includes(p.email.toLowerCase())
    );
    const combined = [...defaults];
    for (const m of matched) {
      if (!combined.some(c => c.name === m.name)) combined.push(m);
    }
    return combined.length ? combined : defaults;
  }

  // ---------------------- demo events (used until feedUrl is set) ----------------------
  function demoEventsFor(feed, weekStart) {
    const members = (feed.defaultPeople || [])
      .map(name => CONFIG.people.find(p => p.name === name))
      .filter(Boolean);
    const samples = ['Dentist appointment', 'Soccer practice', 'Grocery run', 'Book club', 'Piano lesson'];
    const offsets = [1, 3, 5];
    return offsets.map((offset, i) => ({
      summary: samples[(i + feed.name.length) % samples.length] + ' (sample)',
      start: new Date(weekStart.getTime() + offset * DAY_MS + (9 + i * 2) * 60 * 60 * 1000),
      end: new Date(weekStart.getTime() + offset * DAY_MS + (10 + i * 2) * 60 * 60 * 1000),
      allDay: false,
      sample: true,
      members: members.length ? members : [{ name: feed.name, color: '#999999' }]
    }));
  }

  // ---------------------- fetch + render the week ----------------------
  async function loadCalendars() {
    const today = startOfDay(new Date());
    const days = Array.from({ length: 7 }, (_, i) => new Date(today.getTime() + i * DAY_MS));
    let allEvents = [];
    let anyRealFeed = false;

    for (const feed of CONFIG.feeds) {
      if (!feed.feedUrl) {
        allEvents.push(...demoEventsFor(feed, today));
        continue;
      }
      anyRealFeed = true;
      try {
        const res = await fetch(feed.feedUrl);
        if (!res.ok) throw new Error('HTTP ' + res.status);
        const text = await res.text();
        for (const e of parseICS(text)) {
          e.members = resolveMembers(feed, e.attendees);
          allEvents.push(e);
        }
      } catch (err) {
        console.error('Could not load feed', feed.name, err);
        allEvents.push(...demoEventsFor(feed, today));
      }
    }

    if (!anyRealFeed && CONFIG.people.length > 1) {
      // Illustrates multi-person assignment before any real feeds are wired up.
      allEvents.push({
        summary: 'Family game night (sample)',
        start: new Date(today.getTime() + 4 * DAY_MS + 18 * 60 * 60 * 1000),
        end: new Date(today.getTime() + 4 * DAY_MS + 19.5 * 60 * 60 * 1000),
        allDay: false,
        sample: true,
        members: CONFIG.people.slice(0, Math.min(3, CONFIG.people.length))
      });
    }

    renderWeek(days, allEvents);
  }

  function renderWeek(days, events) {
    const grid = document.getElementById('weekGrid');
    grid.innerHTML = '';
    document.getElementById('monthTitle').textContent =
      days[0].toLocaleDateString(undefined, { month: 'long', year: 'numeric' });

    const today = startOfDay(new Date()).getTime();

    for (const day of days) {
      const col = document.createElement('div');
      col.className = 'day-col' + (day.getTime() === today ? ' today' : '');

      const header = document.createElement('div');
      header.className = 'day-header';
      header.innerHTML = `<span class="dow">${fmtDay(day)}</span><span class="dnum">${fmtDate(day)}</span>`;
      col.appendChild(header);

      const dayEvents = events
        .filter(e => e.start && startOfDay(e.start).getTime() === day.getTime())
        .sort((a, b) => a.start - b.start);

      if (dayEvents.length === 0) {
        const empty = document.createElement('div');
        empty.className = 'no-events';
        empty.textContent = '—';
        col.appendChild(empty);
      }

      for (const ev of dayEvents) {
        col.appendChild(renderEventChip(ev));
      }
      grid.appendChild(col);
    }
  }

  function renderEventChip(ev) {
    const chip = document.createElement('div');
    const members = ev.members && ev.members.length ? ev.members : [{ color: '#999999' }];
    chip.className = 'event-chip' + (ev.sample ? ' sample' : '') + (members.length > 1 ? ' shared' : '');

    if (members.length === 1) {
      chip.style.background = hexToRGBA(members[0].color, 0.22);
      chip.style.borderLeftColor = members[0].color;
    } else {
      chip.style.background = 'rgba(241, 236, 221, 0.08)';
    }

    const dots = members.map(m => `<span class="chip-dot" style="background:${m.color}"></span>`).join('');
    chip.innerHTML =
      `<span class="chip-dots">${dots}</span>` +
      `<span class="chip-time">${ev.allDay ? 'All day' : fmtTime(ev.start)}</span>` +
      `<span class="chip-title">${escapeHTML(ev.summary)}</span>`;
    return chip;
  }

  function renderLegend() {
    const legend = document.getElementById('legend');
    legend.innerHTML = CONFIG.people.map(p =>
      `<span class="legend-item"><span class="dot" style="background:${p.color}"></span>${escapeHTML(p.name)}</span>`
    ).join('');

    const ownerSelect = document.getElementById('newChoreOwner');
    ownerSelect.innerHTML =
      CONFIG.people.map(p => `<option value="${escapeHTML(p.name)}">${escapeHTML(p.name)}</option>`).join('') +
      '<option value="">Whole family</option>';
  }

  // ---------------------- chores (Firebase Firestore, with local fallback) ----------------------
  let db = null;
  let localChores = [
    { id: 'l1', text: 'Take out the trash', owner: '', done: false, createdAt: 1 },
    { id: 'l2', text: 'Pack lunches', owner: '', done: false, createdAt: 2 },
  ];

  function initFirebase() {
    if (!CONFIG.firebase.apiKey || !CONFIG.firebase.projectId) {
      document.getElementById('choresStatus').textContent =
        'Add your Firebase config in CONFIG to enable a synced chore list. Showing local-only sample chores for now.';
      renderLocalChores();
      return;
    }
    try {
      firebase.initializeApp(CONFIG.firebase);
      db = firebase.firestore();
      subscribeToChores();
    } catch (err) {
      console.error('Firebase init failed', err);
      document.getElementById('choresStatus').textContent = 'Could not connect to Firebase — check your config.';
      renderLocalChores();
    }
  }

  function subscribeToChores() {
    db.collection('chores').orderBy('createdAt', 'asc').onSnapshot(snapshot => {
      const list = document.getElementById('choreList');
      list.innerHTML = '';
      snapshot.forEach(doc => list.appendChild(choreRow(doc.id, doc.data())));
    }, err => {
      console.error(err);
      document.getElementById('choresStatus').textContent = 'Lost connection to the chore list.';
    });
  }

  function choreRow(id, data) {
    const li = document.createElement('li');
    li.className = 'chore-row' + (data.done ? ' done' : '');
    const cb = document.createElement('input');
    cb.type = 'checkbox';
    cb.checked = !!data.done;
    cb.addEventListener('change', () => toggleChore(id, cb.checked));
    const label = document.createElement('span');
    label.className = 'chore-text';
    label.textContent = data.text + (data.owner ? ` — ${data.owner}` : '');
    li.append(cb, label);
    return li;
  }

  function toggleChore(id, done) {
    if (db) {
      db.collection('chores').doc(id).update({ done });
    } else {
      const item = localChores.find(c => c.id === id);
      if (item) { item.done = done; renderLocalChores(); }
    }
  }

  function renderLocalChores() {
    const list = document.getElementById('choreList');
    list.innerHTML = '';
    localChores.forEach(c => list.appendChild(choreRow(c.id, c)));
  }

  document.getElementById('addChoreForm').addEventListener('submit', (e) => {
    e.preventDefault();
    const textInput = document.getElementById('newChoreText');
    const ownerSelect = document.getElementById('newChoreOwner');
    const text = textInput.value.trim();
    if (!text) return;

    if (db) {
      db.collection('chores').add({ text, owner: ownerSelect.value, done: false, createdAt: Date.now() });
    } else {
      localChores.push({ id: 'l' + Date.now(), text, owner: ownerSelect.value, done: false, createdAt: Date.now() });
      renderLocalChores();
    }
    textInput.value = '';
  });

  document.getElementById('refreshBtn').addEventListener('click', loadCalendars);

  // ---------------------- boot ----------------------
  renderLegend();
  loadCalendars();
  initFirebase();
  setInterval(loadCalendars, CONFIG.refreshIntervalMinutes * 60 * 1000);
</script>
</body>
</html>
