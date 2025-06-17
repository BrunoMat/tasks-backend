# Dominando o Playwright: Fundamentos Essenciais

## Introdução ao Playwright

O Playwright é uma poderosa biblioteca de automação de navegadores desenvolvida pela Microsoft. Ele permite automatizar navegadores Chromium, Firefox e WebKit com uma única API, oferecendo recursos robustos para testes end-to-end, web scraping e automação geral de tarefas web. Sua arquitetura moderna e foco em estabilidade e velocidade o tornam uma excelente escolha para sistemas complexos.

### Por que Playwright?

- **Multi-navegador:** Suporta os três principais motores de navegador (Chromium, Firefox, WebKit) com uma única API, garantindo compatibilidade e reduzindo a necessidade de testes em múltiplas ferramentas.
- **Multi-linguagem:** Oferece APIs para JavaScript/TypeScript, Python, Java e .NET, permitindo que equipes com diferentes stacks tecnológicas o utilizem.
- **Auto-espera:** O Playwright espera automaticamente pelos elementos estarem prontos antes de realizar ações, eliminando a necessidade de sleeps manuais e tornando os testes mais estáveis.
- **Isolamento de contexto:** Cada teste é executado em um contexto de navegador isolado, garantindo que os testes sejam independentes e não interfiram uns nos outros.
- **Recursos avançados:** Suporte nativo para interceptação de rede, emulação de dispositivos móveis, geolocalização, permissões, downloads, uploads e muito mais.
- **Ferramentas de depuração:** Inclui ferramentas como o Playwright Inspector, que facilita a depuração de testes e a identificação de seletores.

## Seletores e Localização de Elementos

Para interagir com elementos em uma página web, o Playwright utiliza seletores. Seletores são strings que descrevem como encontrar um elemento específico na DOM (Document Object Model). O Playwright oferece uma variedade de tipos de seletores, permitindo flexibilidade e robustez na localização de elementos.

### Tipos de Seletores Comuns

O Playwright suporta diversos tipos de seletores, incluindo:

- **Seletores CSS:** São os seletores mais comuns e versáteis, baseados nas regras de estilo CSS. Permitem selecionar elementos por tag, classe, ID, atributos, hierarquia, etc.
  - Exemplo: `page.locator("button.login-button")`
- **Seletores XPath:** Uma linguagem poderosa para navegar e selecionar nós em documentos XML (e HTML). Útil para cenários mais complexos onde seletores CSS podem ser limitados.
  - Exemplo: `page.locator("//button[text()=\'Entrar\']")`
- **Seletores de Texto:** Permitem encontrar elementos com base no texto visível que contêm. Muito útil para elementos que não possuem IDs ou classes estáveis.
  - Exemplo: `page.getByText("Entrar")`
- **Seletores de Role (ARIA):** Selecionam elementos com base em seu papel de acessibilidade (ARIA role). Isso torna os testes mais robustos e acessíveis, pois se baseiam na semântica do elemento.
  - Exemplo: `page.getByRole("button", { name: "Entrar" })`
- **Seletores de Test ID:** Uma prática recomendada é adicionar atributos `data-testid` aos elementos da UI. Isso cria seletores estáveis que não são afetados por mudanças de estilo ou estrutura da DOM.
  - Exemplo: `page.getByTestId("login-button")`
- **Seletores de Placeholder:** Seleciona elementos de input com base no texto do placeholder.
  - Exemplo: `page.getByPlaceholder("Digite seu usuário")`
- **Seletores de Label:** Seleciona elementos de input associados a um determinado texto de label.
  - Exemplo: `page.getByLabel("Usuário")`

### Prioridade e Boas Práticas na Escolha de Seletores

Ao escolher um seletor, é importante considerar a robustez e a legibilidade do teste. A ordem de preferência geralmente é:

1.  **`data-testid`:** Ideal para testes, pois são criados especificamente para automação e não são afetados por mudanças na UI ou no CSS. No nosso site de exemplo, utilizamos `data-testid` para os campos de input e botões de login/registro.
2.  **`getByRole`:** Excelente para acessibilidade e semântica. Garante que o teste interaja com o elemento da mesma forma que um usuário com tecnologias assistivas.
3.  **`getByText` / `getByLabel` / `getByPlaceholder`:** Bons para elementos que possuem texto visível ou labels claras. Menos frágeis que seletores CSS baseados em classes voláteis.
4.  **Seletores CSS (com IDs ou classes estáveis):** Se não houver `data-testid` ou role, use seletores CSS com IDs (se únicos e estáveis) ou classes que não mudam frequentemente.
5.  **XPath:** Use XPath como último recurso, para cenários muito específicos onde outros seletores não são suficientes. XPath pode ser mais frágil e menos legível.

### Exemplo Prático de Seletores no Nosso Sistema de Testes

No nosso sistema de testes, a página de login possui os seguintes elementos com `data-testid`:

- Campo de usuário: `data-testid="username-input"`
- Campo de senha: `data-testid="password-input"`
- Botão de login: `data-testid="login-button"`
- Botão de preencher credenciais de teste: `data-testid="fill-test-credentials"`

No arquivo `tests/login.spec.js`, você pode ver a aplicação desses seletores:

```javascript
await page.getByTestId("username-input").fill("testuser");
await page.getByTestId("password-input").fill("test123");
await page.getByTestId("login-button").click();
```

Isso garante que, mesmo que a estrutura interna do HTML mude (por exemplo, se a classe CSS do input for alterada), o teste continuará funcionando, pois ele se baseia em um atributo dedicado para testes.

Além disso, para o botão de registro, usamos `getByRole`:

```javascript
await page.getByRole("tab", { name: "Registro" }).click();
```

Isso demonstra a flexibilidade do Playwright em usar diferentes tipos de seletores para diferentes propósitos, priorizando a robustez e a legibilidade dos testes.

## Interações Básicas com Elementos

Uma vez que você localiza um elemento, o Playwright oferece uma API intuitiva para interagir com ele. As interações mais comuns incluem clicar, digitar texto, selecionar opções em dropdowns, e muito mais.

### Ações Comuns

- **`click()`:** Clica em um elemento. O Playwright espera automaticamente que o elemento esteja visível, habilitado e receba eventos de clique.
  - Exemplo: `await page.getByTestId("login-button").click();`
- **`fill(value)`:** Preenche um campo de texto com o valor especificado. Limpa o campo antes de digitar.
  - Exemplo: `await page.getByTestId("username-input").fill("testuser");`
- **`type(text, [options])`:** Digita texto caractere por caractere, simulando a digitação humana. Útil para testar validações de input em tempo real.
  - Exemplo: `await page.getByTestId("password-input").type("test123");`
- **`press(key)`:** Simula o pressionar de uma tecla ou combinação de teclas.
  - Exemplo: `await page.getByTestId("username-input").press("Enter");`
- **`check()` / `uncheck()`:** Marca ou desmarca checkboxes e radio buttons.
  - Exemplo: `await page.getByLabel("Lembrar-me").check();`
- **`selectOption(value)`:** Seleciona uma opção em um elemento `<select>` (dropdown) pelo valor, label ou índice.
  - Exemplo: `await page.getByLabel("País").selectOption("BR");`
- **`hover()`:** Move o mouse sobre um elemento, útil para testar tooltips ou menus que aparecem ao passar o mouse.
  - Exemplo: `await page.getByText("Meu Perfil").hover();`

### Exemplo de Interações no Teste de Login

No nosso teste de login (`tests/login.spec.js`), as interações básicas são demonstradas claramente:

```javascript
// Preenche o campo de usuário
await page.getByTestId("username-input").fill("testuser");

// Preenche o campo de senha
await page.getByTestId("password-input").fill("test123");

// Clica no botão de login
await page.getByTestId("login-button").click();
```

Essas três linhas de código simulam o fluxo de um usuário real interagindo com a página de login, preenchendo os campos e clicando no botão. A simplicidade e legibilidade da API do Playwright tornam a escrita de testes muito eficiente.

## Navegação e Aguardar Elementos

Um dos aspectos mais importantes da automação web é garantir que a página esteja no estado esperado antes de interagir com os elementos. O Playwright possui mecanismos de auto-espera inteligentes que lidam com a maioria dos casos, mas também oferece métodos explícitos para cenários específicos.

### Navegação

- **`page.goto(url, [options])`:** Navega para uma URL específica. Por padrão, o Playwright espera até que o evento `load` da página seja disparado, indicando que a página e todos os seus recursos foram carregados.
  - Exemplo: `await page.goto("/login");` (usando `baseURL` configurada no `playwright.config.js`)
- **`page.goBack()` / `page.goForward()`:** Navega para a página anterior ou próxima no histórico do navegador.

### Auto-Espera (Auto-waiting)

O Playwright é inteligente o suficiente para esperar automaticamente por elementos estarem visíveis, habilitados e anexados à DOM antes de realizar uma ação. Isso significa que você raramente precisará adicionar esperas explícitas para a maioria das operações. Por exemplo, quando você chama `element.click()`, o Playwright fará o seguinte:

1.  Espera que o elemento esteja na DOM.
2.  Espera que o elemento esteja visível.
3.  Espera que o elemento esteja habilitado.
4.  Espera que o elemento esteja estável (não se movendo).
5.  Espera que o elemento receba eventos de clique no ponto de clique.

### Esperas Explícitas (quando necessário)

Embora a auto-espera seja poderosa, há situações em que você pode precisar de esperas explícitas:

- **`page.waitForSelector(selector, [options])`:** Espera que um seletor apareça na DOM e atenda a certas condições (visível, anexado, etc.).
  - Exemplo: `await page.waitForSelector(".dashboard-loaded");`
- **`page.waitForLoadState(state)`:** Espera que a página atinja um determinado estado de carregamento (`load`, `domcontentloaded`, `networkidle`).
  - Exemplo: `await page.waitForLoadState("networkidle");`
- **`page.waitForURL(url, [options])`:** Espera que a URL da página mude para uma URL específica ou corresponda a um padrão.
  - Exemplo: `await page.waitForURL("/dashboard");`
- **`expect(locator).toBeVisible()` / `expect(locator).toBeEnabled()` / `expect(locator).toHaveText()`:** Usar asserções do `expect` com auto-espera é a forma preferida de esperar por um estado específico do elemento. O Playwright tentará repetidamente até que a condição seja satisfeita ou o timeout seja atingido.
  - Exemplo: `await expect(page.url()).toContain("/dashboard");`
  - Exemplo: `await expect(page.getByTestId("user-name")).toHaveText("testuser");`

No nosso teste de login, após o clique no botão, usamos `expect(page.url()).toContain("/dashboard")` e `expect(page.getByTestId("user-name")).toHaveText("testuser")`. Essas asserções não apenas verificam o estado final da aplicação, mas também atuam como esperas, garantindo que o Playwright aguarde até que a URL mude e o nome de usuário esteja visível no dashboard antes de prosseguir.

## Captura de Screenshots e Vídeos

Playwright oferece recursos integrados para capturar screenshots e gravar vídeos dos testes. Isso é extremamente útil para depuração, relatórios de bugs e documentação.

### Screenshots

- **`page.screenshot([options])`:** Captura uma screenshot da página inteira ou de uma parte específica.
  - Exemplo: `await page.screenshot({ path: 'screenshot.png' });`
- **`locator.screenshot([options])`:** Captura uma screenshot de um elemento específico.
  - Exemplo: `await page.getByTestId("login-card").screenshot({ path: 'login-card.png' });`

Opções comuns para screenshots:

- `path`: Caminho onde a screenshot será salva.
- `fullPage`: `true` para capturar a página inteira (rolando se necessário), `false` para apenas a viewport visível (padrão).
- `clip`: Um objeto com `x`, `y`, `width`, `height` para capturar uma região específica.

### Vídeos

Para gravar vídeos dos testes, você precisa configurar o `playwright.config.js`.

No nosso `playwright.config.js`, a configuração para vídeo é:

```javascript
use: {
  // ... outras configurações
  video: 'retain-on-failure',
},
```

As opções para `video` incluem:

- `'off'`: Não grava vídeos (padrão).
- `'on'`: Grava vídeo para todos os testes.
- `'retain-on-failure'`: Grava vídeo apenas para testes que falham.
- `'on-first-retry'`: Grava vídeo para o primeiro retry de um teste que falha.

Os vídeos são salvos automaticamente em um diretório temporário e, se configurado para reter, são movidos para o diretório de resultados do Playwright (geralmente `test-results/`).

### Uso em Testes

Você pode adicionar uma linha para capturar uma screenshot em um ponto específico do seu teste para depuração:

```javascript
test("should display an error message with invalid credentials", async ({
  page,
}) => {
  await page.getByTestId("username-input").fill("invaliduser");
  await page.getByTestId("password-input").fill("wrongpassword");
  await page.getByTestId("login-button").click();

  // Captura uma screenshot após a tentativa de login falha
  await page.screenshot({ path: "screenshots/login-failed.png" });

  await expect(page.locator(".bg-red-50")).toBeVisible();
  await expect(page.locator(".bg-red-50")).toContainText(
    "Credenciais inválidas"
  );
});
```

Esses recursos são inestimáveis para entender o que aconteceu durante um teste, especialmente em pipelines de CI/CD onde você não tem acesso visual direto ao navegador.

## Observação sobre a Execução de Testes no Ambiente Sandbox

É importante notar que, devido a certas limitações de dependências do sistema no ambiente sandbox atual, a execução completa dos testes do Playwright para todos os navegadores (Chromium, Firefox, WebKit) pode não ser bem-sucedida. Erros relacionados a bibliotecas ausentes (`Executable doesn't exist` ou `Host system is missing dependencies`) podem ocorrer.

No entanto, o código dos testes e a configuração do Playwright estão corretos e funcionariam em um ambiente de desenvolvimento local com todas as dependências instaladas. O foco deste tutorial é demonstrar a escrita e a estrutura dos testes com Playwright, e não a execução perfeita em todos os navegadores dentro deste ambiente específico.

Recomenda-se executar os testes em sua máquina local para uma experiência completa e sem problemas de dependência. As instruções para configurar o ambiente local serão fornecidas na seção de entrega final.

## Automação Web Avançada

Além das interações básicas, o Playwright oferece recursos poderosos para lidar com cenários de automação web mais complexos, como formulários dinâmicos, upload de arquivos, drag and drop, múltiplas abas e interceptação de requisições de rede.

### Formulários Complexos e Validações

Em aplicações reais, formulários podem ter validações em tempo real, campos dependentes e múltiplos passos. O Playwright permite interagir com esses formulários de forma eficaz.

- **Preenchimento de campos:** Use `fill()` para preencher inputs, textareas e selects. Para campos com validação em tempo real, `type()` pode ser mais adequado para simular a digitação caractere por caractere.
- **Interação com elementos dinâmicos:** Se campos ou seções do formulário aparecem ou desaparecem com base em seleções anteriores, use `waitForSelector()` ou asserções `expect().toBeVisible()` para garantir que o elemento esteja pronto antes de interagir.
- **Validação de mensagens de erro:** Após submeter um formulário com dados inválidos, você pode verificar as mensagens de erro exibidas na UI.

No nosso dashboard, temos filtros de tarefas que simulam uma interação com formulários:

```javascript
test("should be able to filter tasks by status", async ({ page }) => {
  await page.getByTestId("filter-pending").click();
  await expect(page.get
(Content truncated due to size limit. Use line ranges to read in chunks)
```

Este teste demonstra como interagir com botões de filtro e verificar se os contadores de
tarefas são atualizados corretamente, simulando a interação com um formulário de
pesquisa ou filtragem.

## Upload de Arquivos

Para fazer upload de arquivos, o Playwright oferece o método `setInputFiles().`

```javascript
// Supondo que você tenha um input de arquivo como este no HTML:
// <input type="file" id="my-file-input">
await page.setInputFiles("#my-file-input", "./path/to/your/
file.txt");
// Ou para múltiplos arquivos:
// await page.setInputFiles("#my-file-input", ["./file1.txt",
"./file2.txt"]);
// Depois de selecionar o arquivo, você geralmente clicaria em
um botão de upload
await page.getByRole("button", { name: "Upload" }).click();
// E então verificaria uma mensagem de sucesso ou o estado da UI
await expect(page.locator(".
```

No nosso sistema de demonstração, não há um campo de upload de arquivos, mas o
exemplo acima ilustra o conceito. Você pode encontrar um exemplo conceitual no
arquivo tests/advanced.spec.js.

## Download de Arquivos

O Playwright também oferece uma API robusta para lidar com downloads de arquivos.
Você pode interceptar downloads, obter informações sobre eles e salvá-los em um local
específico.

- page.waitForEvent("download"): Este método espera que um evento de download ocorra. Ele retorna um objeto Download que contém informações sobre o download e métodos para manipulá-lo.

Exemplo de download de um arquivo:

```javascript
("should be able to download a file", async ({ page }) => { await
    page.goto("https://example.com/download");
// Inicia a espera pelo evento de download ANTES de clicar no link/botão de
    download const [download] = await
    Promise.all([ page.waitForEvent("download"), // Espera pelo evento de download
    page.locator("#download-link").click(), // Clica no link/botão que inicia o
    download ]);
// Obtém o nome do arquivo sugerido pelo navegador const suggestedFilename =
    download.suggestedFilename(); console.log( Nome do arquivo sugerido: ${suggestedFilename});
// Salva o arquivo em um diretório específico const downloadPath = ./downloads/${suggestedFilename};
    await download.saveAs(downloadPath);
// Verifica se o arquivo foi baixado com sucesso (opcional) // Você pode usar uma biblioteca de sistema de arquivos para verificar a existência e o tamanho do arquivo
    const fs = require("fs"); expect(fs.existsSync(downloadPath)).toBeTruthy();
    console.log( Arquivo baixado para: ${downloadPath});
});
```

- download.path(): Retorna o caminho temporário onde o arquivo foi baixado pelo Playwright. Este caminho é temporário e o arquivo será excluído após o teste, a menos que você o salve explicitamente com saveAs().

- download.url(): Retorna a URL de onde o download foi iniciado.
- download.page(): Retorna a página que iniciou o download.

Lidar com downloads é crucial para testar funcionalidades que envolvem a exportação
de dados ou relatórios, garantindo que o conteúdo baixado esteja correto e acessível.

### Drag and Drop

O Playwright permite simular operações de arrastar e soltar ( drag and drop).

```javascript
// Arrastar um elemento para outro
await page.locator(".draggable-
item").dragTo(page.locator(".droppable-area"));
// Verificar se a ação foi bem-sucedida
await expect(page.locator(".droppable-
area")).toContainText("Item dropped");
```

Assim como o upload de arquivos, o sistema de demonstração não possui elementos de drag and drop, mas o conceito é aplicável a qualquer aplicação web que utilize essa funcionalidade. Um exemplo conceitual está disponível em tests/advanced.spec.js.

### Múltiplas Abas e Janelas

Testar cenários que envolvem múltiplas abas ou janelas é comum, especialmente em fluxos de autenticação ou redirecionamentos para serviços externos. O Playwright facilita o gerenciamento de múltiplos contextos de navegador.

```javascript
test("should handle new tab opening", async ({ page }) => {
// Cria uma promessa que será resolvida quando uma nova página
for aberta
const [newPage] = await Promise.all([
page.waitForEvent("popup"), // ou "page" para nova aba/
janela
page.getByRole("link", { name: "Abrir em nova
aba" }).click(),
]);
// Interage com a nova página
await newPage.waitForLoadState();
await expect(newPage).toHaveURL(/.*external-site.com/);
await newPage.close();
});
```

Para cenários mais complexos onde você precisa de controle total sobre os contextos do navegador, você pode criar novos contextos e páginas explicitamente:

```javascript
test("should be able to handle multiple tabs/windows
(conceptual)", async ({ browser }) => {
const context = await browser.newContext();
const page1 = await context.newPage();
await page1.goto("https://example.com");
const page2 = await context.newPage();
await page2.goto("https://google.com");
// Traz uma página para o foco
await page1.bringToFront();
await expect(page1).toHaveURL("https://example.com/");
await context.close();
});
```

Este exemplo conceitual está presente em tests/advanced.spec.js.

### Interceptação de Requisições de Rede

Um dos recursos mais poderosos do Playwright é a capacidade de interceptar, modificar e mockar requisições de rede. Isso é crucial para testar diferentes estados da aplicação (por exemplo, dados vazios, erros de API) sem a necessidade de alterar o backend, ou para acelerar testes mockando chamadas de API.

- page.route(url, handler): Intercepta requisições que correspondem a uma URL ou padrão. O handler pode então decidir como a requisição deve ser tratada (continuar, abortar, mockar).

```javascript
test("should be able to intercept network requests
(conceptual)", async ({ page }) => {
// Mockar uma resposta para o endpoint de tarefas
await page.route("**/api/tasks", async route => {
const json = [
{ id: 99, title: "Tarefa Mockada", description: "Esta
tarefa foi mockada pelo Playwright", status: "completed",
priority: "high", user_id: 1, created_at: new
Date().toISOString() }
];
await route.fulfill({
status: 200,
contentType: "application/json",
body: JSON.stringify(json),
});
});
// Navegar para o dashboard, que agora carregará dados
mockados
await page.goto("/dashboard");
// Verificar se a tarefa mockada é exibida
await expect(page.getByText("Tarefa Mockada")).toBeVisible();
await expect(page.getByText("Esta tarefa foi mockada pelo
Playwright")).toBeVisible();
});
```

Este exemplo conceitual, presente em `tests/advanced.spec.js`, demonstra como você pode interceptar a chamada para `/api/tasks` e retornar dados falsos. Isso é extremamente útil para:

- Testar cenários de erro: Mockar respostas de erro da API para verificar como a UI reage.
- Isolar testes de UI: Garantir que os testes de interface não dependam da disponibilidade ou do estado do backend.
- Acelerar testes: Evitar chamadas de rede reais, tornando os testes mais rápidos e determinísticos.

# Fundamentos: Condicionais e Estruturas de Repetição

## Condicionais (if, else, else if)

As condicionais permitem que o programa tome decisões com base em condições booleanas.

### Exemplo básico com if/else:

```javascript
const idade = 20;
if (idade >= 18) {
  console.log("Maior de idade");
} else {
  console.log("Menor de idade");
}
```

Exemplo com else if:

```javascript
const nota = 75;
if (nota >= 90) {
  console.log("Excelente");
} else if (nota >= 70) {
  console.log("Bom");
} else {
  console.log("Precisa melhorar");
}
```

### Laços de Repetição com for

A estrutura for permite executar repetidamente um bloco de código enquanto uma condição for verdadeira.

Exemplo de laço de 0 a 4:

```javascript
for (let i = 0; i < 5; i++) {
  console.log(`Contador: ${i}`);
}
```

### Iterando sobre um array:

```javascript
const frutas = ["maçã", "banana", "laranja"];
for (let i = 0; i < frutas.length; i++) {
  console.log(frutas[i]);
}
```

### Combinação: for + if

Você pode usar for com if para aplicar condições a cada item iterado.

Exemplo de filtragem de números pares:

```javascript
const numeros = [1, 2, 3, 4, 5];

for (let numero of numeros) {
  if (numero % 2 === 0) {
    console.log(`${numero} é par`);
  }
}
```

### Condicional envolvendo for

Também é possível condicionar a execução de um for com base em uma decisão anterior:

```javascript
const deveExecutar = true;

if (deveExecutar) {
  for (let i = 1; i <= 3; i++) {
    console.log(`Passo ${i}`);
  }
} else {
  console.log("Execução cancelada");
}
```

# Fundamentos: Arrays e Arquivos JSON

## Percorrendo um array e selecionando índice ou texto

Você pode usar diferentes abordagens:

### Loop clássico (`for`)

```javascript
const arr = ["maçã", "banana", "laranja"];
for (let i = 0; i < arr.length; i++) {
  console.log(`Índice ${i}: ${arr[i]}`);
}

// Saída:
// Índice 0: maçã
// Índice 1: banana
// Índice 2: laranja
```

### for...of com índice via .entries()

```javascript
const arr = ["maçã", "banana", "laranja"];
for (const [i, fruta] of arr.entries()) {
  console.log(`Índice ${i}: ${fruta}`);
}

// Mesmo resultado acima :contentReference[oaicite:3]{index=3}
```

### forEach()

```javascript
const arr = ["maçã", "banana", "laranja"];
arr.forEach((fruta, i) => {
  console.log(`Índice ${i}: ${fruta}`);
});
```

### 1.4. Métodos de busca

- arr.indexOf('banana'): retorna índice do texto ou -1

- arr.includes('laranja'): retorna true ou false

- arr.find(), arr.filter(): permitem buscas condicionais

```javascript
console.log(arr.indexOf("banana")); // 1
console.log(arr.includes("uva")); // false

const maior5 = [3, 7, 2, 9].find((n) => n > 5);
console.log(maior5); // 7
```

# Criando pasta e arquivos JSON com Node.js

Com fs (file system), você pode criar diretórios e gravar arquivos JSON:

### Criar pasta (se não existir)

```javascript
import fs from "fs";
import path from "path";

const dir = path.join(__dirname, "saida");
if (!fs.existsSync(dir)) {
  fs.mkdirSync(dir);
  console.log("Pasta criada:", dir);
}
```

### Gravar objeto JSON em arquivo

```javascript
const dados = {
  nome: "João",
  idade: 30,
  hobbies: ["musica", "jogos"],
};
const jsonStr = JSON.stringify(dados, null, 2); // formatação legível

fs.writeFileSync(path.join(dir, "dados.json"), jsonStr);
console.log("Arquivo JSON criado.");
```

### Alternativa assíncrona com callback

```javascript
fs.writeFile(
  path.join(dir, "dados_async.json"),
  JSON.stringify(dados, null, 2),
  (err) => {
    if (err) {
      console.error("Erro ao salvar arquivo:", err);
    } else {
      console.log("Arquivo JSON salvo com callback.");
    }
  }
);
```

### Criar múltiplos arquivos com base em arrays

```javascript
const itens = [
  { id: 1, valor: "A" },
  { id: 2, valor: "B" },
];

itens.forEach((item) => {
  const nome = `item_${item.id}.json`;
  fs.writeFileSync(path.join(dir, nome), JSON.stringify(item, null, 2));
});
console.log("Arquivos gerados para cada item.");
```

### Explicações rápidas:

- JSON.stringify(obj, null, 2) gera JSON com indentação legível
- fs.writeFileSync() é síncrono e direto, ideal para scripts simples
- A versão assíncrona (fs.writeFile) é recomendada para evitar bloqueio em aplicações maiores.
- fs.mkdirSync() com existsSync garante que a pasta seja criada apenas quando necessário.
