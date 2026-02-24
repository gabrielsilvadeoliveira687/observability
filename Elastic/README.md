# 📘 Documentação – Integração Elastic APM com Servidores Java

### Este documento apresenta um resumo da estruturação do projeto, utilizando como base documentações oficiais do framework e documentação interna já validada em ambiente real.

## INTRODUÇÃO
- O objetivo é padronizar e facilitar a configuração, validação e troubleshooting do Elastic APM em ambientes Java legados e modernos.
- As versoes podem ser alteradas, pois utlizamos a docker compose, e tambem onde sera referenciado 


- Consideracoes a serem a levadas logo abaixo e tambem verificacao de suporte para monitorar 

### 🔎 CHECKLIST GERAL (vale para TODOS)
- Faça isso antes de olhar Tomcat/JBoss/WildFly:

### 1️⃣ Java compatível

- Você já confirmou:
```bash
java -version
```
- openjdk version "1.8.0_472"


### ✅ OK — o APM 1.23.x suporta Java 7+ (inclusive Java 8).

## 2️⃣ Jar do agent está íntegro
```bash
ls -lh /opt/apm/elastic-apm-agent-1.23.0.jar
jar tf /opt/apm/elastic-apm-agent-1.23.0.jar > /dev/null
```

## Se não der erro, o JAR está OK.
## ❌ Se der erro → JAR corrompido (baixa de novo).

### 3️⃣ Permissão
```bash
ls -l /opt/apm/elastic-apm-agent-1.23.0.jar
```

### Precisa ter ao menos:

- **-rw-r--r--**

### 4️⃣ Teste isolado do agent (IMPORTANTE)

- Antes de subir app, teste só o agent:
```bash
java -javaagent:/opt/apm/elastic-apm-agent-1.23.0.jar -version
```

👉 Resultado esperado:

Mostra a versão do Java

Não pode crashar

❌ Se cair aqui → o problema é o agent (não o app).

🟢 AGORA VAMOS POR TIPO DE SERVIDOR
🐱 TOMCAT
Passo 1 – Onde configurar

Arquivo:

$CATALINA_HOME/bin/setenv.sh


Se não existir, crie.

Passo 2 – Configuração mínima (SEM package ainda)

Comece simples, sem application_packages:
```bash
export CATALINA_OPTS="
-javaagent:/opt/apm/elastic-apm-agent-1.23.0.jar
-Delastic.apm.service_name=tomcat-test
-Delastic.apm.server_urls=http://localhost ou ip do host:8200
-Delastic.apm.environment=homolog
-Delastic.apm.log_level=DEBUG
"
```

❗ Não use config file no primeiro teste

Passo 3 – Subir e olhar log
catalina.sh run


E em outro terminal:
```bash
tail -f $CATALINA_HOME/logs/catalina.out
```

## 🔎 Procure por:

Elastic APM agent started

ERROR ou Exception

📌 Se Tomcat não sobe, quase sempre aparece:

- conflito de javaagent
- Java errado
- permissão no jar

🐗 JBOSS 5 (SEU CASO MAIS SENSÍVEL)

JBoss 5 é chato com agent, então siga isso à risca.

Passo 1 – Onde configurar
Arquivo:

/opt//jboss-5.1.0.GA/bin/run.conf

Passo 2 – Configuração mínima

### ⚠️ Não use config_file nem application_packages ainda


```bash
JAVA_OPTS="$JAVA_OPTS \
-javaagent:/opt/apm/elastic-apm-agent-1.23.0.jar \
-Delastic.apm.service_name= \
-Delastic.apm.server_urls=http://localhost ou ip do host:8200 \
-Delastic.apm.environment=prd \
-Delastic.apm.disable_bootstrap_checks=true \
-Delastic.apm.log_level=DEBUG"
```
- Passo 3 – Subir em foreground
./run.sh -c diameterro

- Passo 4 – Log do APM

** O agent cria log em:**
```bash
/opt/jboss-server/log/elastic-apm.log
```

Ou:

/opt/apm/logs/elastic-apm.log


Procure por:

Agent started
Instrumentation failed
Unsupported class version

📌 JBoss 5 costuma falhar se application_packages estiver errado
👉 por isso não use ainda.

### 🦅 WILDFLY
## Passo 1 – Onde configurar
##$WILDFLY_HOME/bin/standalone.conf

Passo 2 – Configuração mínima
```bash
JAVA_OPTS="$JAVA_OPTS \
-javaagent:/opt/apm/elastic-apm-agent-1.23.0.jar \
-Delastic.apm.service_name=wildfly-test \
-Delastic.apm.server_urls=http://localhost ou ip do host:8200 \
-Delastic.apm.environment=homolog \
-Delastic.apm.log_level=DEBUG"
```
## Passo 3 – Subir
./standalone.sh


E observar:

tail -f standalone/log/server.log

⚙️ SERVICE (systemd / init.d)

Se for serviço:

1️⃣ Descubra o service
systemctl status nome-do-service

2️⃣ Veja como o Java sobe
systemctl cat nome-do-service


Ou:

ps -ef | grep java

# 3️⃣ Confirme se o -javaagent está realmente presente

- Se não estiver, o service não está usando suas variáveis.
- 🚨 QUANDO A APLICAÇÃO NÃO SOBE – ORDEM DE DEBUG

1️⃣ Testar:

java -javaagent:... -version


2️⃣ Subir SEM:

application_packages

config_file

3️⃣ Ativar:

-Delastic.apm.log_level=DEBUG


4️⃣ Ver elastic-apm.log

📌 SOBRE application_packages

Só configure depois que subir.

Para achar corretamente:
```bash
jar tf seuapp.jar | grep ".class" | head
```

Exemplo:

br/com/api/modules/service


Então:

-Delastic.apm.application_packages+


❌ Se errar → agent sobe, mas não instrumenta nada
❌ Em JBoss 5 → pode quebrar startup
