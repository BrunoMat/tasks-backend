# Testes de API com Playwright

O Playwright não é apenas uma ferramenta para automação de navegadores; ele também oferece recursos robustos para testes de API. Isso permite que você teste tanto o frontend quanto o backend da sua aplicação usando uma única ferramenta, garantindo uma cobertura de teste mais abrangente.

## Configuração de Testes de API

Para realizar testes de API com Playwright, você utiliza o objeto request que permite fazer requisições HTTP diretamente, sem a necessidade de um navegador. Isso torna os testes de API mais rápidos e eficientes.

```javascript
const { test, expect, request } = require("@playwright/test");
test.describe("API Testing with Playwright", () => {
  let apiContext;
  test.beforeAll(async ({ playwright }) => {
    // Criar um contexto de API para fazer requisições HTTP
    apiContext = await playwright.request.newContext({
      baseURL: "https://lnh8imcddg1g.manus.space/api",
      extraHTTPHeaders: {
        "Content-Type": "application/json",
      },
    });
  });
  test.afterAll(async () => {
    // Limpar o contexto de API após todos os testes
    await apiContext.dispose();
  });
});
```

O apiContext é um contexto de requisição que mantém configurações como URL
base, headers padrão e cookies de sessão entre as requisições. Isso é especialmente útil
para APIs que requerem autenticação baseada em sessão.

### Autenticação em APIs

Para APIs que requerem autenticação, você pode fazer login uma vez e reutilizar a sessão para testes subsequentes. No nosso sistema de demonstração, a autenticação é baseada em cookies de sessão.

```javascript
test("should be able to login via API", async () => {
  const response = await apiContext.post("/login", {
    data: {
      username: "testuser",
      password: "test123",
    },
  });
  expect(response.status()).toBe(200);
  const responseBody = await response.json();
  expect(responseBody.message).toBe("Login realizado com sucesso");
  expect(responseBody.user).toHaveProperty("id");
  expect(responseBody.user.username).toBe("testuser");
});
```

Após fazer login, o apiContext automaticamente mantém os cookies de sessão,
permitindo que requisições subsequentes sejam autenticadas automaticamente.

### Validação de Responses

Os testes de API devem validar não apenas o código de status HTTP, mas também a estrutura e o conteúdo da resposta. O Playwright oferece asserções robustas para isso.

```javascript
test("should be able to get dashboard stats after login", async() => {
    // Primeiro, fazer login para obter cookies de sessão
    const loginResponse = await apiContext.post("/login", {
        data: {
            username: "testuser",
            password: "test123",
        },
    });
    expect(loginResponse.status()).toBe(200);
    // Agora, buscar estatísticas do dashboard
    const statsResponse = await apiContext.get("/dashboard/
    stats");
    expect(statsResponse.status()).toBe(200);
    const statsData = await statsResponse.json();
    expect(statsData).toHaveProperty("total_tasks");
    expect(statsData).toHaveProperty("pending_tasks");
    expect(statsData).toHaveProperty("in_progress_tasks");
    expect(statsData).toHaveProperty("completed_tasks");
    expect(statsData).toHaveProperty("completion_rate");
    // Verificar se os valores são números
    expect(typeof statsData.total_tasks).toBe("number");
    expect(typeof statsData.pending_tasks).toBe("number");
    expect(typeof statsData.in_progress_tasks).toBe("number");
    expect(typeof statsData.completed_tasks).toBe("number");
    expect(typeof statsData.completion_rate).toBe("number");
});
```

Este teste não apenas verifica se a API retorna um status 200, mas também valida que a
resposta contém todas as propriedades esperadas e que elas são do tipo correto.

### Testes CRUD Completos

Uma das vantagens dos testes de API é a capacidade de testar operações CRUD (Create,Read, Update, Delete) de forma isolada e eficiente.

```javascript
test("should be able to create a new task via API", async () => {
  // Primeiro, fazer login para obter cookies de sessão
  const loginResponse = await apiContext.post("/login", {
    data: {
      username: "testuser",
      password: "test123",
    },
  });
  expect(loginResponse.status()).toBe(200);
  // Criar uma nova tarefa
  const newTaskResponse = await apiContext.post("/tasks", {
    data: {
      title: "Nova Tarefa via API",
      description:
        "Esta tarefa foi criada através de um teste de API do Playwright",
      priority: "high",
    },
  });
  expect(newTaskResponse.status()).toBe(201);
  const newTaskData = await newTaskResponse.json();
  expect(newTaskData).toHaveProperty("id");
  expect(newTaskData.title).toBe("Nova Tarefa via API");
  expect(newTaskData.description).toBe(
    "Esta tarefa foi criada através de um teste de API do Playwright"
  );
  expect(newTaskData.priority).toBe("high");
  expect(newTaskData.status).toBe("pending"); // Status padrão
});
```

### Testes de Validação e Tratamento de Erros

É crucial testar não apenas os cenários de sucesso, mas também como a API lida com dados inválidos e situações de erro.

```javascript
test("should return error for invalid login credentials", async () => {
  const response = await apiContext.post("/login", {
    data: {
      username: "invaliduser",
      password: "wrongpassword",
    },
  });
  expect(response.status()).toBe(401);
  const responseBody = await response.json();
  expect(responseBody.error).toBe("Credenciais inválidas");
});
test("should return 401 for protected endpoints without authentication", async () => {
  // Tentar acessar endpoint protegido sem fazer login
  const response = await apiContext.get("/dashboard/stats");
  expect(response.status()).toBe(401);
  const responseBody = await response.json();
  expect(responseBody.error).toBe("Não autenticado");
});
```

### Testes de Performance Básicos

Embora o Playwright não seja uma ferramenta de teste de performance dedicada, você pode incluir verificações básicas de tempo de resposta nos seus testes de API.

```javascript
test("should respond within acceptable time", async () => {
  const startTime = Date.now();
  const response = await apiContext.get("/tasks");
  const endTime = Date.now();
  const responseTime = endTime - startTime;
  expect(response.status()).toBe(200);
  expect(responseTime).toBeLessThan(2000); // Resposta em menos de 2 segundos
});
```

### Integração entre Testes de UI e API

Uma das grandes vantagens do Playwright é a capacidade de combinar testes de UI e API no mesmo framework. Você pode, por exemplo, criar dados via API e depois verificar se eles aparecem corretamente na interface do usuário.

```javascript
test("should display task created via API in the UI", async ({ page }) => {
  // Criar uma tarefa via API
  const loginResponse = await apiContext.post("/login", {
    data: { username: "testuser", password: "test123" },
  });
  const taskResponse = await apiContext.post("/tasks", {
    data: {
      title: "Tarefa Criada via API para UI",
      description: "Esta tarefa deve aparecer na interface",
      priority: "medium",
    },
  });
  const newTask = await taskResponse.json();
  // Agora verificar na UI
  await page.goto("/login");
  await page.getByTestId("username-input").fill("testuser");
  await page.getByTestId("password-input").fill("test123");
  await page.getByTestId("login-button").click();
  await expect(page.url()).toContain("/dashboard");
  await expect(page.getByText("Tarefa Criada via API para UI")).toBeVisible();
});
```

Este tipo de teste híbrido garante que a integração entre frontend e backend está funcionando corretamente, validando o fluxo completo da aplicação.
