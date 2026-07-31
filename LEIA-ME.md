# Pedidos de Marmita — DTCEA-SM
## Guia completo de configuração e uso

---

## Como o app funciona

| Quem | O que faz | Precisa de login? |
|---|---|---|
| Qualquer militar | Entra no link, digita o nome, escolhe o setor e confirma o pedido | Não |
| Motorista | Entra na "Área da equipe", vê o total do dia por refeição e por setor, marca quem recebeu, manda a lista pro rancho pelo WhatsApp | Sim |
| Administrador (Túlio) | Mesma coisa que o motorista + aba de Histórico (relatórios por período, exportação CSV) + aba de Administração (cadastra/edita/remove motoristas) | Sim |

**Prazos automáticos:** o almoço fecha às 09:45 e a janta às 15:30. Depois disso o botão trava sozinho — sem precisar de ninguém para fechar.

**Setores fixos (em ordem alfabética):** APP · EMA · SMSA · SMSO · SMST · TWR/EMS  
Para alterar, edite a linha `setores:` no bloco `CONFIG` dentro do `index.html`.

---

## PARTE 1 — Criar a conta no Firebase (banco de dados gratuito)

> Sem isso o app funciona só no modo de teste (dados ficam só no aparelho). Com o Firebase todo mundo vê e alimenta a mesma lista ao vivo.

### 1.1 Criar o projeto

1. Acesse **[console.firebase.google.com](https://console.firebase.google.com)** com sua conta Google.
2. Clique em **"Adicionar projeto"**.
3. Dê um nome, ex.: `dtcea-sm-marmita`.
4. Desative o Google Analytics (não precisa) → **"Criar projeto"**.
5. Aguarde e clique em **"Continuar"**.

### 1.2 Habilitar autenticação anônima

> O app usa login anônimo pra poder gravar no banco sem expor credenciais.

1. No menu lateral: **"Criação" → "Authentication"**.
2. Clique em **"Primeiros passos"**.
3. Na aba **"Método de login"**, clique em **"Anônimo"**.
4. Ative o botão e salve.

### 1.3 Criar o banco Firestore

1. No menu lateral: **"Criação" → "Firestore Database"**.
2. Clique em **"Criar banco de dados"**.
3. Escolha **"Iniciar no modo de produção"** → **"Próxima"**.
4. Selecione a região **`southamerica-east1` (São Paulo)** → **"Ativar"**.
5. Aguarde o banco ser criado.

### 1.4 Configurar as regras de segurança

1. Ainda no Firestore, clique na aba **"Regras"**.
2. Apague o conteúdo e cole:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

3. Clique em **"Publicar"**.

### 1.5 Pegar as credenciais do app

1. Na engrenagem ⚙️ (canto superior esquerdo) → **"Configurações do projeto"**.
2. Role até **"Seus apps"** → clique no ícone **`</>`** (Web).
3. Dê um apelido, ex.: `marmita-web` → **"Registrar app"**.
4. Você verá um bloco assim:

```javascript
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "dtcea-sm-marmita.firebaseapp.com",
  projectId: "dtcea-sm-marmita",
  storageBucket: "...",
  messagingSenderId: "...",
  appId: "1:...:web:..."
};
```

5. **Copie esse bloco inteiro.** Você vai precisar dele no passo 2.

---

## PARTE 2 — Configurar o arquivo `index.html`

Abra o `index.html` num editor de texto simples (Bloco de Notas, Notepad++, VS Code, qualquer um).

### 2.1 Colar as credenciais do Firebase

Procure por (perto da linha 330):

```javascript
const firebaseConfig = { apiKey: "", authDomain: "", projectId: "", appId: "" };
```

Substitua **essa linha inteira** pelo bloco que você copiou no passo 1.5. Exemplo de como deve ficar:

```javascript
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "dtcea-sm-marmita.firebaseapp.com",
  projectId: "dtcea-sm-marmita",
  appId: "1:...:web:..."
};
```

### 2.2 Configurar o WhatsApp do rancho

Procure por:

```javascript
whatsappRancho: "5551999999999",
```

Troque pelo número real do rancho: `55` + DDD + número, só dígitos.  
Exemplo para (55) 3222-1234 → `"555532221234"`.

### 2.3 Ajustar horários se necessário

```javascript
prazos: { almoco: "09:45", janta: "15:30" },
```

Se os prazos mudarem, edite aqui.

### 2.4 Salvar o arquivo

Salve o `index.html` e pronto para a próxima etapa.

---

## PARTE 3 — Publicar no GitHub Pages (hospedagem gratuita)

### 3.1 Criar uma conta no GitHub

Se ainda não tiver: **[github.com](https://github.com)** → "Sign up" → siga o passo a passo.

### 3.2 Criar o repositório

1. Clique no **+** (canto superior direito) → **"New repository"**.
2. Dê um nome, ex.: `marmita-dtcea`.
3. Deixe como **Public**.
4. **NÃO** marque "Add a README file".
5. Clique em **"Create repository"**.

### 3.3 Subir os arquivos

Na tela do repositório recém-criado, clique em **"uploading an existing file"**.

Arraste ou selecione **todos** os arquivos de uma vez:

- `index.html`
- `manifest.json`
- `sw.js`
- `logo.png`
- `icone-192.png`
- `icone-512.png`
- `icone-180.png`

Clique em **"Commit changes"**.

### 3.4 Ativar o GitHub Pages

1. No repositório, clique em **"Settings"** (aba superior).
2. No menu lateral: **"Pages"**.
3. Em **"Source"**: selecione **"Deploy from a branch"**.
4. Em **"Branch"**: escolha **`main`** e pasta **`/ (root)`**.
5. Clique em **"Save"**.
6. Aguarde 1–2 minutos.

O link do app vai aparecer assim:
```
https://SEU-USUARIO.github.io/marmita-dtcea/
```

### 3.5 Autorizar o domínio no Firebase

1. Volte ao **Firebase Console** → **"Authentication"** → **"Settings"** → **"Domínios autorizados"**.
2. Clique em **"Adicionar domínio"**.
3. Digite: `SEU-USUARIO.github.io`
4. Salve.

---

## PARTE 4 — Primeiro acesso e cadastro da equipe

### 4.1 Abrir o app pela primeira vez

Acesse o link do GitHub Pages. O Firebase vai criar automaticamente a conta inicial do administrador:

- **Usuário:** `admin`
- **Senha:** `Tuleco4.0`

### 4.2 Entrar como administrador

1. Na tela de pedido, role até o rodapé → clique em **"Sou motorista ou administrador"**.
2. Digite `admin` e `Tuleco4.0` → **"Entrar"**.

### 4.3 Criar sua conta pessoal

1. Vá na aba **"Administração"**.
2. No formulário "Cadastrar motorista ou administrador":
   - **Usuário:** `tulio` (ou o que preferir, sem espaços)
   - **Nome:** Túlio
   - **Senha:** a senha que quiser (mínimo 6 caracteres)
   - Marque **"Também é administrador"**
3. Clique em **"Cadastrar"**.
4. Saia (`Sair`) e entre com a nova conta.

### 4.4 Cadastrar os motoristas

Repita o processo de cadastro para cada motorista, sem marcar "Também é administrador".

### 4.5 Remover ou trocar a senha do admin genérico

Depois de criar sua conta pessoal, edite o usuário `admin` e troque a senha, ou remova-o — assim só você e os motoristas cadastrados têm acesso.

---

## PARTE 5 — Distribuir o link para os militares

Envie o link pelo grupo do WhatsApp ou Signal:

```
https://SEU-USUARIO.github.io/marmita-dtcea/
```

Sugira que instalem como atalho no celular:

- **Android (Chrome):** menu ⋮ → "Instalar aplicativo" / "Adicionar à tela inicial"
- **iPhone (Safari):** botão compartilhar □↑ → "Adicionar à Tela de Início"

Abre em tela cheia, sem barra de navegador, como um app normal.

---

## Uso diário — motorista

1. Entrar na Área da equipe antes do prazo de cada refeição.
2. Na aba **"Painel"**, verificar o total e a lista por setor.
3. Tocar em **"Mandar o pedido pelo WhatsApp"** → mensagem já formatada vai para o rancho.
4. Na entrega, marcar cada pessoa com ✓ à medida que recebe.

## Uso diário — administrador

Tudo que o motorista faz, mais:

- **Histórico:** aba "Histórico" → escolhe o período → vê total por refeição, por setor e por dia → exporta em CSV para planilha.

---

## Manutenção

### Atualizar o app

1. Edite o `index.html`.
2. No `sw.js`, mude `quentinhas-v4` para `quentinhas-v5` (ou o número seguinte). **Isso é obrigatório** — sem essa mudança os celulares que já instalaram continuam abrindo a versão antiga.
3. Suba os arquivos alterados no GitHub (arrastar para o repositório e "Commit changes").
4. Aguarde 1–2 minutos.

### Alterar os setores

Edite a linha no `index.html`:
```javascript
setores: ["APP", "EMA", "SMSA", "SMSO", "SMST", "TWR/EMS"],
```
Adicione, remova ou renomeie os itens, mantendo a lista entre colchetes e cada nome entre aspas, separados por vírgula. Salve, suba, atualize o `sw.js`.

### Alterar os horários de corte

```javascript
prazos: { almoco: "09:45", janta: "15:30" },
```

---

## Perguntas frequentes

**O pedido some depois da meia-noite?**  
Sim. O app mostra só o pedido do dia atual. Os dados históricos ficam no Firestore e são consultáveis pela aba Histórico.

**Alguém pediu no aparelho errado / esqueceu de pedir?**  
O motorista pode ligar normalmente e contar no total na hora de mandar pro rancho. O app é uma ferramenta de apoio, não substitui o bom senso.

**O app funciona sem internet?**  
A interface abre (service worker em cache), mas os pedidos precisam de conexão para serem gravados e para o motorista ver a lista ao vivo.

**Como exporto todos os pedidos do mês?**  
Aba Histórico → selecione o primeiro e o último dia do mês → Buscar → Exportar CSV. O arquivo abre em qualquer planilha (Excel, LibreOffice, Google Sheets).
