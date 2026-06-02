<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Lista de Alunos</title>

<style>
:root{
    --primary:#2563eb;
    --primary-dark:#1d4ed8;
    --bg:#f5f7fb;
    --card:#ffffff;
    --border:#e5e7eb;
    --text:#111827;
    --muted:#6b7280;
    --success:#16a34a;
}

*{
    margin:0;
    padding:0;
    box-sizing:border-box;
}

body{
    font-family:Arial, Helvetica, sans-serif;
    background:var(--bg);
    color:var(--text);
    min-height:100vh;
}

.container{
    max-width:900px;
    margin:40px auto;
    padding:20px;
}

.card{
    background:var(--card);
    border-radius:16px;
    padding:24px;
    box-shadow:0 10px 30px rgba(0,0,0,0.08);
}

h1{
    text-align:center;
    margin-bottom:24px;
    font-size:2rem;
}

.form-row{
    display:flex;
    gap:12px;
    margin-bottom:24px;
}

input{
    flex:1;
    padding:14px;
    border:1px solid var(--border);
    border-radius:10px;
    font-size:16px;
    outline:none;
}

input:focus{
    border-color:var(--primary);
}

button{
    padding:14px 24px;
    border:none;
    border-radius:10px;
    background:var(--primary);
    color:white;
    cursor:pointer;
    font-size:16px;
    font-weight:bold;
    transition:0.2s;
}

button:hover{
    background:var(--primary-dark);
}

.status{
    margin-bottom:16px;
    color:var(--success);
    font-weight:bold;
    min-height:24px;
}

.list-container{
    border:1px solid var(--border);
    border-radius:12px;
    overflow:hidden;
}

.list-header{
    background:#f1f5f9;
    padding:14px;
    font-weight:bold;
}

.student-row{
    padding:14px;
}

.student-row:nth-child(even){
    background:#f8fafc;
}

.student-row:nth-child(odd){
    background:white;
}

.empty{
    padding:20px;
    text-align:center;
    color:var(--muted);
}

.loading{
    text-align:center;
    padding:20px;
    color:var(--muted);
}

@media (max-width:600px){
    .form-row{
        flex-direction:column;
    }

    button{
        width:100%;
    }

    h1{
        font-size:1.6rem;
    }
}
</style>
</head>
<body>

<div class="container">
    <div class="card">

        <h1>Lista de Alunos</h1>

        <div class="form-row">
            <input
                type="text"
                id="studentName"
                placeholder="Digite o nome do aluno"
            >
            <button onclick="saveStudent()">
                Salvar
            </button>
        </div>

        <div id="status" class="status"></div>

        <div class="list-container">
            <div class="list-header">
                Alunos
            </div>

            <div id="studentsList" class="loading">
                Carregando...
            </div>
        </div>

    </div>
</div>

<script>

/*
====================================================
CONFIGURAÇÃO
Cole aqui a URL do seu Web App do Google Apps Script
====================================================
*/
const CONFIG = {
    WEB_APP_URL: "COLE_AQUI_SUA_URL_DO_WEB_APP"
};

let jsonpCounter = 0;

/*
==================================
JSONP Helper
==================================
*/
function jsonp(url, callback) {

    const callbackName =
        "__jsonpCallback_" +
        Date.now() +
        "_" +
        (++jsonpCounter);

    window[callbackName] = function(data){
        callback(data);

        delete window[callbackName];

        script.remove();
    };

    const script = document.createElement("script");

    script.src =
        url +
        (url.includes("?") ? "&" : "?") +
        "callback=" +
        callbackName;

    document.body.appendChild(script);
}

/*
==================================
Carregar lista
==================================
*/
function loadStudents() {

    const url =
        CONFIG.WEB_APP_URL +
        "?action=list";

    jsonp(url, function(response){

        const container =
            document.getElementById("studentsList");

        if (!response.success) {
            container.innerHTML =
                "<div class='empty'>Erro ao carregar.</div>";
            return;
        }

        const students = response.students || [];

        if (students.length === 0) {
            container.innerHTML =
                "<div class='empty'>Nenhum aluno cadastrado.</div>";
            return;
        }

        let html = "";

        students.forEach(student => {
            html += `
                <div class="student-row">
                    ${escapeHtml(student.nome)}
                </div>
            `;
        });

        container.innerHTML = html;
    });
}

/*
==================================
Salvar aluno
==================================
*/
function saveStudent() {

    const input =
        document.getElementById("studentName");

    const name =
        input.value.trim();

    if (!name) {
        alert("Digite o nome do aluno.");
        return;
    }

    const status =
        document.getElementById("status");

    status.textContent = "Salvando...";

    const url =
        CONFIG.WEB_APP_URL +
        "?action=add" +
        "&name=" +
        encodeURIComponent(name);

    jsonp(url, function(response){

        if(response.success){

            status.textContent =
                "Aluno salvo com sucesso.";

            input.value = "";

            loadStudents();

        } else {

            status.textContent =
                "Erro ao salvar.";
        }

        setTimeout(() => {
            status.textContent = "";
        }, 3000);
    });
}

/*
==================================
Escape HTML
==================================
*/
function escapeHtml(text){

    const div =
        document.createElement("div");

    div.textContent = text;

    return div.innerHTML;
}

/*
==================================
Enter para salvar
==================================
*/
document
.getElementById("studentName")
.addEventListener("keypress", function(e){

    if(e.key === "Enter"){
        saveStudent();
    }
});

/*
==================================
Inicialização
==================================
*/
loadStudents();

</script>

</body>
</html>
