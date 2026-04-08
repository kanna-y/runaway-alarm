<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover" />
  <title>Runaway Alarm</title>
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
      --ok: #35c16f;
      --shadow: 0 12px 30px rgba(0,0,0,.28);
      --radius: 18px;
    }

    * { box-sizing: border-box; }
    html, body { height: 100%; }
    body {
      margin: 0;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      background: linear-gradient(180deg, #12151c 0%, #0d1016 100%);
      color: var(--text);
      -webkit-tap-highlight-color: transparent;
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

    .icon-btn, .fab, .ghost-btn, .danger-btn, .primary-btn {
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

    .ghost-btn, .danger-btn, .primary-btn {
      border-radius: 14px;
      padding: 10px 14px;
      font-weight: 700;
      font-size: .95rem;
    }

    .ghost-btn { background: #242a36; }
    .danger-btn { background: rgba(255,93,93,.16); color: #ff9595; }
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
    .field input[type="number"],
    .field select {
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

    .inline-row {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 12px;
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
      height: 230px;
      border-radius: 24px;
      background: linear-gradient(180deg, #141922, #0f131b);
      border: 1px solid rgba(255,255,255,.08);
      overflow: hidden;
      margin-top: 18px;
    }

    .track {
  position: fixed;
  width: min(320px, calc(100vw - 32px));
  height: 68px;
  border-radius: 999px;
  background: rgba(255,255,255,.06);
  border: 1px solid rgba(255,255,255,.08);
  overflow: hidden;
  left: 16px;
  top: 220px;
  transition: left .16s ease, top .16s ease;
  z-index: 160;
  touch-action: none;
}

    .track-fill {
      position: absolute;
      left: 0;
      top: 0;
      bottom: 0;
      width: 0%;
      background: linear-gradient(90deg, rgba(255,107,139,.3), rgba(122,162,255,.26));
      transition: width .08s linear;
    }

    .runaway-thumb {
      position: absolute;
      top: 8px;
      left: 8px;
      width: 132px;
      height: 52px;
      border-radius: 999px;
      background: linear-gradient(135deg, var(--accent), #ff8fb9);
      display: flex;
      align-items: center;
      justify-content: center;
      font-weight: 900;
      color: white;
      touch-action: none;
      box-shadow: 0 12px 20px rgba(255,107,139,.26);
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
      z-index: 120;
      white-space: nowrap;
    }

    .toast.show {
      opacity: 1;
      transform: translateX(-50%) translateY(0);
    }

    @media (max-width: 370px) {
      .time { font-size: 1.7rem; }
      .alarm-clock { font-size: 3.2rem; }
      .runaway-thumb { width: 120px; }
    }
  </style>
</head>
<body>
  <div class="app">
    <div class="topbar">
      <div>
        <div class="title">Runaway Alarm</div>
        <div class="subtitle">見た目は普通、中身はだいぶ性格悪い。</div>
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
        <div class="alarm-note" id="ringNote">逃げるスライダーを最後まで追い詰めて止めてください。</div>
      </div>

      <div>
        <div class="runaway-zone" id="runawayZone">
          <div class="track" id="track">
            <div class="track-fill" id="trackFill"></div>
            <div class="runaway-thumb" id="runawayThumb">止める</div>
          </div>
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
    const STORAGE_KEY = 'runaway-alarm-app-v1';
    const WEEKDAY_LABELS = ['日', '月', '火', '水', '木', '金', '土'];
    const SOUND_OPTIONS = [
      { id: 'bright', label: 'Bright Bell' },
      { id: 'digital', label: 'Digital Panic' },
      { id: 'pulse', label: 'Pulse Siren' }
    ];

    const appState = {
      alarms: [],
      editingId: null,
      sheetOpen: false,
      ringingQueue: [],
      currentAlarm: null,
      audioContext: null,
      currentSoundNodes: [],
      checkTimer: null,
      dragged: null,
      progress: 0,
      currentThumbLeft: 8,
      previewMode: false
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
      track: document.getElementById('track'),
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
        console.error('Failed to load state', e);
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

    function fmtTime(time) {
      return time;
    }

    function repeatLabel(alarm) {
      if (alarm.type === 'once') return '単発';
      if (alarm.type === 'weekly') {
        return alarm.days.length ? alarm.days.map(i => WEEKDAY_LABELS[i]).join(' ') : '曜日未設定';
      }
      if (alarm.type === 'monthly') return `毎月 ${alarm.monthDay}日`;
      return '';
    }

    function difficultyLabel(level) {
      return level === 'easy' ? 'やさしい' : level === 'normal' ? '普通' : '悪意MAX';
    }

    function soundLabel(id) {
      return SOUND_OPTIONS.find(s => s.id === id)?.label || id;
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

    function getSelectedType() {
      return el.typeSegment.querySelector('.active')?.dataset.type || 'once';
    }

    function setSelectedType(type) {
      [...el.typeSegment.querySelectorAll('button')].forEach(btn => {
        btn.classList.toggle('active', btn.dataset.type === type);
      });
      el.weeklyField.style.display = type === 'weekly' ? 'block' : 'none';
      el.monthlyField.style.display = type === 'monthly' ? 'block' : 'none';
    }

    function getSelectedDifficulty() {
      return el.difficultyRow.querySelector('.active')?.dataset.level || 'easy';
    }

    function setSelectedDifficulty(level) {
      [...el.difficultyRow.querySelectorAll('button')].forEach(btn => {
        btn.classList.toggle('active', btn.dataset.level === level);
      });
    }

    function getSelectedSound() {
      return el.soundRow.querySelector('.active')?.dataset.sound || SOUND_OPTIONS[0].id;
    }

    function setSelectedSound(soundId) {
      [...el.soundRow.querySelectorAll('button')].forEach(btn => {
        btn.classList.toggle('active', btn.dataset.sound === soundId);
      });
    }

    function getSelectedDays() {
      return [...el.weekdayButtons.querySelectorAll('.active')].map(btn => Number(btn.dataset.day));
    }

    function setSelectedDays(days = []) {
      [...el.weekdayButtons.children].forEach(btn => {
        btn.classList.toggle('active', days.includes(Number(btn.dataset.day)));
      });
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
      appState.sheetOpen = true;
      el.sheetBackdrop.classList.add('show');
    }

    function closeSheet() {
      appState.sheetOpen = false;
      el.sheetBackdrop.classList.remove('show');
    }

    function validateForm() {
      const type = getSelectedType();
      const time = el.alarmTime.value;
      if (!time) return '時刻を入力してください。';
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
      const alarm = {
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
      return alarm;
    }

    function saveAlarm() {
      const error = validateForm();
      if (error) {
        showToast(error);
        return;
      }
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
              <div class="time">${fmtTime(alarm.time)}</div>
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
          </div>
        `;

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
      if (!appState.currentAlarm) {
        startNextAlarm();
      }
    }

    function startNextAlarm() {
      if (appState.currentAlarm || appState.ringingQueue.length === 0) return;
      const alarm = appState.ringingQueue.shift();
      appState.currentAlarm = alarm;
      appState.progress = 0;
      appState.previewMode = false;
      el.alarmScreen.classList.add('show');
      el.difficultyBadge.textContent = difficultyLabel(alarm.difficulty);
      el.ringTime.textContent = fmtTime(alarm.time);
      el.ringName.textContent = alarm.name || 'アラーム';
      el.ringNote.textContent = noteByDifficulty(alarm.difficulty);
      resetRunawayThumb();
      startAlarmSound(alarm.sound);
    }

    function noteByDifficulty(level) {
      if (level === 'easy') return 'ゆっくり逃げます。たぶん。';
      if (level === 'normal') return 'いい感じに腹が立つ程度で逃げます。';
      return '悪意MAXです。指先の尊厳を捨てて追いかけてください。';
    }

    function stopCurrentAlarm() {
      stopAlarmSound();
      appState.currentAlarm = null;
      el.alarmScreen.classList.remove('show');
      if (appState.ringingQueue.length > 0) {
        setTimeout(startNextAlarm, 250);
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
      if (!appState.audioContext) {
        appState.audioContext = new (window.AudioContext || window.webkitAudioContext)();
      }
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
      for (let i = 0; i < 4; i++) {
        nodes.push(beepPattern(soundId, 0.18 + (i % 2) * 0.1, i * 0.32));
      }
      appState.currentSoundNodes = nodes;
      setTimeout(() => {
        if (appState.currentAlarm) startAlarmSound(soundId);
      }, 1350);
    }

    function difficultyConfig(level) {
      if (level === 'easy') {
        return { escapeChance: 0.2, leap: 18, resistance: 0.78, jitter: 4 };
      }
      if (level === 'normal') {
        return { escapeChance: 0.48, leap: 34, resistance: 0.56, jitter: 9 };
      }
      return { escapeChance: 0.78, leap: 70, resistance: 0.38, jitter: 18 };
    }

    function resetRunawayThumb() {
      appState.progress = 0;
      el.trackFill.style.width = '0%';
      const maxLeft = getTrackMaxLeft();
      appState.currentThumbLeft = 8;
      positionThumb(Math.min(appState.currentThumbLeft, maxLeft));
    }

    function getTrackMaxLeft() {
      return el.track.clientWidth - el.runawayThumb.offsetWidth - 8;
    }

    function positionThumb(left) {
      const maxLeft = getTrackMaxLeft();
      const clamped = Math.max(8, Math.min(left, maxLeft));
      appState.currentThumbLeft = clamped;
      el.runawayThumb.style.left = `${clamped}px`;
    }

    function startDrag(e) {
      if (!appState.currentAlarm) return;
      appState.dragged = {
        pointerId: e.pointerId,
        startX: e.clientX,
        thumbStart: appState.currentThumbLeft,
      };
      el.runawayThumb.setPointerCapture(e.pointerId);
    }

    function moveDrag(e) {
      if (!appState.dragged || !appState.currentAlarm) return;
      const config = difficultyConfig(appState.currentAlarm.difficulty);
      const delta = e.clientX - appState.dragged.startX;
      let intended = appState.dragged.thumbStart + delta * config.resistance;

      const progressBase = (intended - 8) / (getTrackMaxLeft() - 8 || 1);
      const threat = Math.max(0, Math.min(1, progressBase));

      if (Math.random() < config.escapeChance * (0.35 + threat * 0.95)) {
        intended -= config.leap + Math.random() * config.leap;
      }

      intended += (Math.random() - 0.5) * config.jitter;
      positionThumb(intended);
      const maxLeft = getTrackMaxLeft();
      appState.progress = Math.max(appState.progress, ((appState.currentThumbLeft - 8) / (maxLeft - 8 || 1)) * 100);
      el.trackFill.style.width = `${Math.min(100, appState.progress)}%`;

      if (appState.currentThumbLeft >= maxLeft - 2) {
        completeRunaway();
      }
    }

    function endDrag(e) {
      if (!appState.dragged) return;
      try { el.runawayThumb.releasePointerCapture(e.pointerId); } catch (_) {}
      appState.dragged = null;
      if (appState.currentAlarm && appState.progress < 100) {
        const recoil = Math.max(8, appState.currentThumbLeft - (appState.currentAlarm.difficulty === 'evil' ? 48 : appState.currentAlarm.difficulty === 'normal' ? 26 : 14));
        positionThumb(recoil);
        appState.progress = Math.max(0, appState.progress - (appState.currentAlarm.difficulty === 'evil' ? 18 : 8));
        el.trackFill.style.width = `${appState.progress}%`;
      }
    }

    function completeRunaway() {
      appState.progress = 100;
      el.trackFill.style.width = '100%';
      showToast('捕まえました。お疲れさまです。');
      const current = appState.currentAlarm;
      stopCurrentAlarm();
      if (current && current._temporary) {
        appState.alarms = appState.alarms.filter(a => a.id !== current.id);
        saveState();
        renderAlarms();
      }
    }

    function showToast(message) {
      el.toast.textContent = message;
      el.toast.classList.add('show');
      clearTimeout(showToast.timer);
      showToast.timer = setTimeout(() => el.toast.classList.remove('show'), 1800);
    }

    function removeExpiredTemporaryAlarms() {
      const now = new Date();
      const nowMinutes = now.getHours() * 60 + now.getMinutes();
      appState.alarms = appState.alarms.filter(alarm => {
        if (!alarm._temporary) return true;
        const [h, m] = alarm.time.split(':').map(Number);
        const alarmMinutes = h * 60 + m;
        return alarmMinutes >= nowMinutes - 1;
      });
      saveState();
    }

    function previewAlarm() {
      if (appState.currentAlarm) return;
      const fake = {
        id: uid(),
        name: 'テスト鳴動',
        time: getCurrentHHMM(),
        difficulty: 'normal',
        sound: getSelectedSound ? getSelectedSound() : SOUND_OPTIONS[0].id,
        snoozeMinutes: 5
      };
      appState.currentAlarm = fake;
      appState.previewMode = true;
      el.alarmScreen.classList.add('show');
      el.difficultyBadge.textContent = difficultyLabel(fake.difficulty);
      el.ringTime.textContent = fake.time;
      el.ringName.textContent = fake.name;
      el.ringNote.textContent = 'テスト中です。ちゃんと性格が悪いか確認してください。';
      resetRunawayThumb();
      startAlarmSound(fake.sound);
    }

    function bindEvents() {
      el.addBtn.addEventListener('click', () => openSheet());
      el.previewBtn.addEventListener('click', previewAlarm);
      el.cancelBtn.addEventListener('click', closeSheet);
      el.saveBtn.addEventListener('click', saveAlarm);
      el.sheetBackdrop.addEventListener('click', (e) => {
        if (e.target === el.sheetBackdrop) closeSheet();
      });

      [...el.typeSegment.querySelectorAll('button')].forEach(btn => {
        btn.addEventListener('click', () => setSelectedType(btn.dataset.type));
      });

      [...el.difficultyRow.querySelectorAll('button')].forEach(btn => {
        btn.addEventListener('click', () => setSelectedDifficulty(btn.dataset.level));
      });

      el.runawayThumb.addEventListener('pointerdown', startDrag);
      el.runawayThumb.addEventListener('pointermove', moveDrag);
      el.runawayThumb.addEventListener('pointerup', endDrag);
      el.runawayThumb.addEventListener('pointercancel', endDrag);
      el.snoozeBtn.addEventListener('click', snoozeCurrentAlarm);
      el.giveUpBtn.addEventListener('click', () => showToast('残念です。まだ止まりません。'));

      document.addEventListener('visibilitychange', () => {
        if (document.visibilityState === 'visible') {
          removeExpiredTemporaryAlarms();
        }
      });

      document.body.addEventListener('click', ensureAudioUnlocked, { once: true });
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
      startScheduler();
    }

    boot();
  </script>
</body>
</html>
