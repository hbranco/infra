
## **1. Instalação do agente da New Relic**

Execute o comando abaixo no diretório raiz da sua aplicação para instalar a versão 6.13.0 do agente da New Relic compátivel com a versão 10.14.1 do NodeJs:

```bash
npm install newrelic@6.13.0 --save
```

---

## **2. Configurando a New Relic**

### **2.1 Crie o arquivo `newrelic.js`**

Na raiz do projeto, crie um arquivo chamado **`newrelic.js`** com o seguinte conteúdo:

```javascript
'use strict';

exports.config = {
    app_name: ['APP_NAME_AQUI'],
    license_key: 'LICENÇA_AQUI',
    logging: {
        level: 'off', // ou 'warn' para menos detalhes
    },
    distributed_tracing: {
        enabled: true,
    },
    transaction_tracer: {
        enabled: true,
        transaction_threshold: 'apdex_f', // Apenas transações mais lentas serão registradas.
        record_sql: 'raw', // Coleta de queries SQL completas para o trace.
        slow_sql: {
            enabled: true,
            max_samples: 100
        },
        stack_trace_threshold: 0.5, // Inclui rastros apenas para transações com duração > 500ms.
    },
    custom_insights_events: {
        enabled: false, // Desative eventos personalizados, caso não sejam críticos.
    },
    event_harvest_config: {
        harvest_limits: {
            analytic_event_data: 1000, // Limite de eventos analíticos.
            custom_event_data: 500,   // Limite de eventos personalizados.
            error_event_data: 100,    // Limite de eventos de erro.
            span_event_data: 2000,    // Limite de spans.
        },
    },
    error_collector: {
        enabled: true,
        ignore_status_codes: [400, 404], // Ignore erros de cliente ou recursos não encontrados.
    },
};
```

---

## **3. Carregando o agente no projeto**

O agente da New Relic **deve ser carregado antes de qualquer outro módulo da aplicação**. Edite o arquivo principal da sua aplicação (**ex.: `index.js` ou `server.js`**) e adicione a seguinte linha no topo do arquivo:

```javascript
require('newrelic');
```

Exemplo de `server.js`:

```javascript
require('newrelic'); // Carrega o agente da New Relic
const express = require('express');
const app = express();

app.get('/', (req, res) => {
    res.send('Hello World!');
});

app.listen(3000, () => {
    console.log('Aplicação rodando na porta 3000');
});
```


# NÃO EXECUTAR A NEWRELIC NO AMBIENTE LOCAL OU DE TESTES/Q.A/HOMOLOGAÇÃO
### **Exemplo de código para evitar execução local**

.env
```.env
NEW_RELIC_ENABLED=true
```

Copiar código

```javascript
// Apenas carregue o New Relic se não for ambiente de desenvolvimento
if (process.env.NEW_RELIC_ENABLED === 'true') {  
	require('newrelic');  // Carrega o New Relic
}
// O resto da inicialização do seu aplicativo Node.js 
const express = require('express'); const app = express();  // Suas rotas e outras configurações aqui
```

Neste exemplo:

- O New Relic será carregado apenas se a variável de ambiente **`NEW_RELIC_ENABLED`** for igual a `'true'`.
- Se o ambiente for **`development`** (ou outro valor que você definir para seu ambiente local ou testes), o New Relic não deve ser carregado.

---

## **4. Defina a variável de ambiente**

Configure a variável de ambiente `NEW_RELIC_LICENSE_KEY` com a chave de licença da New Relic.

### No Linux em produção:

```bash
export NEW_RELIC_LICENSE_KEY='LICENÇA_AQUI'
export NEW_RELIC_APP_NAME=APP_NAME_AQUI
export NEW_RELIC_ENABLED=true
```

---

## **5. Inicie a aplicação**

Execute a aplicação normalmente. Por exemplo:

```bash
node server.js
```

O agente da New Relic será iniciado e começará a reportar métricas automaticamente para o painel da New Relic.

---

Pronto! Agora sua aplicação Node.js está integrada com o agente da New Relic e enviando métricas de performance. 🎉