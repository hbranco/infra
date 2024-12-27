
 **instalação do New Relic Infrastructure** no **Ubuntu 20.04 Server** e a configuração para monitorar o **MySQL**.

---

## **1. Pré-requisitos**

Antes de iniciar, certifique-se de ter:

- Um servidor Ubuntu 20.04 com acesso root ou usuário sudo.
- O MySQL já instalado e configurado no servidor.

---

## **2. Instalação do New Relic Infrastructure**

### **2.1 Adicione o repositório do New Relic**

Execute os seguintes comandos para adicionar o repositório oficial:

```bash
curl -o /tmp/newrelic-infra.gpg https://download.newrelic.com/infrastructure_agent/gpg/newrelic-infra.gpg
sudo gpg --dearmor -o /usr/share/keyrings/newrelic-infra-archive-keyring.gpg /tmp/newrelic-infra.gpg

echo "deb [signed-by=/usr/share/keyrings/newrelic-infra-archive-keyring.gpg] https://download.newrelic.com/infrastructure_agent/linux/apt focal main" | sudo tee /etc/apt/sources.list.d/newrelic-infra.list
```

### **2.2 Atualize o apt e instale o agente**

Atualize os pacotes e instale o agente do New Relic Infrastructure:

```bash
sudo apt-get update
sudo apt-get install newrelic-infra -y
```

---

## **3. Configuração do agente**

### **3.1 Adicione a chave de licença**

Edite o arquivo de configuração do agente:

```bash
sudo nano /etc/newrelic-infra.yml
```

Adicione sua chave de licença no arquivo:

```yaml
license_key: LICENSE_KEY
log_level: off
```


---

## **4. Instalação e configuração do MySQL Monitoring**

### **4.1 Instale o plugin do MySQL**

O agente utiliza um **plugin** para monitorar o MySQL. Instale o plugin com o comando:

```bash
sudo apt-get install newrelic-infra-plugins -y
```

### **4.2 Configure o MySQL para o New Relic**

Crie um arquivo de configuração para o MySQL:

```bash
sudo nano /etc/newrelic-infra/integrations.d/mysql-config.yml
```

Adicione o seguinte conteúdo no arquivo:

```yaml
integrations:
  - name: nri-mysql
    env:
      MYSQL_HOST: 127.0.0.1
      MYSQL_PORT: 3306
      MYSQL_USER: 'root'
      MYSQL_PASS: 'sua_senha_mysql'
      MYSQL_DATABASE: 'mysql'
    labels:
      environment: production
      role: mysql-server
```



- Substitua `sua_senha_mysql` pela senha do usuário **root** do MySQL.
- Caso seu MySQL use outro usuário com permissão de leitura, ajuste os valores de `MYSQL_USER` e `MYSQL_PASS`.

### **4.3 Conceda permissões no MySQL**

No MySQL, crie um usuário dedicado ao New Relic e conceda permissões de leitura:

```sql
CREATE USER 'newrelic'@'localhost' IDENTIFIED BY 'newrelic_password';
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'newrelic'@'localhost';
FLUSH PRIVILEGES;
```

- Substitua `'newrelic_password'` pela senha que desejar.

Ajuste o arquivo de configuração do MySQL para usar esse novo usuário:

```yaml
MYSQL_USER: 'newrelic'
MYSQL_PASS: 'newrelic_password'
```

---
## **4.4** Instalar o Integrador do PostgreSQL**

### Baixar o pacote do integrador

bash

Copiar código

```bash
sudo apt install nri-postgresql -y
```

### Configurar o integrador

Edite o arquivo de configuração do PostgreSQL:

bash

Copiar código

```bash
sudo nano /etc/newrelic-infra/integrations.d/postgresql-config.yml
```

```yaml
integrations:
	- name: nri-postgresql
	config: 
		hostname:localhost
		port: 5432 
		username: newrelic 
		password: <YOUR_DB_PASSWORD> 
		database: postgres
```

Substitua `<YOUR_DB_PASSWORD>` pela senha do usuário `newrelic`.

### Criar um usuário para monitoramento no PostgreSQL

Acesse o banco de dados como `postgres`:

```bash 
sudo -u postgres psql
```

Crie o usuário e atribua permissões:

Copiar código

```sql
CREATE ROLE newrelic WITH LOGIN PASSWORD '<YOUR_DB_PASSWORD>';
GRANT CONNECT ON DATABASE postgres TO newrelic; 
GRANT USAGE ON SCHEMA public TO newrelic; 
GRANT SELECT ON ALL TABLES IN SCHEMA public TO newrelic; 
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO newrelic;
```

---

## **5. Reinicie o agente**

Após as configurações, reinicie o agente do New Relic Infrastructure:

```bash
sudo systemctl restart newrelic-infra
```
---

## **6. Verifique a instalação**

### **6.1 Verifique o status do agente**

Execute o seguinte comando:

```bash
sudo systemctl status newrelic-infra
```

O status deve indicar que o serviço está ativo e rodando.

### **6.2 Verifique os logs do agente**

Se precisar verificar os logs, altera o log_level no arquivo de configuração para "debug" e use:

```bash
sudo tail -f /var/log/newrelic-infra/newrelic-infra.log
```
---

### 6.2 Verificar a Integração do PostgreSQL
   Testar diretamente o integrador do PostgreSQL
   Execute o binário do integrador manualmente para verificar se ele está conseguindo coletar os dados:
```bash
sudo /var/db/newrelic-infra/newrelic-integrations/bin/nri-postgresql --config_path /etc/newrelic-infra/integrations.d/postgresql-config.yml
```
**Resultados esperados:**

Se a integração estiver configurada corretamente, você verá uma saída JSON contendo métricas coletadas, como conexões ativas, consultas por segundo, etc.
Se houver erros, eles serão exibidos. Verifique as permissões no PostgreSQL, a senha do usuário e a configuração de rede.

### 6.3 Verificar a Integração do MySQL
   Testar diretamente o integrador do MySQL
   Execute o binário do integrador manualmente:

```bash
sudo /var/db/newrelic-infra/newrelic-integrations/bin/nri-mysql --config_path /etc/newrelic-infra/integrations.d/mysql-config.yml
```
**Resultados esperados:**
Uma saída JSON será exibida com métricas como o número de conexões ativas, o tempo de execução de consultas, etc.
Se houver erros, verifique o arquivo de configuração, a conectividade com o banco de dados e as permissões do usuário.