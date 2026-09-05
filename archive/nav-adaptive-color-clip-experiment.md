# Archiviert: Pixelgenaue Nav-Text-Farbe (Clip-Ebenen-Ansatz)

Stand: getestet, funktioniert, aber wieder durch die einfache
Zonen-Crossfade-Lösung ersetzt (zu viel Overhead für den visuellen Effekt).
Diese Datei enthält alles, um es bei Bedarf erneut einzubauen.

## Funktionsweise
Für jedes Element mit `data-nav-theme` (bzw. genauer: jede „Zone" der Seite)
wird eine komplette, geklonte Kopie des Nav-Inhalts als eigene Ebene übereinander
gestapelt. Jede Ebene bekommt per JS (bei Scroll/Resize) eine 2D-Masken-Freistellung
(`mask-image`, zwei Gradienten kombiniert via `mask-composite:intersect`), die
genau dem Bereich entspricht, in dem die zugehörige Seiten-Zone die Nav-Bar
überlappt — inklusive weichem Feather-Übergang (`--feather`). So entsteht ein
pixelgenauer, weicher Wechsel zwischen Weiß und Schwarz, der auch mitten im
Wort/Buchstaben passieren kann (z. B. wenn About Me links dunkel/rechts hell ist).
Hover wird über einen Index (`data-nav-idx`) von den echten (unsichtbaren,
weiterhin klickbaren) Links auf alle passenden Klone gespiegelt.

## HTML (Nav-Markup)
```html
<nav id="nav" data-navbar>
  <div class="nav-blur"></div>
  <div class="nav-content" data-nav-content>
    <div class="nav-meta">
      <a class="tag-box" href="#top" data-nav-color-target>PORTFOLIO&nbsp;&bull;&nbsp;2026</a>
    </div>
    <div class="nav-links">
      <a class="me" href="#about" data-nav-color-target>About Me</a>
      <a href="#projects" data-nav-color-target>Projects</a>
      <a href="#skills" data-nav-color-target>Skills</a>
      <a href="#cv" data-nav-color-target>CV</a>
      <a href="#contact" data-nav-color-target>Contact</a>
    </div>
  </div>
</nav>
```

## Theme-Zuweisung auf den Sections
```html
<header class="hero" data-nav-theme="dark">...</header>
<section id="about" class="split">
  <div class="blue" data-nav-theme="dark">...</div>
  <div class="media reveal" data-nav-theme="light"><img ...></div>
</section>
<section id="interests" class="shell interests" data-nav-theme="dark">...</section>
<section id="projects" class="shell" data-nav-theme="light">...</section>
<section id="skills" class="shell" data-nav-theme="light">...</section>
<section id="cv" class="shell resume" data-nav-theme="light">...</section>
<section id="contact" class="contact" data-nav-theme="light">...</section>
```
Wichtig: Theme-Attribut auf die kleinste Einheit setzen, die farblich
einheitlich ist (siehe About Me: nicht auf `#about`, sondern getrennt auf
`.blue` und `.media`, weil die Section intern zweigeteilt ist).

## CSS
```css
nav{
  position:fixed;top:16px;left:clamp(8px,1.5vw,20px);right:clamp(8px,1.5vw,20px);z-index:100;
  padding:14px clamp(20px,3vw,36px);
  border-radius:10px;
}
.nav-blur{
  position:absolute;inset:0;z-index:0;border-radius:inherit;
  background:rgba(250,249,245,.2);
  backdrop-filter:blur(8px);-webkit-backdrop-filter:blur(8px);
  box-shadow:0 4px 12px rgba(23,23,29,.04);
  pointer-events:none;
}
.nav-content{position:relative;z-index:1;display:flex;justify-content:space-between;align-items:center;gap:16px;width:100%}
.nav-meta,.nav-links{position:relative}
.nav-meta{font-size:.72rem;letter-spacing:.05em;display:flex;gap:14px;align-items:center;white-space:nowrap}
.nav-meta .tag-box{
  font-family:'Archivo',sans-serif;font-weight:500;font-size:.85rem;
  letter-spacing:-.02em;text-transform:uppercase;
  color:inherit;display:inline-block;transition:opacity .15s;
}
.nav-meta a.tag-box:hover{opacity:.7}
.brand{font-family:'Archivo';font-weight:800;font-size:1.15rem;letter-spacing:.01em}
.nav-links{display:flex;gap:22px;justify-content:flex-end;align-items:center}
.nav-links a{
  font-family:'Archivo',sans-serif;font-weight:500;font-size:.85rem;
  letter-spacing:-.02em;text-transform:uppercase;
  color:inherit;transition:opacity .15s;
}
.nav-links a:hover{opacity:.7}

/* clip-based per-section nav text color */
.nav-content [data-nav-color-target]{color:transparent!important}
.nav-color-stack{
  position:absolute;inset:0;z-index:2;overflow:hidden;pointer-events:none;
  box-sizing:border-box;padding:14px clamp(20px,3vw,36px);
  display:grid;align-items:center;
  background:transparent!important;border:0!important;box-shadow:none!important;
  backdrop-filter:none!important;-webkit-backdrop-filter:none!important;
}
.nav-color-layer{
  --feather:16px;
  grid-area:1/1;width:100%;height:100%;overflow:hidden;pointer-events:none;
  color:var(--nav-text-color);
  mask-image:
    linear-gradient(to bottom,
      transparent 0,
      transparent calc(var(--clip-top,100%) - var(--feather)),
      #000 var(--clip-top,100%),
      #000 calc(100% - var(--clip-bottom,0px)),
      transparent calc(100% - var(--clip-bottom,0px) + var(--feather)),
      transparent 100%),
    linear-gradient(to right,
      transparent 0,
      transparent calc(var(--clip-left,0px) - var(--feather)),
      #000 var(--clip-left,0px),
      #000 calc(100% - var(--clip-right,0px)),
      transparent calc(100% - var(--clip-right,0px) + var(--feather)),
      transparent 100%);
  mask-composite:intersect;
  -webkit-mask-image:
    linear-gradient(to bottom,
      transparent 0,
      transparent calc(var(--clip-top,100%) - var(--feather)),
      #000 var(--clip-top,100%),
      #000 calc(100% - var(--clip-bottom,0px)),
      transparent calc(100% - var(--clip-bottom,0px) + var(--feather)),
      transparent 100%),
    linear-gradient(to right,
      transparent 0,
      transparent calc(var(--clip-left,0px) - var(--feather)),
      #000 var(--clip-left,0px),
      #000 calc(100% - var(--clip-right,0px)),
      transparent calc(100% - var(--clip-right,0px) + var(--feather)),
      transparent 100%);
  -webkit-mask-composite:source-in;
  will-change:mask-image;
}
.nav-color-layer .nav-content{width:100%;height:100%}
.nav-color-layer .nav-content *{visibility:hidden}
.nav-color-layer [data-nav-color-target],
.nav-color-layer [data-nav-color-target] *{
  visibility:visible;color:var(--nav-text-color)!important;
  transition:color .2s ease;
}
.nav-color-layer [data-nav-color-target].nav-hover,
.nav-color-layer [data-nav-color-target].nav-hover *{
  color:var(--blue)!important;
}

@media(max-width:900px){
  .nav-content{justify-content:flex-end}
  .nav-meta{display:none}
  .nav-links{gap:14px}
  .nav-links a.me{display:none}
}
```

## JavaScript
```js
(function(){
  const navEl=document.querySelector('[data-navbar]');
  if(!navEl)return;
  const content=navEl.querySelector('[data-nav-content]');
  if(!content)return;

  const sections=Array.from(document.querySelectorAll('[data-nav-theme]'));
  if(!sections.length)return;

  // tag real targets with a stable index so the clones (copied via
  // cloneNode below, attributes and all) can be matched back up for hover
  const realTargets=Array.from(content.querySelectorAll('[data-nav-color-target]'));
  realTargets.forEach((el,i)=>el.setAttribute('data-nav-idx',i));

  const stack=document.createElement('div');
  stack.className='nav-color-stack';
  stack.setAttribute('aria-hidden','true');
  stack.inert=true;
  navEl.appendChild(stack);

  const layers=sections.map(section=>{
    const layer=document.createElement('div');
    layer.className='nav-color-layer';
    layer.setAttribute('aria-hidden','true');
    layer.inert=true;
    const clone=content.cloneNode(true);
    clone.querySelectorAll('*').forEach(el=>{
      el.removeAttribute('id');
      if(el.matches('a,button,input,select,textarea'))el.setAttribute('tabindex','-1');
    });
    const textColor=section.dataset.navTheme==='dark' ? '#fff' : '#000';
    layer.style.setProperty('--nav-text-color',textColor);
    layer.appendChild(clone);
    stack.appendChild(layer);
    return {section,layer};
  });

  // hover: the real (invisible) links stay clickable/keyboard-focusable and
  // catch the pointer, so on hover just flip the matching clone(s) blue
  realTargets.forEach(el=>{
    const idx=el.getAttribute('data-nav-idx');
    const clones=stack.querySelectorAll(`[data-nav-idx="${idx}"]`);
    const setHover=on=>clones.forEach(c=>c.classList.toggle('nav-hover',on));
    el.addEventListener('mouseenter',()=>setHover(true));
    el.addEventListener('mouseleave',()=>setHover(false));
    el.addEventListener('focus',()=>setHover(true));
    el.addEventListener('blur',()=>setHover(false));
  });

  let pending=false;
  function update(){
    pending=false;
    const navRect=content.getBoundingClientRect();
    const navH=navRect.height,navW=navRect.width;
    layers.forEach(({section,layer})=>{
      const r=section.getBoundingClientRect();
      const top=Math.max(navRect.top,r.top);
      const bottom=Math.min(navRect.bottom,r.bottom);
      const left=Math.max(navRect.left,r.left);
      const right=Math.min(navRect.right,r.right);
      const h=Math.max(0,bottom-top);
      const w=Math.max(0,right-left);
      if(h<=0||w<=0){
        // push fully outside the 0–100% range so the mask's feather ramp
        // (which lives right at the clip edge) can't bleed into view
        const FAR=100;
        layer.style.setProperty('--clip-top',`${navH+FAR}px`);
        layer.style.setProperty('--clip-bottom',`${-FAR}px`);
        layer.style.setProperty('--clip-left',`${navW+FAR}px`);
        layer.style.setProperty('--clip-right',`${-FAR}px`);
        return;
      }
      layer.style.setProperty('--clip-top',`${Math.max(0,top-navRect.top)}px`);
      layer.style.setProperty('--clip-bottom',`${Math.max(0,navRect.bottom-bottom)}px`);
      layer.style.setProperty('--clip-left',`${Math.max(0,left-navRect.left)}px`);
      layer.style.setProperty('--clip-right',`${Math.max(0,navRect.right-right)}px`);
    });
  }
  function schedule(){
    if(pending)return;
    pending=true;
    requestAnimationFrame(update);
  }
  addEventListener('scroll',schedule,{passive:true});
  addEventListener('resize',schedule);
  new ResizeObserver(schedule).observe(navEl);
  sections.forEach(s=>new ResizeObserver(schedule).observe(s));
  schedule();
})();
```

## Bekannte Stolperfallen (bereits gelöst, falls wieder aktiviert)
- `.nav-color-stack`/`.nav-color-layer` müssen dasselbe Padding wie `nav`
  bekommen (`box-sizing:border-box`), sonst ignorieren die geklonten Ebenen
  das Nav-Padding und der Text klebt am Rand.
- Inaktive Ebenen müssen ihre Clip-Werte weit außerhalb 0–100% setzen
  (nicht exakt auf den Rand), sonst blutet der Feather-Gradient am Rand in
  den sichtbaren Bereich (sichtbar z. B. an den letzten Buchstaben von
  "Contact").
- `mask-composite:intersect` (Standard) + `-webkit-mask-composite:source-in`
  (Safari) werden für die 2D-Kombination aus vertikalem und horizontalem
  Gradient gebraucht.
