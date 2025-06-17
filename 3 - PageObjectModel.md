# Técnicas Avançadas para Sistemas Complexos

Quando se trabalha com aplicações web complexas, é essencial adotar padrões e técnicas que tornem os testes mais maintíveis, escaláveis e confiáveis. O Playwright oferece recursos avançados que permitem implementar essas práticas de forma eficiente.

### Page Object Model (POM)

O Page Object Model é um padrão de design que encapsula os elementos e ações de uma página em uma classe, promovendo reutilização de código e facilitando a manutenção dos testes. Em vez de espalhar seletores e interações por todo o código de teste, você centraliza essa lógica em objetos dedicados.

#### Vantagens do Page Object Model:

- Reutilização: Uma vez definida, a página pode ser usada em múltiplos testes.
- Manutenibilidade: Se um seletor muda, você só precisa atualizar em um lugar.
- Legibilidade: Os testes ficam mais legíveis e expressivos.
- Abstração: Os testes se concentram no "o que" fazer, não no "como" fazer.

Exemplo de implementação:

```javascript
class LoginPage {
  constructor(page) {
    this.page = page;
    this.usernameInput = page.getByTestId("username-input");
    this.passwordInput = page.getByTestId("password-input");
    this.loginButton = page.getByTestId("login-button");
  }

  async goto() {
    await this.page.goto("/login");
  }

  async login(username, password) {
    await this.usernameInput.fill(username);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }
  async expectLoginSuccess() {
    await expect(this.page.url()).toContain("/dashboard");
  }
}
```

No nosso projeto de demonstração, criamos Page Objects para as páginas de Login e Dashboard ( pages/LoginPage.js e pages/DashboardPage.js). Esses objetos encapsulam toda a lógica de interação com os elementos da página, tornando os testes mais limpos e maintíveis.

### Fixtures Personalizadas

Fixtures são uma forma poderosa de configurar e compartilhar recursos entre testes. O Playwright permite criar fixtures personalizadas que podem configurar dados de teste, autenticar usuários, ou preparar o ambiente de teste.

#### Tipos de fixtures úteis:

- Fixtures de página: Instanciam Page Objects automaticamente.
- Fixtures de autenticação: Fazem login automaticamente antes do teste.
- Fixtures de dados: Preparam dados de teste limpos.
- Fixtures de API: Configuram contextos de API autenticados.

Exemplo de fixture personalizada:

```javascript
const test = base.extend({
  authenticatedUser: async ({ page, loginPage, dashboardPage }, use) => {
    await loginPage.goto();
    await loginPage.loginWithTestCredentials();
    await dashboardPage.expectUserLoggedIn("testuser");
    await use({ username: "testuser", page, loginPage, dashboardPage });
  },
});
```

No arquivo fixtures/customFixtures.js, demonstramos como criar fixtures que automatizam o processo de login e fornecem contextos de API autenticados, eliminando código repetitivo nos testes.

### Paralelização e Performance

Para sistemas complexos com muitos testes, a paralelização é crucial para manter os tempos de execução aceitáveis. O Playwright oferece paralelização em múltiplos níveis.

Configurações de paralelização:

- fullyParallel: true: Executa todos os testes em paralelo, independentemente do arquivo.
- workers: Define o número de processos paralelos (workers).
- Projetos separados: Permite executar diferentes conjuntos de testes com configurações específicas.

Exemplo de configuração avançada:

```javascript
module.exports = defineConfig({
  fullyParallel: true,
  workers: process.env.CI ? 2 : 4,
  projects: [
    {
      name: "api-tests",
      testMatch: "**/api.spec.js",
      use: { baseURL: "http://localhost:5000/api" },
    },
    {
      name: "ui-tests",
      testMatch: "**/ui.spec.js",
      use: { baseURL: "https://example.com" },
    },
  ],
});
```

No arquivo playwright.config.advanced.js, demonstramos uma configuração completa que inclui paralelização, múltiplos projetos (incluindo testes mobile), e configurações específicas para diferentes tipos de teste.

### Setup e Teardown Globais

Para sistemas complexos, frequentemente é necessário configurar recursos globais antes de executar os testes e limpá-los depois. Isso pode incluir inicializar bancos de dados, configurar serviços externos, ou verificar a disponibilidade da aplicação.

Global Setup ( global-setup.js):

```javascript
async function globalSetup(config) {
// Verificar se a aplicação está acessível
const browser = await chromium.launch();
const page = await browser.newPage();
await page.goto(config.use.baseURL);
await browser.close();
// Configurar dados de teste
// await setupTestDatabase();
}
Global Teardown ( global-teardown.js):
async function globalTeardown(config) {
// Limpar dados de teste
// await cleanupTestData();
// Gerar relatórios
// await generateCoverageReport();
}
```

Esses arquivos são executados uma única vez antes e depois de toda a suíte de testes, permitindo configurações e limpezas que afetam todos os testes.

### Debugging Avançado

Em sistemas complexos, quando os testes falham, é crucial ter ferramentas de debugging robustas para identificar rapidamente a causa do problema.
Técnicas de debugging avançado:

- Screenshots automáticos: Capturar screenshots em pontos específicos ou em falhas.
- Logs de rede: Monitorar todas as requisições HTTP para identificar problemas de API.
- Traces: Gravar traces detalhados da execução do teste.
- Retry com debugging: Tentar novamente ações que falharam, capturando informações de debug.

Exemplo de utilitário de debugging:

```javascript
class DebugUtils {
  async waitForElementWithDebug(selector, options = {}) {
    try {
      return await this.page.waitForSelector(selector, options);
    } catch (error) {
      // Capturar screenshot e HTML em caso de falha
      await this.captureScreenshot(`element_not_found_$
{selector}`);
      await this.captureHTML(`element_not_found_${selector}`);
      throw error;
    }
  }
}
```

No arquivo utils/debug-utils.js, criamos uma classe completa de utilitários de debugging que inclui captura automática de screenshots, logs de rede, HTML da página, e retry inteligente com debugging.

### Relatórios e Integração com CI/CD

Para sistemas complexos em produção, é essencial ter relatórios detalhados e integração com pipelines de CI/CD.

Configuração de relatórios múltiplos:

```javascript
reporter: [
['html', { outputFolder: 'playwright-report', open:
'never' }],
['json', { outputFile: 'test-results.json' }],
['junit', { outputFile: 'test-results.xml' }],
['line'],
],
```

### Integração com CI/CD:

- Artifacts: Salvar screenshots, vídeos e traces como artifacts do build.
- Métricas: Coletar métricas de performance e cobertura de testes.
- Notificações: Enviar notificações em caso de falhas.
- Deployment gates: Usar resultados dos testes como critério para deploy.

### Estratégias para Testes Estáveis

Em sistemas complexos, a estabilidade dos testes é fundamental. Algumas estratégias incluem:
Auto-espera inteligente: O Playwright já inclui auto-espera, mas você pode estendê-la com lógica personalizada:

```javascript
await expect(async () => {
const count = await page.getByTestId("task-
count").textContent();
expect(parseInt(count)).toBeGreaterThan(0);
}).toPass({ timeout: 10000 });
```

Isolamento de testes: Cada teste deve ser independente e não depender do estado deixado por outros testes:

```javascript
test.beforeEach(async ({ page }) => {
  // Limpar estado da aplicação
  await page.evaluate(() => localStorage.clear());
  await page.evaluate(() => sessionStorage.clear());
});
```

Dados de teste determinísticos: Use dados de teste que não dependem de estado externo:

```javascript
const testUser = {
  username: `testuser_${Date.now()}`,
  email: `test_${Date.now()}@example.com`,
  password: "testpassword123",
};
```

### Monitoramento e Observabilidade

Para sistemas complexos, é importante monitorar a saúde dos testes e identificar padrões de falha:
**Métricas importantes:**

- Taxa de sucesso dos testes
- Tempo médio de execução
- Testes mais instáveis (flaky tests)
- Performance da aplicação durante os testes

Alertas automáticos: Configure alertas para quando a taxa de falha exceder um threshold ou quando testes específicos falharem consistentemente.
Essas técnicas avançadas, quando aplicadas corretamente, transformam uma suíte de testes básica em um sistema robusto e confiável, capaz de suportar o desenvolvimento e manutenção de aplicações web complexas em escala empresarial.
