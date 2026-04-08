<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover" />
  <title>Runaway Alarm 3</title>
  <style>
    :root {
      --bg: #0f1115;
      --panel: #181b22;
      --panel-2: #222733;
      --line: #2d3442;
      --text: #f5f7fb;
      --sub: #a9b2c3;
      --accent: #ff6b8b;
      --accent-2: #7aa2ff;
      --danger: #ff5d5d;
      --shadow: 0 12px 30px rgba(0,0,0,.28);
    }

    * { box-sizing: border-box; }
    html, body { height: 100%; }
    body {
      margin: 0;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      background: linear-gradient(180deg, #12151c 0%, #0d1016 100%);
      color: var(--text);
      -webkit-tap-highlight-color: transparent;
      overflow-x: hidden;
    }

    .app {
      min-height: 100%;
      max-width: 480px;
      margin: 0 auto;
      padding: 18px 16px 120px;
    }

    .topbar {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-bottom: 18px;
      padding-top: env(safe-area-inset-top, 0px);
    }

    .title {
      font-size: 1.8rem;
      font-weight: 800;
      letter-spacing: .02em;
    }

    .subtitle {
      color: var(--sub);
      font-size: .92rem;
      margin-top: 4px;
    }

    .icon-btn, .fab, .ghost-btn, .primary-btn {
      border: 0;
      cursor: pointer;
      color: var(--text);
    }

    .icon-btn {
      width: 42px;
      height: 42px;
      border-radius: 50%;
      background: var(--panel);
      box-shadow: var(--shadow);
      font-size: 1.2rem;
    }

    .status-card {
      background: linear-gradient(180deg, rgba(255,107,139,.16), rgba(122,162,255,.14));
      border: 1px solid rgba(255,255,255,.08);
      border-radius: 20px;
      padding: 14px 16px;
      margin-bottom: 16px;
      box-shadow: var(--shadow);
    }

    .status-card strong { display: block; font-size: 1rem; }
    .status-card span { color: #e7ebf5; opacity: .88; font-size: .9rem; }

    .list {
      display: flex;
      flex-direction: column;
      gap: 12px;
    }

    .alarm-item {
      position: relative;
      overflow: hidden;
      border-radius: 20px;
      touch-action: pan-y;
      user-select: none;
    }

    .delete-bg {
      position: absolute;
      inset: 0;
      display: flex;
      align-items: center;
      justify-content: flex-end;
      padding-right: 22px;
      background: linear-gradient(180deg, rgba(255,93,93,.96), rgba(223,60,60,.96));
      color: white;
      font-weight: 700;
      letter-spacing: .04em;
    }

    .alarm-card {
      position: relative;
      background: var(--panel);
      border: 1px solid var(--line);
      border-radius: 20px;
      padding: 16px;
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 12px;
      transform: translateX(0px);
      transition: transform .18s ease;
      box-shadow: var(--shadow);
    }

    .time-block {
      min-width: 0;
      flex: 1;
    }

    .time {
      font-size: 2rem;
      font-weight: 800;
      line-height: 1;
      letter-spacing: .03em;
    }

    .name {
      margin-top: 8px;
      font-weight: 700;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }

    .meta {
      margin-top: 4px;
      color: var(--sub);
      font-size: .87rem;
      display: flex;
      flex-wrap: wrap;
      gap: 6px;
    }

    .chip {
      padding: 4px 8px;
      border-radius: 999px;
      background: var(--panel-2);
      border: 1px solid rgba(255,255,255,.06);
    }

    .toggle {
      position: relative;
      width: 56px;
      height: 34px;
      flex: 0 0 auto;
    }

    .toggle input { display: none; }

    .slider {
      position: absolute;
      inset: 0;
      background: #363c4a;
      border-radius: 999px;
      transition: .2s ease;
      cursor: pointer;
    }

    .slider::before {
      content: "";
      position: absolute;
      width: 28px;
      height: 28px;
      left: 3px;
      top: 3px;
      border-radius: 50%;
      background: white;
      transition: .2s ease;
      box-shadow: 0 4px 10px rgba(0,0,0,.25);
    }

    .toggle input:checked + .slider {
      background: linear-gradient(90deg, var(--accent), #ff8fb9);
    }

    .toggle input:checked + .slider::before {
      transform: translateX(22px);
    }

    .empty {
      background: var(--panel);
      border: 1px dashed var(--line);
      border-radius: 20px;
      padding: 28px 18px;
      text-align: center;
      color: var(--sub);
      box-shadow: var(--shadow);
    }

    .fab {
      position: fixed;
      right: 18px;
      bottom: calc(18px + env(safe-area-inset-bottom, 0px));
      width: 64px;
      height: 64px;
      border-radius: 50%;
      background: linear-gradient(135deg, var(--accent), #ff8fb9);
      box-shadow: 0 14px 24px rgba(255,107,139,.35);
      font-size: 2rem;
      font-weight: 300;
      z-index: 30;
    }

    .sheet-backdrop, .alarm-screen {
      position: fixed;
      inset: 0;
      z-index: 50;
      display: none;
    }

    .sheet-backdrop.show, .alarm-screen.show {
      display: block;
    }

    .sheet-backdrop {
      background: rgba(0,0,0,.52);
      backdrop-filter: blur(10px);
    }

    .sheet {
      position: absolute;
      left: 0;
      right: 0;
      bottom: 0;
      max-width: 480px;
      margin: 0 auto;
      background: #161922;
      border-top-left-radius: 26px;
      border-top-right-radius: 26px;
      border: 1px solid rgba(255,255,255,.06);
      padding: 18px 16px calc(24px + env(safe-area-inset-bottom, 0px));
      box-shadow: 0 -12px 40px rgba(0,0,0,.35);
      max-height: 92vh;
      overflow: auto;
    }

    .sheet-header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-bottom: 16px;
      position: sticky;
      top: 0;
      background: #161922;
      padding-bottom: 10px;
      z-index: 2;
    }

    .sheet-title {
      font-size: 1.2rem;
      font-weight: 800;
    }

    .ghost-btn, .primary-btn {
      border-radius: 14px;
      padding: 10px 14px;
      font-weight: 700;
      font-size: .95rem;
    }

    .ghost-btn { background: #242a36; }
    .primary-btn { background: linear-gradient(135deg, var(--accent), #ff8fb9); }

    .form-grid {
      display: grid;
      gap: 14px;
    }

    .field {
      background: var(--panel);
      border: 1px solid var(--line);
      border-radius: 18px;
      padding: 14px;
    }

    .field label {
      display: block;
      color: var(--sub);
      font-size: .84rem;
      margin-bottom: 8px;
      font-weight: 700;
    }

    .field input[type="text"],
    .field input[type="time"],
    .field input[type="number"] {
      width: 100%;
      background: #10141b;
      color: var(--text);
      border: 1px solid #303949;
      border-radius: 14px;
      padding: 13px 14px;
      font-size: 1rem;
    }

    .segmented, .weekdays, .difficulty-row, .sound-row {
      display: flex;
      flex-wrap: wrap;
      gap: 8px;
    }

    .segmented button,
    .weekdays button,
    .difficulty-row button,
    .sound-row button {
      background: #11161d;
      color: var(--text);
      border: 1px solid #323949;
      border-radius: 999px;
      padding: 10px 12px;
      cursor: pointer;
      font-weight: 700;
    }

    .segmented button.active,
    .weekdays button.active,
    .difficulty-row button.active,
    .sound-row button.active {
      background: linear-gradient(135deg, rgba(255,107,139,.18), rgba(122,162,255,.16));
      border-color: rgba(255,255,255,.2);
    }

    .help {
      color: var(--sub);
      font-size: .83rem;
      line-height: 1.45;
      margin-top: 8px;
    }

    .alarm-screen {
      background: radial-gradient(circle at top, rgba(255,107,139,.18), transparent 35%), #090b0f;
      z-index: 100;
    }

    .alarm-inner {
      min-height: 100%;
      max-width: 480px;
      margin: 0 auto;
      padding: 28px 18px calc(32px + env(safe-area-inset-bottom, 0px));
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      text-align: center;
    }

    .alarm-badge {
      align-self: center;
      background: rgba(255,255,255,.08);
      border: 1px solid rgba(255,255,255,.08);
      border-radius: 999px;
      padding: 10px 14px;
      color: #e8ecf7;
      font-weight: 700;
      margin-bottom: 12px;
    }

    .alarm-clock {
      font-size: 4rem;
      font-weight: 900;
      line-height: 1;
      letter-spacing: .04em;
      margin-top: 8px;
    }

    .alarm-name {
      font-size: 1.4rem;
      font-weight: 800;
      margin-top: 16px;
    }

    .alarm-note {
      color: var(--sub);
      font-size: 1rem;
      line-height: 1.6;
      margin-top: 12px;
    }

    .runaway-zone {
      position: relative;
      height: 200px;
      margin-top: 18px;
    }

    .bar {
      position: fixed;
      width: 300px;
      height: 72px;
      border-radius: 999px;
      background: rgba(255,255,255,.08);
      border: 1px solid rgba(255,255,255,.12);
      box-shadow: 0 16px 32px rgba(0,0,0,.28);
      overflow: hidden;
      z-index: 160;
      transform-origin: center center;
      touch-action: none;
    }

    .track-fill {
      position: absolute;
      left: 0;
      top: 0;
      bottom: 0;
      width: 0%;
      background: linear-gradient(90deg, rgba(255,107,139,.35), rgba(122,162,255,.28));
      transition: width .08s linear;
    }

    .runaway-thumb {
      position: absolute;
      top: 8px;
      left: 8px;
      width: 124px;
      height: 56px;
      border-radius: 999px;
      background: linear-gradient(135deg, var(--accent), #ff8fb9);
      display: flex;
      align-items: center;
      justify-content: center;
      font-weight: 900;
      color: white;
      touch-action: none;
      box-shadow: 0 12px 20px rgba(255,107,139,.26);
      user-select: none;
    }

    .alarm-actions {
      margin-top: 16px;
      display: flex;
      gap: 12px;
    }

    .alarm-actions button {
      flex: 1;
      min-height: 52px;
      border-radius: 16px;
      font-weight: 800;
      border: 0;
      color: var(--text);
      background: #1d2330;
    }

    .alarm-actions .snooze-btn {
      background: linear-gradient(135deg, rgba(122,162,255,.22), rgba(122,162,255,.12));
    }

    .toast {
      position: fixed;
      left: 50%;
      bottom: 96px;
      transform: translateX(-50%) translateY(16px);
      background: rgba(15,17,21,.96);
      color: white;
      padding: 12px 14px;
      border-radius: 999px;
      border: 1px solid rgba(255,255,255,.08);
      opacity: 0;
      pointer-events: none;
      transition: .22s ease;
      z-index: 220;
      white-space: nowrap;
    }

    .toast.show {
      opacity: 1;
      transform: translateX(-50%) translateY(0);
    }
  </style>
</head>
<body>
  <div class="app">
    <div class="topbar">
      <div>
        <div class="title">Runaway Alarm 2</div>
        <div class="subtitle">前より意地悪。勢い突破もしにくくしました。</div>
      </div>
      <button class="icon-btn" id="previewBtn" title="テスト再生">⏰</button>
    </div>

    <div class="status-card">
      <strong>注意</strong>
      <span>このアプリはブラウザを開いているときだけ確実に動きます。GitHub Pages向けの仕様です。</span>
    </div>

    <div class="list" id="alarmList"></div>
  </div>

  <button class="fab" id="addBtn" aria-label="追加">＋</button>

  <div class="sheet-backdrop" id="sheetBackdrop">
    <div class="sheet">
      <div class="sheet-header">
        <button class="ghost-btn" id="cancelBtn">閉じる</button>
        <div class="sheet-title" id="sheetTitle">アラーム追加</div>
        <button class="primary-btn" id="saveBtn">保存</button>
      </div>

      <div class="form-grid">
        <div class="field">
          <label for="alarmName">アラーム名（任意）</label>
          <input id="alarmName" type="text" maxlength="30" placeholder="例: 学校 / 絶対起きる用" />
        </div>

        <div class="field">
          <label for="alarmTime">時刻</label>
          <input id="alarmTime" type="time" />
        </div>

        <div class="field">
          <label>種類</label>
          <div class="segmented" id="typeSegment">
            <button data-type="once" class="active">単発</button>
            <button data-type="weekly">曜日</button>
            <button data-type="monthly">毎月</button>
          </div>
        </div>

        <div class="field" id="weeklyField" style="display:none;">
          <label>曜日</label>
          <div class="weekdays" id="weekdayButtons"></div>
        </div>

        <div class="field" id="monthlyField" style="display:none;">
          <label for="monthDay">毎月何日</label>
          <input id="monthDay" type="number" min="1" max="31" placeholder="1〜31" />
        </div>

        <div class="field">
          <label for="snoozeMinutes">スヌーズ（分）</label>
          <input id="snoozeMinutes" type="number" min="1" max="60" value="5" />
          <div class="help">鳴動中に「スヌーズ」を押すと、この分数だけ後にもう一度鳴ります。</div>
        </div>

        <div class="field">
          <label>難易度</label>
          <div class="difficulty-row" id="difficultyRow">
            <button data-level="easy" class="active">やさしい</button>
            <button data-level="normal">普通</button>
            <button data-level="evil">悪意MAX</button>
          </div>
        </div>

        <div class="field">
          <label>アラーム音</label>
          <div class="sound-row" id="soundRow"></div>
        </div>
      </div>
    </div>
  </div>

  <div class="alarm-screen" id="alarmScreen">
    <div class="alarm-inner">
      <div>
        <div class="alarm-badge" id="difficultyBadge">普通</div>
        <div class="alarm-clock" id="ringTime">00:00</div>
        <div class="alarm-name" id="ringName">アラーム</div>
        <div class="alarm-note" id="ringNote">バー本体を追いかけながら、中の「止める」を右端まで運んでください。</div>
      </div>

      <div>
        <div class="runaway-zone"></div>
        <div class="bar" id="bar">
          <div class="track-fill" id="trackFill"></div>
          <div class="runaway-thumb" id="runawayThumb">止める</div>
        </div>
        <div class="alarm-actions">
          <button class="snooze-btn" id="snoozeBtn">スヌーズ</button>
          <button id="giveUpBtn">まだ無理</button>
        </div>
      </div>
    </div>
  </div>

  <div class="toast" id="toast"></div>

  <script>
    const STORAGE_KEY = 'runaway-alarm-app-v4';
    const WEEKDAY_LABELS = ['日', '月', '火', '水', '木', '金', '土'];
    const SOUND_OPTIONS = [
      { id: 'bright', label: 'Bright Bell' },
      { id: 'digital', label: 'Digital Panic' },
      { id: 'pulse', label: 'Pulse Siren' }
    ];

    const appState = {
      alarms: [],
      editingId: null,
      ringingQueue: [],
      currentAlarm: null,
      audioContext: null,
      currentSoundNodes: [],
      checkTimer: null,
      previewMode: false,
      drag: null,
      thumbLeft: 8,
      progress: 0,
      orbitAngle: 0,
      orbitSpeed: 0.014,
      orbitRadiusX: 100,
      orbitRadiusY: 170,
      centerX: window.innerWidth / 2,
      centerY: window.innerHeight / 2,
      barWidth: 300,
      barHeight: 72,
      rotation: 0,
      targetRotation: 0,
      lastEscapeAt: 0,
      holdStartAt: 0,
      holdActive: false,
      lastPointerX: 0,
      lastMoveAt: 0
    };

    const el = {
      alarmList: document.getElementById('alarmList'),
      addBtn: document.getElementById('addBtn'),
      previewBtn: document.getElementById('previewBtn'),
      sheetBackdrop: document.getElementById('sheetBackdrop'),
      sheetTitle: document.getElementById('sheetTitle'),
      cancelBtn: document.getElementById('cancelBtn'),
      saveBtn: document.getElementById('saveBtn'),
      alarmName: document.getElementById('alarmName'),
      alarmTime: document.getElementById('alarmTime'),
      typeSegment: document.getElementById('typeSegment'),
      weeklyField: document.getElementById('weeklyField'),
      monthlyField: document.getElementById('monthlyField'),
      weekdayButtons: document.getElementById('weekdayButtons'),
      monthDay: document.getElementById('monthDay'),
      snoozeMinutes: document.getElementById('snoozeMinutes'),
      difficultyRow: document.getElementById('difficultyRow'),
      soundRow: document.getElementById('soundRow'),
      alarmScreen: document.getElementById('alarmScreen'),
      difficultyBadge: document.getElementById('difficultyBadge'),
      ringTime: document.getElementById('ringTime'),
      ringName: document.getElementById('ringName'),
      ringNote: document.getElementById('ringNote'),
      bar: document.getElementById('bar'),
      trackFill: document.getElementById('trackFill'),
      runawayThumb: document.getElementById('runawayThumb'),
      snoozeBtn: document.getElementById('snoozeBtn'),
      giveUpBtn: document.getElementById('giveUpBtn'),
      toast: document.getElementById('toast')
    };

    function uid() {
      return Math.random().toString(36).slice(2, 10);
    }

    function loadState() {
      try {
        const raw = localStorage.getItem(STORAGE_KEY);
        if (!raw) return;
        const parsed = JSON.parse(raw);
        if (Array.isArray(parsed.alarms)) appState.alarms = parsed.alarms;
      } catch (e) {
        console.error(e);
      }
    }

    function saveState() {
      localStorage.setItem(STORAGE_KEY, JSON.stringify({ alarms: appState.alarms }));
    }

    function pad(num) {
      return String(num).padStart(2, '0');
    }

    function nowKey(date = new Date()) {
      return `${date.getFullYear()}-${pad(date.getMonth()+1)}-${pad(date.getDate())} ${pad(date.getHours())}:${pad(date.getMinutes())}`;
    }

    function repeatLabel(alarm) {
      if (alarm.type === 'once') return '単発';
      if (alarm.type === 'weekly') return alarm.days.length ? alarm.days.map(i => WEEKDAY_LABELS[i]).join(' ') : '曜日未設定';
      if (alarm.type === 'monthly') return `毎月 ${alarm.monthDay}日`;
      return '';
    }

    function difficultyLabel(level) {
      return level === 'easy' ? 'やさしい' : level === 'normal' ? '普通' : '悪意MAX';
    }

    function noteByDifficulty(level) {
      if (level === 'easy') return 'バー本体はゆっくり動きます。つまみも少しだけ逃げます。';
      if (level === 'normal') return 'バー本体がくるくる逃げます。終盤ほどつまみがしぶとく戻ります。';
      return 'Runaway Alarm 2 仕様です。かなり意地悪です。勢い突破もしにくくしてあります。';
    }

    function initWeekdays() {
      el.weekdayButtons.innerHTML = '';
      WEEKDAY_LABELS.forEach((label, index) => {
        const btn = document.createElement('button');
        btn.type = 'button';
        btn.textContent = label;
        btn.dataset.day = index;
        btn.addEventListener('click', () => btn.classList.toggle('active'));
        el.weekdayButtons.appendChild(btn);
      });
    }

    function initSounds() {
      el.soundRow.innerHTML = '';
      SOUND_OPTIONS.forEach((sound, idx) => {
        const btn = document.createElement('button');
        btn.type = 'button';
        btn.textContent = sound.label;
        btn.dataset.sound = sound.id;
        if (idx === 0) btn.classList.add('active');
        btn.addEventListener('click', () => {
          [...el.soundRow.children].forEach(node => node.classList.remove('active'));
          btn.classList.add('active');
          beepPreview(sound.id);
        });
        el.soundRow.appendChild(btn);
      });
    }

    function setSelectedType(type) {
      [...el.typeSegment.querySelectorAll('button')].forEach(btn => btn.classList.toggle('active', btn.dataset.type === type));
      el.weeklyField.style.display = type === 'weekly' ? 'block' : 'none';
      el.monthlyField.style.display = type === 'monthly' ? 'block' : 'none';
    }

    function setSelectedDifficulty(level) {
      [...el.difficultyRow.querySelectorAll('button')].forEach(btn => btn.classList.toggle('active', btn.dataset.level === level));
    }

    function setSelectedSound(soundId) {
      [...el.soundRow.querySelectorAll('button')].forEach(btn => btn.classList.toggle('active', btn.dataset.sound === soundId));
    }

    function setSelectedDays(days = []) {
      [...el.weekdayButtons.children].forEach(btn => btn.classList.toggle('active', days.includes(Number(btn.dataset.day))));
    }

    function getSelectedType() {
      return el.typeSegment.querySelector('.active')?.dataset.type || 'once';
    }

    function getSelectedDifficulty() {
      return el.difficultyRow.querySelector('.active')?.dataset.level || 'easy';
    }

    function getSelectedSound() {
      return el.soundRow.querySelector('.active')?.dataset.sound || SOUND_OPTIONS[0].id;
    }

    function getSelectedDays() {
      return [...el.weekdayButtons.querySelectorAll('.active')].map(btn => Number(btn.dataset.day));
    }

    function resetForm() {
      appState.editingId = null;
      el.sheetTitle.textContent = 'アラーム追加';
      el.alarmName.value = '';
      el.alarmTime.value = '07:00';
      setSelectedType('once');
      setSelectedDays([]);
      el.monthDay.value = '';
      el.snoozeMinutes.value = 5;
      setSelectedDifficulty('easy');
      setSelectedSound(SOUND_OPTIONS[0].id);
    }

    function openSheet(alarm = null) {
      if (!alarm) {
        resetForm();
      } else {
        appState.editingId = alarm.id;
        el.sheetTitle.textContent = 'アラーム編集';
        el.alarmName.value = alarm.name || '';
        el.alarmTime.value = alarm.time;
        setSelectedType(alarm.type);
        setSelectedDays(alarm.days || []);
        el.monthDay.value = alarm.monthDay || '';
        el.snoozeMinutes.value = alarm.snoozeMinutes || 5;
        setSelectedDifficulty(alarm.difficulty || 'easy');
        setSelectedSound(alarm.sound || SOUND_OPTIONS[0].id);
      }
      el.sheetBackdrop.classList.add('show');
    }

    function closeSheet() {
      el.sheetBackdrop.classList.remove('show');
    }

    function validateForm() {
      const type = getSelectedType();
      if (!el.alarmTime.value) return '時刻を入力してください。';
      if (type === 'weekly' && getSelectedDays().length === 0) return '曜日を1つ以上選んでください。';
      if (type === 'monthly') {
        const day = Number(el.monthDay.value);
        if (!day || day < 1 || day > 31) return '毎月の日付は1〜31で入力してください。';
      }
      const snooze = Number(el.snoozeMinutes.value);
      if (!snooze || snooze < 1 || snooze > 60) return 'スヌーズは1〜60分で入力してください。';
      return '';
    }

    function buildAlarmFromForm() {
      const existing = appState.alarms.find(a => a.id === appState.editingId);
      const type = getSelectedType();
      return {
        id: existing?.id || uid(),
        name: el.alarmName.value.trim(),
        time: el.alarmTime.value,
        type,
        days: type === 'weekly' ? getSelectedDays() : [],
        monthDay: type === 'monthly' ? Number(el.monthDay.value) : null,
        snoozeMinutes: Number(el.snoozeMinutes.value),
        difficulty: getSelectedDifficulty(),
        sound: getSelectedSound(),
        enabled: existing?.enabled ?? true,
        lastTriggeredAt: existing?.lastTriggeredAt || null,
        createdAt: existing?.createdAt || new Date().toISOString(),
        updatedAt: new Date().toISOString()
      };
    }

    function saveAlarm() {
      const error = validateForm();
      if (error) return showToast(error);
      const alarm = buildAlarmFromForm();
      const index = appState.alarms.findIndex(a => a.id === alarm.id);
      if (index >= 0) appState.alarms[index] = alarm;
      else appState.alarms.push(alarm);
      appState.alarms.sort((a, b) => a.time.localeCompare(b.time));
      saveState();
      renderAlarms();
      closeSheet();
      showToast(index >= 0 ? 'アラームを更新しました。' : 'アラームを追加しました。');
    }

    function deleteAlarm(id) {
      appState.alarms = appState.alarms.filter(a => a.id !== id);
      saveState();
      renderAlarms();
      showToast('削除しました。');
    }

    function renderAlarms() {
      el.alarmList.innerHTML = '';
      if (appState.alarms.length === 0) {
        el.alarmList.innerHTML = '<div class="empty">アラームがまだありません。<br>右下の＋から追加してください。</div>';
        return;
      }

      appState.alarms.forEach(alarm => {
        const item = document.createElement('div');
        item.className = 'alarm-item';
        item.innerHTML = `
          <div class="delete-bg">削除</div>
          <div class="alarm-card">
            <div class="time-block">
              <div class="time">${alarm.time}</div>
              <div class="name">${alarm.name || 'アラーム'}</div>
              <div class="meta">
                <span class="chip">${repeatLabel(alarm)}</span>
                <span class="chip">${difficultyLabel(alarm.difficulty)}</span>
                <span class="chip">${alarm.snoozeMinutes}分スヌーズ</span>
              </div>
            </div>
            <label class="toggle">
              <input type="checkbox" ${alarm.enabled ? 'checked' : ''} />
              <span class="slider"></span>
            </label>
          </div>`;

        const card = item.querySelector('.alarm-card');
        const checkbox = item.querySelector('input');
        checkbox.addEventListener('change', (e) => {
          alarm.enabled = e.target.checked;
          alarm.updatedAt = new Date().toISOString();
          saveState();
        });
        card.addEventListener('click', (e) => {
          if (e.target.closest('.toggle')) return;
          openSheet(alarm);
        });
        attachSwipeToDelete(item, card, () => deleteAlarm(alarm.id));
        el.alarmList.appendChild(item);
      });
    }

    function attachSwipeToDelete(container, card, onDelete) {
      let startX = 0;
      let currentX = 0;
      let dragging = false;

      const onPointerMove = (e) => {
        if (!dragging) return;
        currentX = e.clientX;
        const delta = Math.min(0, currentX - startX);
        card.style.transform = `translateX(${Math.max(delta, -110)}px)`;
      };

      const end = () => {
        if (!dragging) return;
        dragging = false;
        const delta = currentX - startX;
        if (delta < -80) {
          card.style.transform = 'translateX(-110px)';
          setTimeout(onDelete, 120);
        } else {
          card.style.transform = 'translateX(0px)';
        }
        window.removeEventListener('pointermove', onPointerMove);
        window.removeEventListener('pointerup', end);
        window.removeEventListener('pointercancel', end);
      };

      card.addEventListener('pointerdown', (e) => {
        if (e.target.closest('.toggle')) return;
        startX = e.clientX;
        currentX = e.clientX;
        dragging = true;
        window.addEventListener('pointermove', onPointerMove);
        window.addEventListener('pointerup', end);
        window.addEventListener('pointercancel', end);
      });
    }

    function shouldTrigger(alarm, now = new Date()) {
      if (!alarm.enabled) return false;
      const currentTime = `${pad(now.getHours())}:${pad(now.getMinutes())}`;
      if (alarm.time !== currentTime) return false;
      const key = nowKey(now);
      if (alarm.lastTriggeredAt === key) return false;
      if (alarm.type === 'once') return true;
      if (alarm.type === 'weekly') return alarm.days.includes(now.getDay());
      if (alarm.type === 'monthly') return now.getDate() === Number(alarm.monthDay);
      return false;
    }

    function queueDueAlarms() {
      const now = new Date();
      const due = appState.alarms.filter(a => shouldTrigger(a, now));
      if (!due.length) return;
      due.forEach(alarm => {
        alarm.lastTriggeredAt = nowKey(now);
        appState.ringingQueue.push({ ...alarm, queueAt: Date.now() });
      });
      saveState();
      if (!appState.currentAlarm) startNextAlarm();
    }

    function difficultyConfig(level) {
      if (level === 'easy') {
        return {
          orbitSpeed: 0.012,
          radiusX: 86,
          radiusY: 128,
          escapeChance: 0.18,
          knockBackMin: 18,
          knockBackMax: 30,
          microChance: 0.12,
          microBack: 12,
          pull: 0.8,
          releasePenalty: 7,
          rotateJolt: 34,
          holdMs: 550,
          antiFlickPx: 45
        };
      }
      if (level === 'normal') {
        return {
          orbitSpeed: 0.019,
          radiusX: 124,
          radiusY: 174,
          escapeChance: 0.29,
          knockBackMin: 28,
          knockBackMax: 46,
          microChance: 0.22,
          microBack: 18,
          pull: 0.68,
          releasePenalty: 10,
          rotateJolt: 56,
          holdMs: 800,
          antiFlickPx: 36
        };
      }
      return {
        orbitSpeed: 0.025,
        radiusX: 148,
        radiusY: 200,
        escapeChance: 0.38,
        knockBackMin: 34,
        knockBackMax: 58,
        microChance: 0.30,
        microBack: 22,
        pull: 0.6,
        releasePenalty: 14,
        rotateJolt: 76,
        holdMs: 1100,
        antiFlickPx: 28
      };
    }

    function resetHoldState() {
      appState.holdStartAt = 0;
      appState.holdActive = false;
    }

    function resetRunawayGame() {
      appState.thumbLeft = 8;
      appState.progress = 0;
      appState.orbitAngle = 0;
      appState.rotation = 0;
      appState.targetRotation = 0;
      appState.lastPointerX = 0;
      appState.lastMoveAt = 0;
      resetHoldState();
      applyDifficultyMotion(appState.currentAlarm?.difficulty || 'normal');
      updateThumb();
      updateProgress();
      updateOrbitCenter();
      updateBarTransform();
    }

    function applyDifficultyMotion(level) {
      const cfg = difficultyConfig(level);
      appState.orbitSpeed = cfg.orbitSpeed;
      appState.orbitRadiusX = Math.min(cfg.radiusX, Math.max(55, window.innerWidth / 2 - 170));
      appState.orbitRadiusY = Math.min(cfg.radiusY, Math.max(70, window.innerHeight / 2 - 180));
    }

    function updateOrbitCenter() {
      appState.centerX = window.innerWidth / 2;
      appState.centerY = Math.max(170, window.innerHeight / 2 + 20);
    }

    function getThumbMaxLeft() {
      return appState.barWidth - el.runawayThumb.offsetWidth - 8;
    }

    function clamp(v, min, max) {
      return Math.max(min, Math.min(max, v));
    }

    function updateThumb() {
      el.runawayThumb.style.left = `${appState.thumbLeft}px`;
    }

    function updateProgress() {
      el.trackFill.style.width = `${appState.progress}%`;
    }

    function updateBarTransform() {
      const x = appState.centerX + Math.cos(appState.orbitAngle) * appState.orbitRadiusX - appState.barWidth / 2;
      const y = appState.centerY + Math.sin(appState.orbitAngle) * appState.orbitRadiusY - appState.barHeight / 2;
      el.bar.style.left = `${x}px`;
      el.bar.style.top = `${y}px`;
      el.bar.style.transform = `rotate(${appState.rotation}deg)`;
    }

    function animateBar() {
      if (appState.currentAlarm) {
        appState.orbitAngle += appState.orbitSpeed;
        appState.rotation += (appState.targetRotation - appState.rotation) * 0.08;
        appState.targetRotation *= 0.985;
        updateBarTransform();
      }
      requestAnimationFrame(animateBar);
    }

    function maybeEscape(threat) {
      const cfg = difficultyConfig(appState.currentAlarm.difficulty);
      const now = performance.now();
      if (now - appState.lastEscapeAt < 180) return;
      if (Math.random() < (0.12 + cfg.escapeChance * (0.75 + threat * 1.1))) {
        appState.lastEscapeAt = now;
        appState.orbitAngle += (Math.random() > 0.5 ? 1 : -1) * (0.18 + Math.random() * 0.28);
        appState.targetRotation += (Math.random() > 0.5 ? 1 : -1) * (cfg.rotateJolt * (0.75 + threat));
      }
    }

    function startNextAlarm() {
      if (appState.currentAlarm || appState.ringingQueue.length === 0) return;
      const alarm = appState.ringingQueue.shift();
      appState.currentAlarm = alarm;
      appState.previewMode = false;
      el.alarmScreen.classList.add('show');
      el.difficultyBadge.textContent = difficultyLabel(alarm.difficulty);
      el.ringTime.textContent = alarm.time;
      el.ringName.textContent = alarm.name || 'アラーム';
      el.ringNote.textContent = noteByDifficulty(alarm.difficulty);
      resetRunawayGame();
      startAlarmSound(alarm.sound);
    }

    function stopCurrentAlarm() {
      stopAlarmSound();
      appState.currentAlarm = null;
      appState.drag = null;
      el.alarmScreen.classList.remove('show');
      if (appState.ringingQueue.length > 0) setTimeout(startNextAlarm, 250);
    }

    function completeAlarm() {
      resetHoldState();
      appState.progress = 100;
      updateProgress();
      showToast('Runaway Alarm 2 制覇です。かなりえらいです。');
      const current = appState.currentAlarm;
      stopCurrentAlarm();
      if (current && current._temporary) {
        appState.alarms = appState.alarms.filter(a => a.id !== current.id);
        saveState();
        renderAlarms();
      }
    }

    function snoozeCurrentAlarm() {
      const alarm = appState.currentAlarm;
      if (!alarm) return;
      const newAlarm = {
        ...alarm,
        id: uid(),
        type: 'once',
        enabled: true,
        name: alarm.name ? `${alarm.name}（スヌーズ）` : 'スヌーズ',
        time: addMinutesToHHMM(getCurrentHHMM(), alarm.snoozeMinutes),
        days: [],
        monthDay: null,
        createdAt: new Date().toISOString(),
        updatedAt: new Date().toISOString(),
        lastTriggeredAt: null,
        _temporary: true
      };
      appState.alarms.push(newAlarm);
      appState.alarms.sort((a, b) => a.time.localeCompare(b.time));
      saveState();
      renderAlarms();
      showToast(`${alarm.snoozeMinutes}分後に再アラームします。`);
      stopCurrentAlarm();
    }

    function getCurrentHHMM() {
      const now = new Date();
      return `${pad(now.getHours())}:${pad(now.getMinutes())}`;
    }

    function addMinutesToHHMM(hhmm, mins) {
      const [h, m] = hhmm.split(':').map(Number);
      const date = new Date();
      date.setHours(h, m + mins, 0, 0);
      return `${pad(date.getHours())}:${pad(date.getMinutes())}`;
    }

    function getAudioContext() {
      if (!appState.audioContext) appState.audioContext = new (window.AudioContext || window.webkitAudioContext)();
      return appState.audioContext;
    }

    function stopAlarmSound() {
      appState.currentSoundNodes.forEach(node => {
        try {
          if (node.stop) node.stop();
          if (node.disconnect) node.disconnect();
          if (node.gain && node.gain.disconnect) node.gain.disconnect();
        } catch (_) {}
      });
      appState.currentSoundNodes = [];
    }

    function beepPattern(soundId, duration = 0.26, offset = 0) {
      const ctx = getAudioContext();
      const start = ctx.currentTime + offset;
      const gain = ctx.createGain();
      gain.connect(ctx.destination);
      gain.gain.setValueAtTime(0.0001, start);
      gain.gain.exponentialRampToValueAtTime(0.22, start + 0.02);
      gain.gain.exponentialRampToValueAtTime(0.0001, start + duration);
      const osc = ctx.createOscillator();
      osc.type = soundId === 'digital' ? 'square' : soundId === 'pulse' ? 'triangle' : 'sine';
      const freq = soundId === 'bright' ? 880 : soundId === 'digital' ? 720 : 520;
      osc.frequency.setValueAtTime(freq, start);
      if (soundId === 'pulse') osc.frequency.linearRampToValueAtTime(640, start + duration);
      if (soundId === 'digital') osc.frequency.linearRampToValueAtTime(900, start + duration * 0.7);
      osc.connect(gain);
      osc.start(start);
      osc.stop(start + duration + 0.02);
      osc.gain = gain;
      return osc;
    }

    async function ensureAudioUnlocked() {
      const ctx = getAudioContext();
      if (ctx.state === 'suspended') {
        try { await ctx.resume(); } catch (_) {}
      }
    }

    async function beepPreview(soundId) {
      await ensureAudioUnlocked();
      stopAlarmSound();
      const a = beepPattern(soundId, 0.22, 0);
      const b = beepPattern(soundId, 0.18, 0.28);
      appState.currentSoundNodes = [a, b];
      setTimeout(() => { if (!appState.currentAlarm) stopAlarmSound(); }, 700);
    }

    async function startAlarmSound(soundId) {
      await ensureAudioUnlocked();
      stopAlarmSound();
      const nodes = [];
      for (let i = 0; i < 4; i++) nodes.push(beepPattern(soundId, 0.18 + (i % 2) * 0.1, i * 0.32));
      appState.currentSoundNodes = nodes;
      setTimeout(() => { if (appState.currentAlarm) startAlarmSound(soundId); }, 1350);
    }

    function removeExpiredTemporaryAlarms() {
      const now = new Date();
      const nowMinutes = now.getHours() * 60 + now.getMinutes();
      appState.alarms = appState.alarms.filter(alarm => {
        if (!alarm._temporary) return true;
        const [h, m] = alarm.time.split(':').map(Number);
        return h * 60 + m >= nowMinutes - 1;
      });
      saveState();
    }

    function previewAlarm() {
      if (appState.currentAlarm) return;
      const fake = {
        id: uid(),
        name: 'テスト鳴動',
        time: getCurrentHHMM(),
        difficulty: 'evil',
        sound: getSelectedSound(),
        snoozeMinutes: 5
      };
      appState.currentAlarm = fake;
      appState.previewMode = true;
      el.alarmScreen.classList.add('show');
      el.difficultyBadge.textContent = difficultyLabel(fake.difficulty);
      el.ringTime.textContent = fake.time;
      el.ringName.textContent = fake.name;
      el.ringNote.textContent = 'Runaway Alarm 2 のテストです。前よりさらにしぶといです。';
      resetRunawayGame();
      startAlarmSound(fake.sound);
    }

    function showToast(message) {
      el.toast.textContent = message;
      el.toast.classList.add('show');
      clearTimeout(showToast.timer);
      showToast.timer = setTimeout(() => el.toast.classList.remove('show'), 1800);
    }

    function bindEvents() {
      el.addBtn.addEventListener('click', () => openSheet());
      el.previewBtn.addEventListener('click', previewAlarm);
      el.cancelBtn.addEventListener('click', closeSheet);
      el.saveBtn.addEventListener('click', saveAlarm);
      el.sheetBackdrop.addEventListener('click', (e) => {
        if (e.target === el.sheetBackdrop) closeSheet();
      });
      [...el.typeSegment.querySelectorAll('button')].forEach(btn => btn.addEventListener('click', () => setSelectedType(btn.dataset.type)));
      [...el.difficultyRow.querySelectorAll('button')].forEach(btn => btn.addEventListener('click', () => setSelectedDifficulty(btn.dataset.level)));
      el.snoozeBtn.addEventListener('click', snoozeCurrentAlarm);
      el.giveUpBtn.addEventListener('click', () => showToast('まだです。Runaway Alarm 2 はさらに甘くありません。'));

      el.runawayThumb.addEventListener('pointerdown', (e) => {
        if (!appState.currentAlarm) return;
        appState.drag = {
          pointerId: e.pointerId,
          startX: e.clientX,
          thumbStart: appState.thumbLeft
        };
        appState.lastPointerX = e.clientX;
        appState.lastMoveAt = performance.now();
        el.runawayThumb.setPointerCapture(e.pointerId);
      });

      el.runawayThumb.addEventListener('pointermove', (e) => {
        if (!appState.drag || !appState.currentAlarm) return;
        const cfg = difficultyConfig(appState.currentAlarm.difficulty);
        const now = performance.now();
        const dt = Math.max(1, now - (appState.lastMoveAt || now));
        const flickDistance = Math.abs(e.clientX - (appState.lastPointerX || e.clientX));
        appState.lastPointerX = e.clientX;
        appState.lastMoveAt = now;

        const dx = e.clientX - appState.drag.startX;
        let next = appState.drag.thumbStart + dx * cfg.pull;
        const threat = clamp((next - 8) / (getThumbMaxLeft() - 8 || 1), 0, 1);

        if (flickDistance > cfg.antiFlickPx && dt < 26) {
          next -= cfg.knockBackMax * (0.8 + threat * 0.7);
          appState.targetRotation += cfg.rotateJolt * 0.7;
          resetHoldState();
        }

        if (Math.random() < (cfg.escapeChance + threat * 0.34)) {
          next -= cfg.knockBackMin + Math.random() * (cfg.knockBackMax - cfg.knockBackMin);
          resetHoldState();
        }
        if (Math.random() < (cfg.microChance + threat * 0.18)) {
          next -= cfg.microBack;
          resetHoldState();
        }

        maybeEscape(threat);

        appState.thumbLeft = clamp(next, 8, getThumbMaxLeft());
        appState.progress = Math.max(appState.progress, ((appState.thumbLeft - 8) / (getThumbMaxLeft() - 8 || 1)) * 100);
        updateThumb();
        updateProgress();

        if (appState.thumbLeft >= getThumbMaxLeft() - 2) {
          if (!appState.holdActive) {
            appState.holdActive = true;
            appState.holdStartAt = now;
            showToast('右端キープ中…！');
          }
          if (now - appState.holdStartAt >= cfg.holdMs) {
            completeAlarm();
          }
        } else {
          resetHoldState();
        }
      });

      function endDrag(e) {
        if (!appState.drag) return;
        try { el.runawayThumb.releasePointerCapture(e.pointerId); } catch (_) {}
        const cfg = difficultyConfig(appState.currentAlarm?.difficulty || 'normal');
        appState.drag = null;
        appState.thumbLeft = clamp(appState.thumbLeft - cfg.releasePenalty, 8, getThumbMaxLeft());
        appState.progress = clamp(appState.progress - Math.max(4, cfg.releasePenalty * 0.4), 0, 100);
        updateThumb();
        updateProgress();
        resetHoldState();
        maybeEscape(0.75);
      }

      el.runawayThumb.addEventListener('pointerup', endDrag);
      el.runawayThumb.addEventListener('pointercancel', endDrag);

      document.addEventListener('visibilitychange', () => {
        if (document.visibilityState === 'visible') removeExpiredTemporaryAlarms();
      });
      document.body.addEventListener('click', ensureAudioUnlocked, { once: true });
      window.addEventListener('resize', () => {
        updateOrbitCenter();
        applyDifficultyMotion(appState.currentAlarm?.difficulty || 'normal');
        updateBarTransform();
      });
    }

    function startScheduler() {
      clearInterval(appState.checkTimer);
      queueDueAlarms();
      appState.checkTimer = setInterval(queueDueAlarms, 1000);
    }

    function boot() {
      loadState();
      initWeekdays();
      initSounds();
      bindEvents();
      removeExpiredTemporaryAlarms();
      renderAlarms();
      updateOrbitCenter();
      updateThumb();
      updateProgress();
      updateBarTransform();
      animateBar();
      startScheduler();
    }

    boot();
  </script>
</body>
</html>
