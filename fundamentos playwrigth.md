# Dominando o Playwright: Fundamentos Essenciais

## Introdução ao Playwright

O Playwright é uma poderosa biblioteca de automação de navegadores desenvolvida pela Microsoft. Ele permite automatizar navegadores Chromium, Firefox e WebKit com uma única API, oferecendo recursos robustos para testes end-to-end, web scraping e automação geral de tarefas web. Sua arquitetura moderna e foco em estabilidade e velocidade o tornam uma excelente escolha para sistemas complexos.

### Por que Playwright?

*   **Multi-navegador:** Suporta os três principais motores de navegador (Chromium, Firefox, WebKit) com uma única API, garantindo compatibilidade e reduzindo a necessidade de testes em múltiplas ferramentas.
*   **Multi-linguagem:** Oferece APIs para JavaScript/TypeScript, Python, Java e .NET, permitindo que equipes com diferentes stacks tecnológicas o utilizem.
*   **Auto-espera:** O Playwright espera automaticamente pelos elementos estarem prontos antes de realizar ações, eliminando a necessidade de sleeps manuais e tornando os testes mais estáveis.
*   **Isolamento de contexto:** Cada teste é executado em um contexto de navegador isolado, garantindo que os testes sejam independentes e não interfiram uns nos outros.
*   **Recursos avançados:** Suporte nativo para interceptação de rede, emulação de dispositivos móveis, geolocalização, permissões, downloads, uploads e muito mais.
*   **Ferramentas de depuração:** Inclui ferramentas como o Playwright Inspector, que facilita a depuração de testes e a identificação de seletores.

## Seletores e Localização de Elementos

Para interagir com elementos em uma página web, o Playwright utiliza seletores. Seletores são strings que descrevem como encontrar um elemento específico na DOM (Document Object Model). O Playwright oferece uma variedade de tipos de seletores, permitindo flexibilidade e robustez na localização de elementos.

![Playwright Selectors](upload/search_images/upakEhQUeZsw.jpg)

### Tipos de Seletores Comuns

O Playwright suporta diversos tipos de seletores, incluindo:

*   **Seletores CSS:** São os seletores mais comuns e versáteis, baseados nas regras de estilo CSS. Permitem selecionar elementos por tag, classe, ID, atributos, hierarquia, etc.
    *   Exemplo: `page.locator("button.login-button")`
*   **Seletores XPath:** Uma linguagem poderosa para navegar e selecionar nós em documentos XML (e HTML). Útil para cenários mais complexos onde seletores CSS podem ser limitados.
    *   Exemplo: `page.locator("//button[text()=\'Entrar\']")`
*   **Seletores de Texto:** Permitem encontrar elementos com base no texto visível que contêm. Muito útil para elementos que não possuem IDs ou classes estáveis.
    *   Exemplo: `page.getByText("Entrar")`
*   **Seletores de Role (ARIA):** Selecionam elementos com base em seu papel de acessibilidade (ARIA role). Isso torna os testes mais robustos e acessíveis, pois se baseiam na semântica do elemento.
    *   Exemplo: `page.getByRole("button", { name: "Entrar" })`
*   **Seletores de Test ID:** Uma prática recomendada é adicionar atributos `data-testid` aos elementos da UI. Isso cria seletores estáveis que não são afetados por mudanças de estilo ou estrutura da DOM.
    *   Exemplo: `page.getByTestId("login-button")`
*   **Seletores de Placeholder:** Seleciona elementos de input com base no texto do placeholder.
    *   Exemplo: `page.getByPlaceholder("Digite seu usuário")`
*   **Seletores de Label:** Seleciona elementos de input associados a um determinado texto de label.
    *   Exemplo: `page.getByLabel("Usuário")`

### Prioridade e Boas Práticas na Escolha de Seletores

Ao escolher um seletor, é importante considerar a robustez e a legibilidade do teste. A ordem de preferência geralmente é:

1.  **`data-testid`:** Ideal para testes, pois são criados especificamente para automação e não são afetados por mudanças na UI ou no CSS. No nosso site de exemplo, utilizamos `data-testid` para os campos de input e botões de login/registro.
2.  **`getByRole`:** Excelente para acessibilidade e semântica. Garante que o teste interaja com o elemento da mesma forma que um usuário com tecnologias assistivas.
3.  **`getByText` / `getByLabel` / `getByPlaceholder`:** Bons para elementos que possuem texto visível ou labels claras. Menos frágeis que seletores CSS baseados em classes voláteis.
4.  **Seletores CSS (com IDs ou classes estáveis):** Se não houver `data-testid` ou role, use seletores CSS com IDs (se únicos e estáveis) ou classes que não mudam frequentemente.
5.  **XPath:** Use XPath como último recurso, para cenários muito específicos onde outros seletores não são suficientes. XPath pode ser mais frágil e menos legível.

### Exemplo Prático de Seletores no Nosso Sistema de Testes

No nosso sistema de testes, a página de login possui os seguintes elementos com `data-testid`:

*   Campo de usuário: `data-testid="username-input"`
*   Campo de senha: `data-testid="password-input"`
*   Botão de login: `data-testid="login-button"`
*   Botão de preencher credenciais de teste: `data-testid="fill-test-credentials"`

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

![Playwright Interactions](upload/search_images/7CXG1PhFV3di.png)

### Ações Comuns

*   **`click()`:** Clica em um elemento. O Playwright espera automaticamente que o elemento esteja visível, habilitado e receba eventos de clique.
    *   Exemplo: `await page.getByTestId("login-button").click();`
*   **`fill(value)`:** Preenche um campo de texto com o valor especificado. Limpa o campo antes de digitar.
    *   Exemplo: `await page.getByTestId("username-input").fill("testuser");`
*   **`type(text, [options])`:** Digita texto caractere por caractere, simulando a digitação humana. Útil para testar validações de input em tempo real.
    *   Exemplo: `await page.getByTestId("password-input").type("test123");`
*   **`press(key)`:** Simula o pressionar de uma tecla ou combinação de teclas.
    *   Exemplo: `await page.getByTestId("username-input").press("Enter");`
*   **`check()` / `uncheck()`:** Marca ou desmarca checkboxes e radio buttons.
    *   Exemplo: `await page.getByLabel("Lembrar-me").check();`
*   **`selectOption(value)`:** Seleciona uma opção em um elemento `<select>` (dropdown) pelo valor, label ou índice.
    *   Exemplo: `await page.getByLabel("País").selectOption("BR");`
*   **`hover()`:** Move o mouse sobre um elemento, útil para testar tooltips ou menus que aparecem ao passar o mouse.
    *   Exemplo: `await page.getByText("Meu Perfil").hover();`

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

![Playwright Waits](upload/search_images/aA8KgDK9UoxH.jpg)

### Navegação

*   **`page.goto(url, [options])`:** Navega para uma URL específica. Por padrão, o Playwright espera até que o evento `load` da página seja disparado, indicando que a página e todos os seus recursos foram carregados.
    *   Exemplo: `await page.goto("/login");` (usando `baseURL` configurada no `playwright.config.js`)
*   **`page.goBack()` / `page.goForward()`:** Navega para a página anterior ou próxima no histórico do navegador.

### Auto-Espera (Auto-waiting)

O Playwright é inteligente o suficiente para esperar automaticamente por elementos estarem visíveis, habilitados e anexados à DOM antes de realizar uma ação. Isso significa que você raramente precisará adicionar esperas explícitas para a maioria das operações. Por exemplo, quando você chama `element.click()`, o Playwright fará o seguinte:

1.  Espera que o elemento esteja na DOM.
2.  Espera que o elemento esteja visível.
3.  Espera que o elemento esteja habilitado.
4.  Espera que o elemento esteja estável (não se movendo).
5.  Espera que o elemento receba eventos de clique no ponto de clique.

### Esperas Explícitas (quando necessário)

Embora a auto-espera seja poderosa, há situações em que você pode precisar de esperas explícitas:

*   **`page.waitForSelector(selector, [options])`:** Espera que um seletor apareça na DOM e atenda a certas condições (visível, anexado, etc.).
    *   Exemplo: `await page.waitForSelector(".dashboard-loaded");`
*   **`page.waitForLoadState(state)`:** Espera que a página atinja um determinado estado de carregamento (`load`, `domcontentloaded`, `networkidle`).
    *   Exemplo: `await page.waitForLoadState("networkidle");`
*   **`page.waitForURL(url, [options])`:** Espera que a URL da página mude para uma URL específica ou corresponda a um padrão.
    *   Exemplo: `await page.waitForURL("/dashboard");`
*   **`expect(locator).toBeVisible()` / `expect(locator).toBeEnabled()` / `expect(locator).toHaveText()`:** Usar asserções do `expect` com auto-espera é a forma preferida de esperar por um estado específico do elemento. O Playwright tentará repetidamente até que a condição seja satisfeita ou o timeout seja atingido.
    *   Exemplo: `await expect(page.url()).toContain("/dashboard");`
    *   Exemplo: `await expect(page.getByTestId("user-name")).toHaveText("testuser");`

No nosso teste de login, após o clique no botão, usamos `expect(page.url()).toContain("/dashboard")` e `expect(page.getByTestId("user-name")).toHaveText("testuser")`. Essas asserções não apenas verificam o estado final da aplicação, mas também atuam como esperas, garantindo que o Playwright aguarde até que a URL mude e o nome de usuário esteja visível no dashboard antes de prosseguir.

## Captura de Screenshots e Vídeos

Playwright oferece recursos integrados para capturar screenshots e gravar vídeos dos testes. Isso é extremamente útil para depuração, relatórios de bugs e documentação.

### Screenshots

*   **`page.screenshot([options])`:** Captura uma screenshot da página inteira ou de uma parte específica.
    *   Exemplo: `await page.screenshot({ path: 'screenshot.png' });`
*   **`locator.screenshot([options])`:** Captura uma screenshot de um elemento específico.
    *   Exemplo: `await page.getByTestId("login-card").screenshot({ path: 'login-card.png' });`

Opções comuns para screenshots:

*   `path`: Caminho onde a screenshot será salva.
*   `fullPage`: `true` para capturar a página inteira (rolando se necessário), `false` para apenas a viewport visível (padrão).
*   `clip`: Um objeto com `x`, `y`, `width`, `height` para capturar uma região específica.

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

*   `'off'`: Não grava vídeos (padrão).
*   `'on'`: Grava vídeo para todos os testes.
*   `'retain-on-failure'`: Grava vídeo apenas para testes que falham.
*   `'on-first-retry'`: Grava vídeo para o primeiro retry de um teste que falha.

Os vídeos são salvos automaticamente em um diretório temporário e, se configurado para reter, são movidos para o diretório de resultados do Playwright (geralmente `test-results/`).

### Uso em Testes

Você pode adicionar uma linha para capturar uma screenshot em um ponto específico do seu teste para depuração:

```javascript
test("should display an error message with invalid credentials", async ({ page }) => {
  await page.getByTestId("username-input").fill("invaliduser");
  await page.getByTestId("password-input").fill("wrongpassword");
  await page.getByTestId("login-button").click();

  // Captura uma screenshot após a tentativa de login falha
  await page.screenshot({ path: 'screenshots/login-failed.png' });

  await expect(page.locator(".bg-red-50")).toBeVisible();
  await expect(page.locator(".bg-red-50")).toContainText("Credenciais inválidas");
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

*   **Preenchimento de campos:** Use `fill()` para preencher inputs, textareas e selects. Para campos com validação em tempo real, `type()` pode ser mais adequado para simular a digitação caractere por caractere.
*   **Interação com elementos dinâmicos:** Se campos ou seções do formulário aparecem ou desaparecem com base em seleções anteriores, use `waitForSelector()` ou asserções `expect().toBeVisible()` para garantir que o elemento esteja pronto antes de interagir.
*   **Validação de mensagens de erro:** Após submeter um formulário com dados inválidos, você pode verificar as mensagens de erro exibidas na UI.

No nosso dashboard, temos filtros de tarefas que simulam uma interação com formulários:

```javascript
test("should be able to filter tasks by status", async ({ page }) => {
  await page.getByTestId("filter-pending").click();
  await expect(page.get
(Content truncated due to size limit. Use line ranges to read in chunks)
```


# Fundamentos: Condicionais e Estruturas de Repetição

## Condicionais (if, else, else if)

As condicionais permitem que o programa tome decisões com base em condições booleanas.

### Exemplo básico com if/else:

```javascript
const idade = 20;
if (idade >= 18) {
    console.log('Maior de idade');
} else {
    console.log('Menor de idade');
}
```

Exemplo com else if:

```javascript
const nota = 75;
if (nota >= 90) {
    console.log('Excelente');
} else if (nota >= 70) {
    console.log('Bom');
} else {
    console.log('Precisa melhorar');
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
const frutas = ['maçã', 'banana', 'laranja'];
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
    console.log('Execução cancelada');
}
```
