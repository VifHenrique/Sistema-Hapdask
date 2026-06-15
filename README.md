# HelpDesk Pro — Sistema de Aprovação em Tempo Real (Firebase)

Sistema de solicitação e aprovação de acesso, com painel administrativo,
chat de suporte e atualização em tempo real via **Firebase Firestore**.
Visual (tema preto/vermelho, layout, responsividade, cards, tabelas, chat,
cabeçalho, fontes e animações) **permanece idêntico** ao projeto original —
apenas o armazenamento (antes `localStorage`) foi substituído por Firebase
e novas funcionalidades de aprovação/rejeição foram adicionadas.

---

## 1. Arquivos alterados

- `painel.html` — adicionadas colunas **Status** e **Ações** (✅/❌) na tabela
  de solicitações, novas colunas (CPF, Telefone, Ramal), filtro por status,
  renomeação da aba "Registros" para "Solicitações". Scripts inline movidos
  para `js/painel.js` (módulo ES). CSS extraído para `css/styles.css`.
- `usuario.html` — adicionada **tela de aguardo** (⏳ "Solicitação enviada /
  Aguardando aprovação do suporte") e **tela de resultado** (✅ aprovado /
  ❌ rejeitado), exibidas no mesmo `card` do formulário, sem alterar layout.
  Departamento agora é salvo como `departamento` (antes `depto`). Scripts
  movidos para `js/usuario.js`. CSS extraído para `css/styles.css`.
- `admin-login.html` — senha de admin agora validada via Firestore
  (`hdp_config/admin`) em vez de `localStorage`. CSS extraído.
- `index.html` — sem mudanças visuais; apenas referencia o novo
  `css/styles.css` e pequenos ajustes de texto ("Registros" → "solicitações").
- `db.js` — **removido** (substituído por `js/firebase.js`).

## 2. Arquivos novos

```
/css
   styles.css          → todo o CSS original preservado + novos estilos de status/notificação
/js
   firebase.js          → módulo central de integração Firebase (Firestore)
   usuario.js            → lógica do portal do usuário
   painel.js             → lógica do painel administrativo
firebase.json            → configuração do Firebase Hosting
firestore.rules          → regras de segurança do Firestore
firestore.indexes.json    → índices necessários para as queries em tempo real
README.md                 → este arquivo
```

## 3. Estrutura final do projeto

```
/css
   styles.css
/js
   firebase.js
   usuario.js
   painel.js
index.html
usuario.html
admin-login.html
painel.html
firebase.json
firestore.rules
firestore.indexes.json
```

---

## 4. Estrutura de dados no Firestore

### Coleção `hdp_registros` (solicitações)
```json
{
  "login": "",
  "nome": "",
  "cpf": "",
  "departamento": "",
  "cargo": "",
  "telefone": "",
  "ramal": "",
  "nivel": "",
  "status": "aguardando",   // aguardando | aprovado | rejeitado
  "createdAt": "<timestamp>",
  "decidedAt": "<timestamp>" // adicionado ao aprovar/rejeitar
}
```

### Coleção `hdp_colaboradores`
```json
{
  "nome": "", "email": "", "cargo": "", "status": "disponivel|ocupado|ausente",
  "desc": "", "foto": "<base64 ou vazio>", "createdAt": "<timestamp>"
}
```

### Coleção `hdp_chats`
```json
{
  "clienteNome": "",
  "msgs": [ { "de": "usuario|colaborador", "texto": "", "autor": "", "time": "" } ],
  "ts": "", "createdAt": "<timestamp>"
}
```

### Coleção `hdp_config`
```json
{ "admin": { "senha": "admin123" } }
```

---

## 5. Passo a passo para conectar ao Firebase

1. **Criar projeto**: acesse [console.firebase.google.com](https://console.firebase.google.com),
   clique em "Adicionar projeto" e siga o assistente.

2. **Criar app Web**: dentro do projeto, clique no ícone `</>` ("Web"), dê um
   nome ao app e registre. O Firebase mostrará um objeto `firebaseConfig`.

3. **Copiar as credenciais**: abra `js/firebase.js` e substitua o objeto
   `firebaseConfig` pelos valores do seu projeto:

```js
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "seu-projeto.firebaseapp.com",
  projectId: "seu-projeto",
  storageBucket: "seu-projeto.appspot.com",
  messagingSenderId: "1234567890",
  appId: "1:1234567890:web:abcdef123456",
};
```

4. **Ativar o Firestore**: no menu lateral, vá em "Firestore Database" →
   "Criar banco de dados". Escolha o modo (produção ou teste) e a região.

5. **Aplicar as regras de segurança**: use o conteúdo de `firestore.rules`
   na aba "Regras" do Firestore (ou faça deploy via CLI, veja seção 6).
   ⚠️ As regras fornecidas são permissivas (leitura/escrita pública) para
   permitir o fluxo sem autenticação, conforme o sistema atual (login/senha
   apenas no formulário, sem Firebase Auth). Para produção real, recomenda-se
   adicionar Firebase Authentication e restringir `update`/`delete` da
   coleção `hdp_registros` apenas a usuários autenticados como admin.

6. **Criar os índices**: use o conteúdo de `firestore.indexes.json`
   (ou deixe o Firestore criar automaticamente ao clicar no link de erro
   que aparece no console do navegador na primeira execução).

7. **Definir a senha do admin (opcional)**: por padrão, se o documento
   `hdp_config/admin` não existir, o sistema usa `admin123`. Para definir
   outra senha, crie manualmente no console:
   - Coleção: `hdp_config`
   - ID do documento: `admin`
   - Campo: `senha` (string) = `sua_senha_aqui`

---

## 6. Instruções de deploy no Firebase Hosting

1. **Instalar o Firebase CLI** (se ainda não tiver):
```bash
npm install -g firebase-tools
```

2. **Login**:
```bash
firebase login
```

3. **Inicializar o projeto** (na pasta raiz do projeto, onde está o
   `firebase.json`):
```bash
firebase use --add
# selecione o projeto criado no passo 1 da seção anterior
```

4. **Deploy do Hosting + Firestore (regras e índices)**:
```bash
firebase deploy
```

   Ou, separadamente:
```bash
firebase deploy --only hosting
firebase deploy --only firestore:rules,firestore:indexes
```

5. **Acessar o sistema**: ao final do deploy, o terminal mostrará a URL
   pública (ex.: `https://seu-projeto.web.app`). Compartilhe:
   - `https://seu-projeto.web.app/usuario.html` → link para o usuário
   - `https://seu-projeto.web.app/admin-login.html` → acesso do suporte

---

## 7. Fluxo funcional implementado

**Usuário** (celular, computador ou tablet):
1. Faz login (login/senha apenas identificam o usuário, não autenticam no Firebase).
2. Preenche: Nome, CPF, Departamento, Cargo, Telefone, Ramal, Nível de Acesso.
3. Clica em **"Continuar"** na última etapa → grava no Firestore com
   `status: "aguardando"` e exibe a tela **"⏳ Solicitação enviada / Aguardando
   aprovação do suporte"**.
4. Um listener `onSnapshot` no documento da solicitação aguarda mudança de status:
   - `aprovado` → mostra **"✅ Solicitação aprovada"** automaticamente, sem F5.
   - `rejeitado` → mostra **"❌ Solicitação rejeitada"**, com mensagem amigável
     e bloqueio de avanço.
5. O chat lateral continua funcionando normalmente e recebe respostas do
   suporte em tempo real.

**Suporte / Admin** (computador ou celular):
1. Faz login em `admin-login.html`.
2. Aba **"📋 Solicitações"**: lista em tempo real (via `onSnapshot`) todas as
   solicitações com Login, Nome, CPF, Departamento, Cargo, Telefone, Ramal,
   Nível de Acesso, Data/Hora e **Status**.
3. Coluna **Ações**: botões **✅ Aprovar** / **❌ Rejeitar** (visíveis apenas
   para solicitações `aguardando`). A alteração de status é refletida
   instantaneamente para o usuário.
4. Ao chegar uma nova solicitação, aparece uma **notificação flutuante**
   ("🔔 Nova solicitação recebida") no canto da tela e um toast — sem
   bibliotecas externas, apenas CSS/JS nativos.
5. Aba **"📊 Dashboard"** mostra estatísticas (total, aguardando, hoje,
   colaboradores disponíveis) e as últimas solicitações com status.
6. Histórico completo (pendentes, aprovadas, rejeitadas) disponível na aba
   "Solicitações" via filtro de status.

Tudo funciona 100% online, sincronizado entre todos os dispositivos
conectados via `onSnapshot()`.
