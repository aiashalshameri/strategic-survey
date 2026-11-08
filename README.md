<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>استبيان التحليل الاستراتيجي للبيئة الداخلية للمستشفى</title>
<style>
  * { box-sizing: border-box; }
  body { margin:0; font-family: Tahoma, Arial, sans-serif; background:#f3f4f9; color:#1f2937; line-height:1.7; }
  .wrap { max-width:1100px; margin:0 auto; padding:16px; }
  .head { position:relative; overflow:hidden; background:#4f46e5; color:#fff; border-radius:16px; padding:26px 22px; margin-bottom:14px; }
  .hbg { position:absolute; inset:0; width:100%; height:100%; object-fit:cover; opacity:.35; }
  .htext { position:relative; }
  .head h1 { margin:0 0 8px; font-size:1.35em; }
  .head p { margin:0; font-size:.95em; }
  .bar { height:8px; background:#e5e7eb; border-radius:8px; overflow:hidden; margin-bottom:6px; }
  .fill { height:100%; background:#4f46e5; transition:width .3s; }
  .pinfo { font-size:.9em; color:#6b7280; margin-bottom:14px; }
  .card { background:#fff; border-radius:14px; padding:18px; margin-bottom:14px; box-shadow:0 1px 4px rgba(0,0,0,.08); }
  .dtitle { margin:0 0 14px; color:#4f46e5; font-size:1.2em; font-weight:700; }
  .dim { margin-bottom:22px; }
  .dim-title { font-weight:700; margin-bottom:6px; padding-bottom:6px; border-bottom:2px solid #e5e7eb; }
  .row { display:grid; grid-template-columns:minmax(220px,1.4fr) 2fr; gap:10px; align-items:center; padding:10px 0; border-bottom:1px solid #f0f0f4; }
  .row.missing { background:#fef2f2; }
  .stmt { font-size:.95em; overflow-wrap:anywhere; }
  .opts { display:grid; grid-template-columns:repeat(5, minmax(0,1fr)); gap:6px; }
  .opt { display:flex; flex-direction:column; align-items:center; justify-content:center; text-align:center; font-size:.85em; line-height:1.4; border:1px solid #e5e7eb; border-radius:10px; padding:8px 4px; cursor:pointer; background:#f9fafb; }
  .opt input { margin:0 0 4px; width:18px; height:18px; accent-color:#4f46e5; }
  .opt:has(input:checked) { border-color:#4f46e5; background:#eef2ff; }
  .msg { display:none; background:#fef2f2; color:#b91c1c; padding:10px 14px; border-radius:10px; margin-bottom:12px; }
  .nav { display:flex; justify-content:space-between; gap:10px; margin-top:12px; }
  .btn { background:#4f46e5; color:#fff; border:0; border-radius:10px; padding:12px 24px; font-size:1em; cursor:pointer; }
  .btn.sec { background:#e5e7eb; color:#1f2937; }
  .btn:disabled { opacity:.6; cursor:default; }
  @media (max-width:700px) {
    .row { grid-template-columns:1fr; }
    .opts { grid-template-columns:1fr 1fr; }
  }
</style>
</head>
<body>
<div class="wrap">
  <div class="head">
    <img id="hbg" class="hbg" alt="" style="display:none">
    <div class="htext">
      <h1>استبيان التحليل الاستراتيجي للبيئة الداخلية للمستشفى</h1>
      <p>استبيان لتحديد نقاط القوة ونقاط الضعف في البيئة الداخلية للمستشفى. يرجى قراءة كل عبارة واختيار الإجابة الأقرب إلى واقع المستشفى، واختر «لا أعلم / لا ينطبق» عند عدم توفر معلومات كافية.</p>
    </div>
  </div>
  <div class="bar"><div class="fill" id="fill"></div></div>
  <div class="pinfo" id="pinfo"></div>
  <div id="app"><div class="card">جارٍ تحميل الاستبيان...</div></div>
</div>

<script>
// ===== الإعدادات (الخطوة الوحيدة التي تحتاج لصق) =====
const APPS_URL = 'https://script.google.com/macros/s/AKfycbxV14Jfwt6hQ46taMC2AS0U_MFTA2KVwgjxWhtan6eCFYqZWhdIRmcFGPsQa8Tm35sa3Q/exec';   // ضع رابط /exec هنا
const HEADER_IMAGE_URL = '';               // اختياري: رابط صورة من Imgur

let DATA = null, page = 0;
const answers = {}, missing = {};
const app = document.getElementById('app');

if (HEADER_IMAGE_URL) {
  const img = document.getElementById('hbg');
  img.src = HEADER_IMAGE_URL;
  img.style.display = 'block';
}

// يستدعيها Apps Script بعد تحميل البيانات
function onData(d) { DATA = d; render(); }

const loader = document.createElement('script');
loader.src = APPS_URL + '?callback=onData';
loader.onerror = function () {
  app.innerHTML = '<div class="card">تعذر الاتصال بخادم الاستبيان من هذه الشبكة. جرّب تشغيل تطبيق الحماية (مثل Psiphon) ثم أعد فتح الصفحة.</div>';
};
setTimeout(function () {
  if (!DATA) {
    app.innerHTML = '<div class="card">انتهت مهلة تحميل الاستبيان. تأكد من اتصال الإنترنت أو جرّب تشغيل Psiphon ثم أعد المحاولة.</div>';
  }
}, 15000);
document.body.appendChild(loader);

app.addEventListener('change', function (e) {
  if (e.target.type === 'radio') {
    answers[e.target.name] = e.target.value;
    const row = e.target.closest('.row');
    if (row) row.classList.remove('missing');
  }
});

function esc(s) {
  return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
}
function pageIds(p) {
  const out = [];
  DATA.domains[p].d.forEach(function (dim, di) {
    dim.r.forEach(function (s, ri) { out.push((p+1) + '.' + (di+1) + '.' + (ri+1)); });
  });
  return out;
}
function allIds() {
  let out = [];
  DATA.domains.forEach(function (d, p) { out = out.concat(pageIds(p)); });
  return out;
}

function render() {
  const d = DATA.domains[page];
  let h = '<div class="card"><div class="dtitle">' + esc(d.t) + '</div>';
  d.d.forEach(function (dim, di) {
    const dc = (page+1) + '.' + (di+1);
    h += '<div class="dim"><div class="dim-title">' + dc + ' - ' + esc(dim.t) + '</div>';
    dim.r.forEach(function (st, ri) {
      const id = dc + '.' + (ri+1);
      h += '<div class="row' + (missing[id] ? ' missing' : '') + '" id="r' + id.replace(/\./g,'_') + '">';
      h += '<div class="stmt">' + (ri+1) + '. ' + esc(st) + '</div><div class="opts">';
      DATA.scale.forEach(function (s) {
        const chk = answers[id] === s ? ' checked' : '';
        h += '<label class="opt"><input type="radio" name="' + id + '" value="' + esc(s) + '"' + chk + '><span>' + esc(s) + '</span></label>';
      });
      h += '</div></div>';
    });
    h += '</div>';
  });
  h += '</div>';

  const last = page === DATA.domains.length - 1;
  h += '<div class="nav">' +
    (page > 0 ? '<button class="btn sec" onclick="go(-1)">السابق</button>' : '<span></span>') +
    (last ? '<button class="btn" id="sub" onclick="submitAll()">إرسال الاستبيان</button>'
          : '<button class="btn" onclick="go(1)">التالي</button>') + '</div>';

  app.innerHTML = '<div class="msg" id="msg"></div>' + h;
  document.getElementById('fill').style.width = ((page+1) / DATA.domains.length * 100) + '%';
  document.getElementById('pinfo').textContent = 'القسم ' + (page+1) + ' من ' + DATA.domains.length;
  window.scrollTo(0, 0);
}

function showMsg(t) {
  const m = document.getElementById('msg');
  if (m) { m.textContent = t; m.style.display = 'block'; }
}
function highlight(id) {
  const row = document.getElementById('r' + id.replace(/\./g,'_'));
  if (row) { row.classList.add('missing'); row.scrollIntoView({ behavior:'smooth', block:'center' }); }
}

function go(dir) {
  if (dir > 0) {
    const miss = pageIds(page).filter(function (id) { return !answers[id]; });
    if (miss.length) {
      miss.forEach(function (id) { missing[id] = true; });
      render();
      showMsg('يرجى الإجابة على جميع العبارات في هذا القسم قبل المتابعة (' + miss.length + ' عبارة)');
      highlight(miss[0]);
      return;
    }
  }
  page += dir;
  render();
}

function submitAll() {
  const miss = allIds().filter(function (id) { return !answers[id]; });
  if (miss.length) {
    miss.forEach(function (id) { missing[id] = true; });
    page = parseInt(miss[0].split('.')[0], 10) - 1;
    render();
    showMsg('توجد ' + miss.length + ' عبارة بدون إجابة، تم نقلك إلى أول عبارة ناقصة');
    highlight(miss[0]);
    return;
  }
  const b = document.getElementById('sub');
  b.disabled = true;
  b.textContent = 'جارٍ الإرسال...';
  fetch(APPS_URL, {
    method: 'POST',
    mode: 'no-cors',
    headers: { 'Content-Type': 'text/plain' },
    body: JSON.stringify(answers)
  }).then(function () {
    document.getElementById('fill').parentElement.style.display = 'none';
    document.getElementById('pinfo').style.display = 'none';
    app.innerHTML = '<div class="card" style="text-align:center"><h2>شكراً لمشاركتك</h2><p>تم استلام إجاباتك بنجاح.</p></div>';
    window.scrollTo(0, 0);
  }).catch(function () {
    b.disabled = false;
    b.textContent = 'إرسال الاستبيان';
    showMsg('حدث خطأ أثناء الإرسال، يرجى المحاولة مرة أخرى');
  });
}
</script>
</body>
</html>
