# Public Gists by Rich Lewis

<style>
:root { --line:#e5e7eb; --bg:#ffffff; --bg-alt:#fafafa; --text:#0f172a; --muted:#64748b; --hint:#94a3b8; }
.controls { display:flex; gap:.5rem; align-items:center; margin:.5rem 0 1rem; flex-wrap: wrap; }
.controls input[type="search"]{ padding:.45rem .6rem; width:260px; border:1px solid var(--line); border-radius:.5rem; }
.badge { display:inline-block; padding:.15rem .45rem; border-radius:.5rem; background:#e8efff; color:#1d4ed8; font-size:.75rem; line-height:1; }
.table-wrap { overflow-x:auto; }
table { width:100%; border-collapse:collapse; font-size:.95rem; background:var(--bg); }
thead th { position:sticky; top:0; background:var(--bg); text-align:left; border-bottom:1px solid var(--line); padding:.55rem .6rem; }
tbody td { border-top:1px solid var(--line); padding:.5rem .6rem; vertical-align:top; }
tbody tr:nth-child(even){ background:var(--bg-alt); }
th.sortable { cursor:pointer; }
th.sortable::after { content:" \2195"; color:var(--hint); font-weight:normal; }
small.muted{ color:var(--muted); }
</style>
_Auto-generated at 2025-08-28 10:48 PM EDT_

**Last updated:** 2025-08-28 10:48 PM EDT

<div class="controls">
  <input id="filter" type="search" placeholder="Filter by title or language…">
  <span class="badge" id="count"></span>
</div>
<div class="table-wrap" markdown="1">
| Title | Files | Lang | Public | Updated | Link |
|---|---:|---|:---:|---|---|
| Auto-generated index of my PUBLIC gists | 1 | <span class="badge">Markdown</span> | ✅ | 2025-08-28 10:42 PM EDT | [open](https://gist.github.com/RichLewis007/a48c0ac6b651a36724ce6314d5242c74) |
| 3D Objects in GitHub Markdown files | 1 | <span class="badge">Markdown</span> | ✅ | 2025-08-12 11:48 AM EDT | [open](https://gist.github.com/RichLewis007/31093ca3c017021f1e502edd96bf89e7) |
| Bookmarklet to find the font of the text on a web page | 1 | <span class="badge">Markdown</span> | ✅ | 2025-08-12 01:48 AM EDT | [open](https://gist.github.com/RichLewis007/45384ad7d26361b85d8acbd2127a48fe) |
| (no description) | 1 | <span class="badge">Python</span> | ✅ | 2025-08-05 02:32 AM EDT | [open](https://gist.github.com/RichLewis007/e17ee64d75a3310518a50b3109211284) |
| 🔄 Custom React Hook – useFetch | 1 | <span class="badge">Markdown</span> | ✅ | 2025-08-02 03:18 AM EDT | [open](https://gist.github.com/RichLewis007/94dc04cd0150766bff8cd23c984843c0) |
| 🧠 TensorFlow ConvNet | 1 | <span class="badge">Markdown</span> | ✅ | 2025-08-02 03:18 AM EDT | [open](https://gist.github.com/RichLewis007/33acea5d8a3ff20ffe01918e73551b83) |
| 🔄 Custom React Hook – useFetch | 1 | <span class="badge">Markdown</span> | ✅ | 2025-08-02 03:14 AM EDT | [open](https://gist.github.com/RichLewis007/5a8690880627e4b66fe231ba5691fe18) |
| 🧠 TensorFlow ConvNet | 1 | <span class="badge">Markdown</span> | ✅ | 2025-08-02 03:14 AM EDT | [open](https://gist.github.com/RichLewis007/39c9c5bcf59037c030a84501212a0733) |
</div>

_Generated automatically by [gist-index workflow](https://github.com/RichLewis007/gist-index). Updated daily at 9:00 AM Eastern._

<script>
(function(){
  const q=document.getElementById('filter');
  const table=document.querySelector('table'); if(!table) return;
  const tbody=table.tBodies[0]; const rows=[...tbody.rows];
  const ths=table.tHead? table.tHead.rows[0].cells : [];
  const count=document.getElementById('count');
  // mark sortable columns: Title(0), Files(1), Lang(2), Updated(4)
  [0,1,2,4].forEach(i=>ths[i] && ths[i].classList.add('sortable'));

  function applyFilter(){
    const term=(q?.value||"").toLowerCase();
    let visible=0;
    rows.forEach(tr=>{
      const text=tr.textContent.toLowerCase();
      const show=!term || text.includes(term);
      tr.style.display=show?"":"none";
      if(show) visible++;
    });
    if(count){ count.textContent = visible + " gists"; }
  }
  q && q.addEventListener('input', applyFilter);
  applyFilter();

  function cellVal(tr,i){
    const t=(tr.cells[i]?.textContent||"").trim();
    return t;
  }

  function sortBy(idx){
    let asc = !ths[idx].dataset.desc;
    rows.sort((a,b)=>{
      const A=cellVal(a,idx), B=cellVal(b,idx);
      // try date
      if(!isNaN(Date.parse(A)) && !isNaN(Date.parse(B))){
        return asc ? (new Date(A)-new Date(B)) : (new Date(B)-new Date(A));
      }
      // try number
      const nA=parseFloat(A), nB=parseFloat(B);
      if(!isNaN(nA) && !isNaN(nB)){ return asc ? (nA-nB) : (nB-nA); }
      // fallback string
      return asc ? A.localeCompare(B) : B.localeCompare(A);
    });
    rows.forEach(r=>tbody.appendChild(r));
    // toggle state
    ths[idx].dataset.desc = asc ? "" : "1";
  }

  [0,1,2,4].forEach(i=>{
    if(!ths[i]) return;
    ths[i].addEventListener('click', ()=>sortBy(i));
  });

  // Re-apply filter after sort to keep count correct
  table.addEventListener('click', e=>{
    if(e.target.closest('th')) setTimeout(applyFilter, 0);
  });

  // Support Material's client-side swaps if this ever sits behind it
  if(window.document$){ window.document$.subscribe(applyFilter); }
})();
</script>
