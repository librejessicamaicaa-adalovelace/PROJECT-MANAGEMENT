let projects = JSON.parse(localStorage.getItem('corepro_db')) || [
    { id: 1, name: "SALES ORDER", category: "SALES", tasks: [], theme: "navy" }
];
let activeId = projects[0]?.id || Date.now();
let currentFilter = 'ALL', editingProjId = null, editingCatName = null;
let draggedProjId = null, draggedTaskIdx = null;

const save = () => localStorage.setItem('corepro_db', JSON.stringify(projects));

function getCatIcon(cat) {
    const icons = { "SALES": "🛒", "HR": "👥", "TECH": "💻", "FINANCE": "💰", "MARKETING": "📢", "GENERAL": "📁" };
    return icons[cat.toUpperCase()] || "📁";
}

function render() {
    const list = document.getElementById('project-list');
    list.innerHTML = '';
    const categories = [...new Set(projects.map(p => p.category || "GENERAL"))];

    categories.forEach(cat => {
        const head = document.createElement('div');
        head.className = 'category-label';
        if (editingCatName === cat) {
            head.innerHTML = `<span>${getCatIcon(cat)}</span> <input type="text" style="color:var(--accent); font-size:1.4rem; width:70%" value="${cat}" onblur="editingCatName=null; render()" oninput="renameCategory('${cat}', this.value)">`;
            setTimeout(() => head.querySelector('input').focus(), 50);
        } else {
            head.innerHTML = `<div><span>${getCatIcon(cat)}</span> ${cat}</div> <div class="cat-actions"><span style="cursor:pointer" onclick="editingCatName='${cat}'; render()">✎</span><span style="cursor:pointer; margin-left:8px" onclick="deleteCategory('${cat}')">×</span></div>`;
        }
        list.appendChild(head);

        projects.filter(p => (p.category || "GENERAL") === cat).forEach(p => {
            const div = document.createElement('div');
            div.className = `project-item ${p.id === activeId ? 'active' : ''}`;
            div.draggable = true;
            div.ondragstart = () => draggedProjId = p.id;
            div.ondragover = (e) => e.preventDefault();
            div.ondrop = () => {
                const f = projects.findIndex(x => x.id === draggedProjId);
                const t = projects.findIndex(x => x.id === p.id);
                const [moved] = projects.splice(f, 1);
                projects.splice(t, 0, moved);
                save(); render();
            };
            if (editingProjId === p.id) {
                div.innerHTML = `<input type="text" style="color:white" value="${p.name}" onblur="editingProjId=null; render()" oninput="renameProj(${p.id}, this.value)">`;
                setTimeout(() => div.querySelector('input').focus(), 50);
            } else {
                div.innerHTML = `<span>VE| ${p.name}</span><div class="proj-actions"><span onclick="event.stopPropagation(); editingProjId=${p.id}; render()">✎</span><span onclick="event.stopPropagation(); delProj(${p.id})">×</span></div>`;
                div.onclick = () => { activeId = p.id; currentFilter = 'ALL'; render(); };
            }
            list.appendChild(div);
        });
    });

    const activeProj = projects.find(p => p.id === activeId) || projects[0];
    document.body.setAttribute('data-theme', activeProj.theme);
    document.getElementById('active-project-title').innerText = `VE| ${activeProj.name}`;
    document.getElementById('theme-selector').value = activeProj.theme;

    [['pending','PENDING'],['progress','IN PROGRESS'],['review','FOR REVIEW'],['completed','COMPLETED']].forEach(([id, s]) => {
        const count = activeProj.tasks.filter(t => t.status === s).length;
        const pct = activeProj.tasks.length ? Math.round((count/activeProj.tasks.length)*100) : 0;
        document.getElementById(`count-${id}`).innerText = count.toString().padStart(2, '0');
        document.getElementById(`ring-${id}`).style.strokeDashoffset = 264 - (264 * pct) / 100;
        document.getElementById(`percent-${id}`).innerText = pct + '%';
        document.getElementById(`card-${id}`).classList.toggle('active-filter', currentFilter === s);
    });

    const tbody = document.getElementById('task-tbody');
    tbody.innerHTML = '';
    const search = document.getElementById('task-search').value.toUpperCase();
    let filtered = currentFilter === 'ALL' ? [...activeProj.tasks] : activeProj.tasks.filter(t => t.status === currentFilter);
    if(search) filtered = filtered.filter(t => t.name.toUpperCase().includes(search));

    filtered.forEach((t) => {
        const realIdx = activeProj.tasks.indexOf(t);
        const tr = document.createElement('tr');
        tr.draggable = true;
        tr.setAttribute('data-status', t.status);
        tr.ondragstart = () => draggedTaskIdx = realIdx;
        tr.ondragover = (e) => e.preventDefault();
        tr.ondrop = () => {
            const [moved] = activeProj.tasks.splice(draggedTaskIdx, 1);
            activeProj.tasks.splice(realIdx, 0, moved);
            save(); render();
        };
        tr.innerHTML = `<td style="text-align:center; font-weight:900; color:var(--accent)">${realIdx + 1}</td>
            <td><textarea oninput="autoGrow(this)" onchange="up(${realIdx},'name',this.value)">${t.name}</textarea></td>
            <td><input type="text" value="${t.reportedBy}" onchange="up(${realIdx},'reportedBy',this.value)"></td>
            <td><input type="date" value="${t.timeline}" onchange="up(${realIdx},'timeline',this.value)"></td>
            <td><input type="date" value="${t.modified}" onchange="up(${realIdx},'modified',this.value)"></td>
            <td><select onchange="up(${realIdx},'status',this.value); render();">
                <option ${t.status==='PENDING'?'selected':''}>PENDING</option>
                <option ${t.status==='IN PROGRESS'?'selected':''}>IN PROGRESS</option>
                <option ${t.status==='FOR REVIEW'?'selected':''}>FOR REVIEW</option>
                <option ${t.status==='COMPLETED'?'selected':''}>COMPLETED</option>
            </select></td>
            <td><textarea oninput="autoGrow(this)" onchange="up(${realIdx},'remarks',this.value)">${t.remarks || ''}</textarea></td>
            <td><button onclick="delTask(${realIdx})" style="color:red; background:none; border:none; font-size:25px;">×</button></td>`;
        tbody.appendChild(tr);
    });
    document.querySelectorAll('textarea').forEach(autoGrow);
}

function renameCategory(oldCat, newCat) { projects.forEach(p => { if(p.category === oldCat) p.category = newCat.toUpperCase(); }); save(); }
function deleteCategory(cat) { if(confirm("Delete category?")) { projects = projects.filter(p => p.category !== cat); activeId = projects[0]?.id; save(); render(); } }
function renameProj(id, val) { const p = projects.find(x => x.id === id); if(p) { p.name = val.toUpperCase(); save(); } }
function changeTheme(val) { const p = projects.find(x => x.id === activeId); if(p) { p.theme = val; document.body.setAttribute('data-theme', val); save(); render(); } }
function up(i, f, v) { const p = projects.find(x => x.id === activeId); p.tasks[i][f] = v; if(f !== 'modified') p.tasks[i].modified = new Date().toISOString().split('T')[0]; save(); }
function addTask() { const p = projects.find(x => x.id === activeId); const d = new Date().toISOString().split('T')[0]; p.tasks.unshift({name:"", reportedBy:"", timeline:d, modified:d, status:"PENDING", remarks:""}); save(); render(); }
function delTask(i) { if(confirm("Delete task?")) { projects.find(x => x.id === activeId).tasks.splice(i,1); save(); render(); } }
function delProj(id) { if(confirm("Delete workspace?")) { projects = projects.filter(x => x.id !== id); activeId = projects[0]?.id; save(); render(); } }
function exportToExcel() { const activeProj = projects.find(p => p.id === activeId); const ws = XLSX.utils.json_to_sheet(activeProj.tasks); const wb = XLSX.utils.book_new(); XLSX.utils.book_append_sheet(wb, ws, "Tasks"); XLSX.writeFile(wb, `${activeProj.name}.xlsx`); }
function filterByStatus(s) { currentFilter = (currentFilter === s) ? 'ALL' : s; render(); }
function openProjectModal() { document.getElementById('modal-overlay').style.display = 'flex'; }
function closeModal() { document.getElementById('modal-overlay').style.display = 'none'; }
function submitModal() {
    const name = document.getElementById('workspace-input').value.trim().toUpperCase();
    const cat = document.getElementById('category-input').value.trim().toUpperCase() || "GENERAL";
    if(!name) return;
    projects.push({ id: Date.now(), name, category: cat, tasks: [], theme: "navy" });
    activeId = projects[projects.length - 1].id;
    save(); render(); closeModal();
}
function autoGrow(el) { el.style.height = 'auto'; el.style.height = el.scrollHeight + 'px'; }
function handleEnter(e) { if (e.key === "Enter") submitModal(); }
window.onload = render;
