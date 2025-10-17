<!doctype html>
<html lang="kk">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Квадраттық (толымсыз) теңдеулер шешуші</title>

  <!-- Simple modern styling -->
  <style>
    :root{
      --bg:#0f172a; --card:#0b1220; --accent:#7c3aed; --muted:#94a3b8; --glass: rgba(255,255,255,0.03);
    }
    *{box-sizing:border-box}
    body{
      margin:0;
      font-family:Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
      background: linear-gradient(180deg, #071022 0%, #071028 60%);
      color:#e6eef8;
      min-height:100vh;
      display:flex;
      align-items:center;
      justify-content:center;
      padding:24px;
    }
    .container{
      width:100%;
      max-width:920px;
      background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
      border-radius:14px;
      padding:22px;
      box-shadow:0 10px 30px rgba(2,6,23,0.7);
      border:1px solid rgba(255,255,255,0.03);
    }
    h1{margin:0 0 6px 0;font-size:20px}
    p.lead{margin:0 0 18px 0;color:var(--muted)}
    .row{display:flex;gap:12px;flex-wrap:wrap}
    label{font-size:13px;color:var(--muted)}
    .input{
      background:var(--glass);
      border:1px solid rgba(255,255,255,0.03);
      padding:10px 12px;border-radius:8px;color:inherit;
      min-width:0;
      width:120px;
      outline:none;
      font-size:16px;
    }
    .controls{margin-top:12px;display:flex;gap:10px;align-items:center}
    button{
      background:var(--accent);border:0;padding:10px 14px;border-radius:10px;color:white;font-weight:600;
      cursor:pointer;
    }
    button.secondary{background:transparent;border:1px solid rgba(255,255,255,0.06);color:var(--muted);font-weight:600}
    .card{background:rgba(255,255,255,0.02);padding:14px;border-radius:10px;margin-top:16px;border:1px solid rgba(255,255,255,0.02)}
    pre{white-space:pre-wrap;font-family:inherit}
    .step{margin-bottom:8px;padding:8px 10px;border-radius:8px;background:linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.005))}
    .small{font-size:13px;color:var(--muted)}
    footer{margin-top:14px;font-size:13px;color:var(--muted)}
    @media (max-width:560px){
      .input{width:100%}
      .row{flex-direction:column}
    }
  </style>

  <!-- MathJax for nicer math rendering (optional) -->
  <script>
    window.MathJax = {
      tex: {inlineMath: [['$','$'], ['\\(','\\)']]},
      options: {skipHtmlTags: ['script','style','textarea','pre']}
    };
  </script>
  <script src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js" defer></script>
</head>
<body>
  <div class="container" role="main" aria-labelledby="title">
    <h1 id="title">Квадраттық (толымсыз) теңдеулер шешуші</h1>
    <p class="lead">Теңдеуді $ax^2 + bx + c = 0$ түрінде бер. Қажет болса бір коэффициентті нөлге қойып «толымсыз» (мысалы $bx$ жоқ немесе $c$ жоқ) теңдеуді шешуге болады. Сайт шешім жолын қадамдап көрсетеді.</p>

    <div class="card" aria-live="polite">
      <div class="row" style="align-items:end">
        <div>
          <label for="a">a (x² коэффициенті)</label><br>
          <input id="a" class="input" type="number" step="any" value="1" />
        </div>
        <div>
          <label for="b">b (x коэффициенті)</label><br>
          <input id="b" class="input" type="number" step="any" value="0" />
        </div>
        <div>
          <label for="c">c (тұрақты)</label><br>
          <input id="c" class="input" type="number" step="any" value="-4" />
        </div>
        <div style="flex:1;min-width:160px">
          <label>Теңдеу көрінісі</label>
          <div id="equation" class="small" style="padding:8px 10px;border-radius:8px;background:rgba(255,255,255,0.01);margin-top:6px">---</div>
        </div>
      </div>

      <div class="controls">
        <button id="solveBtn">Шешу (шешімді көрсет)</button>
        <button id="clearBtn" class="secondary">Тазалау</button>
        <div style="margin-left:auto" class="small">Нұсқа: a=0 болғанда теңдеу сызықты болады.</div>
      </div>

      <div id="output" class="card" style="display:none">
        <div id="steps"></div>
        <hr style="border:none;height:1px;background:rgba(255,255,255,0.03);margin:12px 0">
        <div id="result" style="font-weight:700"></div>
      </div>

      <footer>Қолдануға дайын. Файлды сақтап браузерде ашып көр.</footer>
    </div>
  </div>

<script>
  const aIn = document.getElementById('a');
  const bIn = document.getElementById('b');
  const cIn = document.getElementById('c');
  const equationDiv = document.getElementById('equation');
  const solveBtn = document.getElementById('solveBtn');
  const clearBtn = document.getElementById('clearBtn');
  const output = document.getElementById('output');
  const stepsDiv = document.getElementById('steps');
  const resultDiv = document.getElementById('result');

  function formatNumber(x){
    if (Number.isInteger(x)) return String(x);
    return (Math.round(x*1000000)/1000000).toString();
  }

  function updateEquation(){
    const a = parseFloat(aIn.value) || 0;
    const b = parseFloat(bIn.value) || 0;
    const c = parseFloat(cIn.value) || 0;

    // build string like "2x^2 + 3x - 4 = 0" but nicer
    function term(coef, power){
      if (coef === 0) return '';
      const abs = Math.abs(coef);
      const sign = coef > 0 ? '+' : '-';
      let coefStr = (abs === 1 && power !== 0) ? '' : formatNumber(abs);
      if (power === 2) return `${sign} ${coefStr}x^2`;
      if (power === 1) return `${sign} ${coefStr}x`;
      return `${sign} ${coefStr}`;
    }
    let parts = [];
    if (a !== 0) parts.push((a === -1 ? '- ' : (a === 1 ? '' : formatNumber(a))) + 'x^2');
    if (b !== 0) parts.push((b === -1 ? '- ' : (b === 1 ? '' : formatNumber(b))) + 'x');
    if (c !== 0) parts.push(formatNumber(c));
    let expr = parts.join(' + ');
    expr = expr.replace(/\+\s-\s/g, '- ');
    if (!expr) expr = '0';
    equationDiv.innerText = expr + ' = 0';
  }

  aIn.addEventListener('input', updateEquation);
  bIn.addEventListener('input', updateEquation);
  cIn.addEventListener('input', updateEquation);
  updateEquation();

  clearBtn.addEventListener('click', ()=>{
    aIn.value = '1'; bIn.value = '0'; cIn.value = '-4';
    updateEquation();
    output.style.display = 'none';
    stepsDiv.innerHTML = ''; resultDiv.innerHTML = '';
  });

  function addStep(html){
    const el = document.createElement('div');
    el.className = 'step';
    el.innerHTML = html;
    stepsDiv.appendChild(el);
  }

  function renderMath(){
    if (window.MathJax && window.MathJax.typesetPromise){
      MathJax.typesetClear();
      MathJax.typesetPromise();
    }
  }

  solveBtn.addEventListener('click', ()=>{
    stepsDiv.innerHTML = '';
    resultDiv.innerHTML = '';
    output.style.display = 'block';

    const a = parseFloat(aIn.value);
    const b = parseFloat(bIn.value);
    const c = parseFloat(cIn.value);

    // input validation
    if (isNaN(a) || isNaN(b) || isNaN(c)){
      addStep('<span style="color:#ffb4a2">Қате: a, b, c сан болуы керек.</span>');
      return;
    }

    // Display original equation
    addStep(`<div class="small">Берілген теңдеу:</div> \\( ${a}x^2 ${b>=0?'+':'-'} ${Math.abs(b)}x ${c>=0?'+':'-'} ${Math.abs(c)} = 0 \\)`);

    // Case a == 0 => linear
    if (Math.abs(a) < 1e-12){
      addStep('<div class="small">Бұл сызықты теңдеу (a = 0).</div>');
      if (Math.abs(b) < 1e-12){
        if (Math.abs(c) < 1e-12){
          addStep('Барлық коэффициенттер нөл: теңдеу барлық x үшін орындалады (шексіз шешім).');
          resultDiv.innerHTML = 'Шешім: барлық нақты x (шексіз көпшілік).';
        } else {
          addStep('b = 0 және c ≠ 0: тұрақты теңдеу c = 0 емес, шешімі жоқ.');
          resultDiv.innerHTML = 'Шешім: жоқ (шектелмеген теңсіздік).';
        }
      } else {
        // bx + c = 0 => x = -c/b
        const root = -c / b;
        addStep(`Сызықты теңдеуді шешу: \\( bx + c = 0 \\) -> \\( x = -\\dfrac{c}{b} = -\\dfrac{${formatNumber(c)}}{${formatNumber(b)}} = ${formatNumber(root)} \\)`);
        resultDiv.innerHTML = `Шешім: x = ${formatNumber(root)}`;
      }
      renderMath();
      return;
    }

    // Quadratic
    // Identify incomplete types
    const bZero = Math.abs(b) < 1e-12;
    const cZero = Math.abs(c) < 1e-12;

    if (bZero && cZero){
      // ax^2 = 0 => x = 0 (double root)
      addStep('Екі коэффициент b = 0 және c = 0 болғандықтан: \\( ax^2 = 0 \\) -> \\( x = 0 \\) (екі еселенген түбір).');
      resultDiv.innerHTML = 'Шешім: x = 0 (екі еселенген түбір).';
      renderMath();
      return;
    }

    if (bZero){
      // ax^2 + c = 0 => ax^2 = -c => x^2 = -c/a => x = ± sqrt(-c/a)
      addStep('Толымсыз түрі: \\( b = 0 \\). Теңдеу: \\( ax^2 + c = 0 \\) => \\( ax^2 = -c \\) => \\( x^2 = -\\dfrac{c}{a} \\).');
      const inside = -c / a;
      addStep(`Одан кейін \\( x = \\pm\\sqrt{-\\dfrac{c}{a}} = \\pm\\sqrt{${formatNumber(inside)}} \\).`);
      if (inside >= 0){
        const r = Math.sqrt(inside);
        addStep(`Науқыс: \\( \\sqrt{${formatNumber(inside)}} = ${formatNumber(r)} \\).`);
        resultDiv.innerHTML = `Шешім: x = ${formatNumber(r)} және x = ${formatNumber(-r)}`;
      } else {
        // complex roots
        const r = Math.sqrt(Math.abs(inside));
        addStep('Теңдеу нақты түбір таппайды — кешенді түбірлер бар:');
        addStep(`\\( x = \\pm i \\sqrt{${formatNumber(Math.abs(inside))}} = ${formatNumber(r)} i \\)`);
        resultDiv.innerHTML = `Шешім (кешенді): x = ${formatNumber(r)}i және x = ${formatNumber(-r)}i`;
      }
      renderMath();
      return;
    }

    if (cZero){
      // ax^2 + bx = 0 => x(ax + b) = 0 => x=0, x = -b/a
      addStep('Толымсыз түрі: \\( c = 0 \\). Теңдеу: \\( ax^2 + bx = 0 \\) бұл бөлшектеуге (факторизация) болады:');
      addStep('\\( ax^2 + bx = x(ax + b) = 0 \\).');
      const root1 = 0;
      const root2 = -b / a;
      addStep(`Одан соң \\( x = 0 \\) немесе \\( ax + b = 0 \\Rightarrow x = -\\dfrac{b}{a} = -\\dfrac{${formatNumber(b)}}{${formatNumber(a)}} = ${formatNumber(root2)} \\).`);
      resultDiv.innerHTML = `Шешім: x = 0 және x = ${formatNumber(root2)}`;
      renderMath();
      return;
    }

    // General quadratic (full)
    addStep('Толық квадраттық теңдеу. Қадамдар: дискриминант және квадраттық формула.');
    addStep('Дискриминант: \\( D = b^2 - 4ac \\).');
    const D = b*b - 4*a*c;
    addStep(`Есептейік: \\( D = (${formatNumber(b)})^2 - 4 \\cdot ${formatNumber(a)} \\cdot ${formatNumber(c)} = ${formatNumber(D)} \\).`);

    if (D > 0){
      addStep('D > 0, екі әртүрлі нақты түбір бар. Түбірлер формуласы: \\( x = \\dfrac{-b \\pm \\sqrt{D}}{2a} \\).');
      const sqrtD = Math.sqrt(D);
      const x1 = (-b + sqrtD) / (2*a);
      const x2 = (-b - sqrtD) / (2*a);
      addStep(`\\( \\sqrt{D} = ${formatNumber(sqrtD)} \\).`);
      addStep(`Сонда \\( x_1 = \\dfrac{-${formatNumber(b)} + ${formatNumber(sqrtD)}}{2\\cdot ${formatNumber(a)}} = ${formatNumber(x1)} \\).`);
      addStep(`Сонда \\( x_2 = \\dfrac{-${formatNumber(b)} - ${formatNumber(sqrtD)}}{2\\cdot ${formatNumber(a)}} = ${formatNumber(x2)} \\).`);
      resultDiv.innerHTML = `Екі нақты түбір: x₁ = ${formatNumber(x1)}, x₂ = ${formatNumber(x2)}.`;
    } else if (Math.abs(D) < 1e-12){
      addStep('D = 0, екі еселенген нақты түбір бар (екі бірдей): \\( x = -\\dfrac{b}{2a} \\).');
      const x = -b / (2*a);
      addStep(`\\( x = -\\dfrac{${formatNumber(b)}}{2\\cdot ${formatNumber(a)}} = ${formatNumber(x)} \\).`);
      resultDiv.innerHTML = `Екі еселенген түбір: x = ${formatNumber(x)} (екі рет).`;
    } else {
      addStep('D < 0, нақты түбірлер жоқ — кешенді түбірлер бар.');
      const sqrtAbsD = Math.sqrt(Math.abs(D));
      const real = -b / (2*a);
      const imag = sqrtAbsD / (2*a);
      addStep(`Кешенді түбірлер: \\( x = \\dfrac{-b}{2a} \\pm i\\dfrac{\\sqrt{|D|}}{2a} \\).`);
      addStep(`Сонда \\( x = ${formatNumber(real)} \\pm ${formatNumber(imag)} i \\).`);
      resultDiv.innerHTML = `Кешенді түбірлер: x = ${formatNumber(real)} + ${formatNumber(imag)}i және x = ${formatNumber(real)} - ${formatNumber(imag)}i.`;
    }

    renderMath();
  });

  // initial render
  updateEquation();
</script>
</body>
</html>

