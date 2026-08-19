# Laura Workspace — site + app sincronizado

Este projeto tem três partes, todas já prontas no código. Você só precisa
seguir os passos abaixo **na ordem**.

1. **GitHub + Vercel** → coloca o site no ar com um link fixo (ex:
   `laura-workspace.vercel.app`), e toda vez que você atualizar o código no
   GitHub, o site atualiza sozinho.
2. **Instalar como app** → depois que o site estiver no ar, dá pra "instalar"
   ele no celular e no notebook, com ícone próprio, como um app de verdade.
3. **Supabase (sincronização)** → é o que faz o que você altera em um
   aparelho aparecer no outro. Sem isso, cada aparelho fica com seus
   próprios dados (como já era antes).

---

## Passo 1 — Colocar no GitHub

1. Crie uma conta em https://github.com (se ainda não tiver).
2. Clique em **New repository**. Nome sugerido: `laura-workspace`.
   Pode deixar como **privado** (só você acessa o código-fonte; o site
   publicado é público de qualquer forma, a não ser que você proteja com
   senha dentro do próprio app).
3. Suba os arquivos desta pasta para o repositório. Duas formas:
   - **Pelo navegador:** na página do repositório, clique em "uploading an
     existing file" e arraste todos os arquivos e a pasta `icons/`.
   - **Pelo terminal**, dentro desta pasta:
     ```bash
     git init
     git add .
     git commit -m "Primeira versão"
     git branch -M main
     git remote add origin https://github.com/SEU-USUARIO/laura-workspace.git
     git push -u origin main
     ```

## Passo 2 — Publicar na Vercel

1. Crie uma conta em https://vercel.com (dá pra entrar direto com o GitHub).
2. Clique em **Add New → Project**.
3. Selecione o repositório `laura-workspace` que você acabou de criar.
4. Não precisa mudar nenhuma configuração (é HTML puro, sem build). Clique
   em **Deploy**.
5. Em ~30 segundos você terá uma URL tipo `https://laura-workspace.vercel.app`.

A partir de agora, **qualquer alteração que você enviar para o GitHub**
(`git push`) atualiza o site automaticamente em poucos segundos — isso já é
a "integração" entre código e site.

---

## Passo 3 — Ativar a sincronização entre aparelhos (Firebase)

Sem este passo, o app funciona normalmente, mas cada aparelho guarda seus
próprios dados separadamente (como um app comum). Com ele, o que você
altera no celular aparece no notebook e vice-versa — e quase na hora,
porque o Firestore avisa o app em tempo real quando algo muda.

### 3.1 Criar o projeto

1. Acesse https://console.firebase.google.com e entre com uma conta Google.
2. Clique em **Adicionar projeto**, dê um nome (ex: `laura-workspace`) e
   siga o assistente (pode desativar o Google Analytics, não é necessário).
3. Aguarde a criação (~1 minuto).

### 3.2 Criar o banco de dados (Firestore)

1. No menu lateral do projeto, abra **Firestore Database**.
2. Clique em **Criar banco de dados**.
3. Escolha o modo **Produção** (não "modo de teste") e a localização mais
   próxima de você (ex: `southamerica-east1`, se disponível). Clique em
   **Ativar**.

### 3.3 Definir as regras de acesso

1. Ainda no Firestore, abra a aba **Regras**.
2. Substitua o conteúdo por:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /sync_data/{docId} {
      allow read, write: if true;
    }
  }
}
```

3. Clique em **Publicar**.

> Isso cria uma "gaveta" no banco (coleção `sync_data`) onde os dados de
> cada código de sincronização (PIN) ficam guardados, um documento por
> código. O acesso é liberado publicamente na regra, mas cada aparelho só
> consegue *achar* o documento certo se souber o código secreto (o PIN é
> transformado em um hash e usado como nome do documento) — por isso não é
> necessário criar login/senha de usuário. Assim como no app original, não
> é um cofre de dados sigilosos: é o mesmo nível de proteção de um link
> "não listado".

### 3.4 Registrar um app Web e pegar a configuração

1. No menu lateral, clique na engrenagem ⚙️ → **Configurações do projeto**.
2. Na aba **Geral**, role até "Seus apps" e clique no ícone **</>** (Web).
3. Dê um apelido (ex: `laura-web`) e clique em **Registrar app**. Não
   precisa marcar a opção de Hosting.
4. O Firebase vai mostrar um bloco `firebaseConfig = { ... }` — é esse
   objeto que você vai usar no próximo passo.

### 3.5 Colar no código

Abra o arquivo `index.html`, procure por `CLOUD_SYNC_CONFIG` (bem no início
do `<script>`, logo depois do `<body>`) e substitua:

```js
window.CLOUD_SYNC_CONFIG = {
  FIREBASE_CONFIG: {
    apiKey: 'SUA_API_KEY_AQUI',
    authDomain: 'SEU-PROJETO.firebaseapp.com',
    projectId: 'SEU-PROJETO',
    storageBucket: 'SEU-PROJETO.appspot.com',
    messagingSenderId: 'SEU_SENDER_ID',
    appId: 'SEU_APP_ID',
  },
};
```

pelos valores exatos que o Firebase te mostrou no passo 3.4. Salve, envie
para o GitHub (`git add . && git commit -m "ativa sincronização" && git
push`) — a Vercel republica sozinha.

> Nota: essas chaves do Firebase (`apiKey` etc.) são feitas para ficar no
> código do navegador — não são segredos como uma senha. Quem protege os
> seus dados é a regra do Firestore + o código de sincronização (PIN).

### 3.6 Parear os aparelhos

Na primeira vez que abrir o site (ou app instalado) em cada aparelho, vai
aparecer uma tela pedindo um **código de sincronização** (mínimo 6
caracteres). Use **o mesmo código** em todos os aparelhos que você quer
manter integrados. É esse código — e só ele — que liga um aparelho ao
outro; guarde-o em um lugar seguro (ex: seu gerenciador de senhas).

Para trocar de código depois (ex: parear um aparelho novo em outro espaço
de dados), abra o console do navegador (F12) e rode:
`window.CloudSync.changePin()`.

---

## Passo 4 — Instalar como app (ícone no celular/notebook)

O site já é um **PWA** (Progressive Web App), então dá pra "instalar" sem
precisar de loja de aplicativos.

**Android (Chrome):** abra o site → toque nos 3 pontinhos (⋮) → **Instalar
app** (ou "Adicionar à tela inicial").

**iPhone/iPad (Safari):** abra o site → toque no ícone de compartilhar
(quadrado com seta ↑) → **Adicionar à Tela de Início**.
> No iPhone precisa ser pelo Safari — o Chrome no iOS não tem essa opção.

**Notebook/PC (Chrome ou Edge):** abra o site → clique no ícone de
"instalar" que aparece do lado direito da barra de endereço (ou menu ⋮ →
**Instalar Laura Workspace**).

Depois de instalado, o app abre em janela própria, com ícone na tela
inicial / área de trabalho, igual um aplicativo nativo — e continua
puxando a versão mais nova do site automaticamente.

---

## Como funciona a sincronização, por baixo dos panos

- Cada módulo do app continua salvando os dados no armazenamento local do
  navegador (`localStorage`), exatamente como antes — nada mudou aí.
- Um script (`CloudSync`, dentro do `index.html`) observa essas gravações
  e, ~1 segundo depois de qualquer alteração, envia uma cópia para o
  Firestore (banco de dados do Firebase).
- Enquanto o app está aberto, ele mantém uma conexão em tempo real com o
  Firestore (`onSnapshot`). Assim que outro aparelho salva algo, o aviso
  com o botão **Atualizar** aparece quase na hora — sem precisar ficar
  checando de tempos em tempos.
- Sem internet, o app continua funcionando normalmente com os dados que já
  estavam salvos localmente; a sincronização retoma sozinha quando a
  conexão voltar (o Firestore reconecta automaticamente).

## Backup manual (continua disponível)

O app já tinha um recurso de exportar/importar um arquivo de backup
(`.json`) entre aparelhos — ele continua funcionando normalmente como uma
segunda camada de segurança, independente da sincronização automática.
