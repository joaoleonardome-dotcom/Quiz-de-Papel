# Quiz de Papel — Matemática

App para criar quizzes de matemática (com suporte a LaTeX), projetar em sala,
e gerar folhas de resposta para os alunos preencherem no papel. As questões
ficam salvas no Firebase (Firestore), então sincronizam entre seus
dispositivos, e o app é instalável como PWA no tablet.

## Estrutura do projeto

```
index.html          -> o app
firebase-config.js   -> suas chaves do Firebase (você edita este arquivo)
manifest.json         -> configuração do PWA (ícone, nome, cores)
sw.js                  -> service worker (cache offline)
icons/icon-192.png
icons/icon-512.png
```

## 1. Configurar o Firebase

1. Acesse **console.firebase.google.com** e clique em **Adicionar projeto**.
   Dê um nome (ex: `quiz-matematica`) e conclua a criação.
2. No menu lateral, vá em **Compilação → Firestore Database** e clique em
   **Criar banco de dados**.
   - Escolha a localização mais próxima (ex: `southamerica-east1`).
   - Para começar rápido, selecione **modo de teste** (dá acesso de leitura/
     escrita por 30 dias). Antes de usar em produção, veja a seção
     **Regras de segurança** abaixo.
3. No menu lateral, clique no ícone de engrenagem → **Configurações do
   projeto**. Na aba **Geral**, role até **Seus apps** e clique no ícone
   `</>` (Web) para registrar um app. Não precisa marcar Firebase Hosting.
4. O Firebase vai mostrar um bloco de código com um objeto chamado
   `firebaseConfig`. Copie os valores (`apiKey`, `authDomain`,
   `projectId` etc.) e cole em **`firebase-config.js`**, substituindo os
   valores de exemplo.
5. Salve. Ao abrir `index.html`, o app já deve conectar automaticamente
   (o indicador no canto superior da barra lateral mostra "Sincronizado
   com o Firebase").

### Regras de segurança do Firestore

O modo de teste libera leitura/escrita para qualquer pessoa com o link,
por 30 dias — ok para testar rapidamente, mas depois disso o banco passa
a bloquear tudo. Para uma sala de aula, uma opção simples e razoável é
liberar por tempo indeterminado só a coleção `tests`, em
**Firestore Database → Regras**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /tests/{testId} {
      allow read, write: if true;
    }
  }
}
```

Isso mantém o app simples (sem precisar de login), mas qualquer pessoa
com o link do site poderia editar as questões. Se isso for um problema,
o próximo passo seria ativar **Firebase Authentication** e restringir
`allow write` a usuários autenticados — mas isso já é uma etapa extra,
me avise se quiser que eu implemente.

## 2. Publicar no GitHub Pages

1. Crie um repositório novo no GitHub (ex: `quiz-papel-matematica`).
2. Suba todos os arquivos deste projeto para a raiz do repositório
   (mantendo a pasta `icons/`).
3. No repositório, vá em **Settings → Pages**.
4. Em **Source**, selecione **Deploy from a branch**, escolha a branch
   `main` e a pasta `/ (root)`. Salve.
5. Em alguns minutos o GitHub mostra o link do site, algo como:
   `https://SEU_USUARIO.github.io/quiz-papel-matematica/`
6. Abra esse link — é o endereço que você vai usar em sala e instalar
   no tablet.

O GitHub Pages já serve os arquivos via HTTPS, o que é obrigatório para
o service worker e a instalação como PWA funcionarem.

## 3. Instalar no tablet (PWA)

- **Android (Chrome):** abra o link do site, toque no menu (⋮) e em
  **"Instalar app"** ou **"Adicionar à tela inicial"**.
- **iPad (Safari):** abra o link, toque no ícone de compartilhar
  (quadrado com seta) e em **"Adicionar à Tela de Início"**.

Depois de instalado, o app abre em tela cheia, sem a barra do navegador,
com o ícone que está em `icons/`.

## 4. Atualizando o app depois de publicado

Sempre que enviar mudanças novas para o GitHub, o site atualiza sozinho.
Mas como o `sw.js` guarda uma cópia em cache do app, é bom trocar o
número da versão no topo do arquivo (`CACHE_NAME = 'quiz-papel-v2'`,
`v3`...) a cada atualização, para forçar o dispositivo a baixar a versão
nova em vez de continuar usando a antiga do cache.

## Observações

- As questões de cada teste ficam num único documento do Firestore
  (coleção `tests`), o que é simples e funciona bem para o tamanho normal
  de uma prova. Não é recomendado para milhares de questões no mesmo
  teste.
- Sem internet, o app abre (graças ao cache do service worker) mas não
  sincroniza — é preciso conexão para salvar ou carregar questões do
  Firebase.
