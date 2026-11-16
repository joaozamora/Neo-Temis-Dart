# NeoTêmis
<img width="500" height="400" alt="Logo" src="docs/prints/NeoTemis OFICIAL png 1.png" />

Aplicativo de gestão de processos jurídicos para advogados. O advogado entra com o registro da OAB, cadastra seus processos (com status, prazo de vencimento e imagem) e pode pedir ajuda à **Têmis**, uma assistente com IA que filtra os processos na tela e avisa quais prazos estão mais próximos.

## Estrutura do projeto

| Pasta | O que é | Tecnologias |
|---|---|---|
| [`neotemis/`](neotemis/) | App (interface) + assistente Têmis | Flutter / Dart, Google Gemini |
| [`neotemis_api/`](neotemis_api/) | API (servidor) | Node.js, Express, MongoDB, JWT |

Como as partes se conectam:

- **Login e processos:** o app não acessa o banco diretamente, tudo passa pela API, que precisa estar rodando para o app funcionar
- **Assistente Têmis:** é feita em Dart, dentro do app, o app busca os processos na API e conversa direto com o Google Gemini (Diferencial do Projeto)

## Prints

| Login | Cadastro | Início |
|---|---|---|
| ![Tela de login](docs/prints/login.png) | ![Tela de cadastro](docs/prints/cadastro.png) | ![Tela inicial](docs/prints/home.png) |

| Processos | Detalhes do processo | Assistente Têmis |
|---|---|---|
| ![Lista de processos](docs/prints/processos.png) | ![Diálogo do processo](docs/prints/processo.png) | ![Chat com a Têmis](docs/prints/assistente.png) |

---

## Requisitos

| Programa | Versão | Para quê |
|---|---|---|
| **Git** | qualquer recente | baixar o projeto |
| **Node.js** | 18 ou mais nova (testado na 26) | rodar a API |
| **MongoDB Community Server** | 6 ou mais nova (testado na 8.3) | banco de dados (ou use o MongoDB Atlas) |
| **Flutter SDK** | com Dart 3.13 ou mais novo (testado no Flutter 3.47) | rodar o app |
| **Chave do Google Gemini** | gratuita, sem cartão | assistente Têmis (opcional) |

### Para rodar o app

O app está configurado para **Linux** e **Windows**.

- **Windows:** Necessario Visual Studio 2022
- **Linux:** compilador e bibliotecas do GTK:

  ```bash
  # Ubuntu / Debian / Mint
  sudo apt install clang cmake ninja-build pkg-config libgtk-3-dev

  # Arch / CachyOS / Manjaro
  sudo pacman -S clang cmake ninja pkgconf gtk3
  ```

Depois rode `flutter doctor`: os itens **Flutter** e do desktop (Linux toolchain ou Visual Studio)

### Bibliotecas (instaladas automaticamente)

Não é preciso instalar à mão: `npm install` e `flutter pub get` baixam tudo

- **API** ([`package.json`](neotemis_api/package.json)): `express`, `mongoose`, `jsonwebtoken`, `bcryptjs`, `cors`, `dotenv`.
- **App** ([`pubspec.yaml`](neotemis/pubspec.yaml)): `http`, `shared_preferences`, `image_picker`, `google_fonts`, `flutter_localizations`.

---

## Instalação

### 1. Baixar o projeto

```bash
git clone https://github.com/RenanAndreata141/NeoTemis.git
cd NeoTemis
```

### 2. Ligar o MongoDB

- **Windows:** costuma iniciar sozinho como serviço. Se não, abra "Serviços" e inicie o **MongoDB Server**.
- **Linux:** `sudo systemctl start mongodb` (em algumas distros o serviço se chama `mongod`)

Se preferir o **MongoDB Atlas** (na nuvem), pule este passo e use a URL de conexão dele no `.env`

### 3. Configurar e rodar a API

```bash
cd neotemis_api
npm install
```

Crie o `.env` a partir do modelo:

```bash
# Linux / macOS
cp .env.example .env

# Windows (PowerShell)
copy .env.example .env
```

Preencha o `.env`:

```env
# Endereço do banco. Se usar o Atlas, cole a URL de conexão dele aqui.
MONGODB_URI=mongodb://127.0.0.1:27017/neotemis

# Qualquer texto longo e aleatório. Serve para assinar os logins.
JWT_SECRET=troque-por-um-texto-longo-e-aleatorio

PORT=3000
```

Inicie a API:

```bash
npm start        # ou: npm run dev  (reinicia sozinha ao salvar arquivos) E VALE CONSTAR QUE TEM QUE ESTAR DENTRO DA PASTA neotemis_api PRO COMANDO FUNCIONAR!
```

**Deixe esse terminal aberto** enquanto usa o app

> Rode `npm start` 

### 4. Rodar o app

Em **outro terminal**, a partir da pasta `NeoTemis`:

```bash
cd neotemis
flutter pub get
flutter run -d linux      # no Windows: flutter run -d windows
```

Para usar a assistente Têmis, passe também a chave do Gemini:

```bash
flutter run -d linux --dart-define=GEMINI_API_KEY=sua_chave
```

Na primeira tela, clique em **Criar conta**, cadastre-se com um registro da OAB e uma senha, e pronto

---

## Assistente Têmis (IA)

O botão redondo com a logo, no canto inferior direito, abre o chat. Exemplos de pedidos:

- "Quais processos vencem primeiro?"
- "Mostre os processos vencidos"
- "Filtre os processos aguardando do cliente João"

Quando a assistente filtra, a tela de Processos mostra só os processos escolhidos, com uma faixa **"Assistente: …"** e o botão **LIMPAR**

### Configuração

A chave do Gemini é gratuita em https://aistudio.google.com/apikey e é passada ao rodar o app:

```bash
flutter run -d linux --dart-define=GEMINI_API_KEY=sua_chave

# Opcional: escolher os modelos (separados por vírgula)
flutter run -d linux --dart-define=GEMINI_API_KEY=sua_chave --dart-define=GEMINI_MODEL=gemini-3.5-flash,gemini-3.1-flash-lite
```

- Sem a chave, o resto do app funciona normalmente e o chat avisa que a assistente não está configurada
- Para a assistente responder sobre vencimentos, preencha o campo **Prazo** dos processos
- O código fica em [`neotemis/lib/data/repositories/assistente_repository.dart`](neotemis/lib/data/repositories/assistente_repository.dart) (prompt) e [`neotemis/lib/data/services/gemini_client.dart`](neotemis/lib/data/services/gemini_client.dart) (chamada à API do Gemini). A arquitetura do app está descrita em [`neotemis/README.md`](neotemis/README.md)

---

## Testes e análise

```bash
cd neotemis
flutter analyze   # análise estática (lints do flutter_lints)
flutter test      # testes de unidade da lógica (pasta test/)
```

---

## Documentação da API

As rotas estão em [`neotemis_api/README.md`](neotemis_api/README.md)

---

## Participantes do projeto
- Renan Pereira Andreata RGM: 11251100818
- Igor Dias Viana RGM: 11251100289
- Guilherme Borges RGM: 11251100564
- Glaucio dos Santos Abrantes Soares RGM: 11251100887
- João Vitor Albuquerque Zamora RGM: 11251100929
