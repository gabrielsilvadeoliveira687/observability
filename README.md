# Observability Lab

Este repositório é destinado ao estudo e prática de **observabilidade em ambientes de TI**, com foco em monitoramento de logs, métricas e performance de aplicações legadas e modernas.

Ele contém exemplos de configuração e implantação de ferramentas como **ELK Stack**, **APM**, **Glowroot** e outros agentes de monitoramento.

---

## 🗂 Estrutura do Repositório

| Pasta | Conteúdo |
|-------|----------|
| `Elastic/ELK-8.17.3/` | Estrutura de containers Docker com Elasticsearch, Logstash e Kibana versão 8.17.3 |
| `Elastic/Apm/` | Agent e configuração do Elastic APM para monitoramento de aplicações Java |
| `Glowroot/` | Agent de monitoramento Glowroot para aplicações legadas (Java 6/7/8) |
| `README.md` | Documentação do repositório e instruções de uso |

---

## ⚙️ Pré-requisitos

- Docker >= 24
- Docker Compose >= 2
- Java 8+ (para aplicações monitoradas via APM Elastic)
- Acesso à rede para integração entre containers
- Caso tenha aplicações Abaixo de Java 8 ate 6, considerar utilizar o Glowroot 
---

## 🚀 Como subir o ambiente

1. Clone o repositório:

```bash
git clone https://github.com/gabrielsilvadeoliveira687/observability.git
cd observability

