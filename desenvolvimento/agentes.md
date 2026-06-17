# Estrutura Completa de Desenvolvimento para Cursor IA

Este arquivo consolida os Requisitos do Sistema, as Regras de Desenvolvimento (Django/Migração) e as Instruções dos Agentes de IA para guiar o Cursor na construção do software.

---

## 1. Requisitos do Sistema e Arquitetura

### 1.1 Performance e Concorrência (Async)
* **Objetivo:** Suportar no mínimo 200 acessos simultâneos sem gargalos ou degradação de performance (*overload*).
* **Diretriz Técnica:**
  * Toda a arquitetura de Views, chamadas de API, processamento interno e consultas ao banco de dados no Django devem utilizar, obrigatoriamente, funções assíncronas (`async/await` / `ASGI`).
  * Evitar funções síncronas bloqueantes nas rotas principais de tráfego.

### 1.2 Arquitetura Multitenant (Isolamento Absoluto de Dados)
* **Objetivo:** Blindagem total de dados. Usuários de uma empresa jamais podem acessar dados de outra empresa.
* **Diretriz Técnica:**
  * **Banco de Dados Único com Isolamento por Chave Estrangeira:** Todas as tabelas do sistema devem possuir obrigatoriamente a coluna `empresa_id`.
  * **Middleware/Manager Automático:** Implementar um Django *Manager* personalizado ou um *Middleware* global que intercepte todas as queries do ORM para injetar automaticamente o filtro `filter(empresa_id=...)` baseado na sessão do usuário logado. Isso evita falhas humanas e esquecimentos no código.

### 1.3 Gestão de Licenças e Gateways de Pagamento
* **Objetivo:** Controle automatizado de bloqueio/desbloqueio por inadimplência com flexibilidade total de contas.
* **Regras de Negócio:**
  * Cada empresa possui os campos `data_de_pagamento` e `data_de_vencimento`. Se a data atual for maior que o vencimento, o acesso é bloqueado.
  * **Integração Dinâmica via API:** Integração com Mercado Pago e Asaas. Os tokens, chaves e *hashes* de configuração das APIs NÃO podem ficar fixos no código (*hardcoded*). Devem ser guardados no banco de dados em uma tabela de configurações ou variáveis de ambiente gerenciáveis por painel administrativo.
  * **Troca de Conta sem Downtime:** Deve ser possível alterar/remover/adicionar novas contas de recebimento instantaneamente pelo painel administrativo, sem quebrar o sistema em produção ou deslogar usuários ativos.
  * **Baixa Manual:** O sistema precisa de uma funcionalidade administrativa para que um superusuário dê baixa manual em um pagamento para liberar o acesso imediatamente em casos de contingência.

### 1.4 Distribuição Mobile (PWA)
* **Objetivo:** Aplicação construída em Django Web, mas 100% instalável em Android e iOS, operando online.
* **Diretriz Técnica:**
  * Configurar a aplicação como um **PWA (Progressive Web App)** completo, incluindo `manifest.json`, *Service Workers* para cache de assets visuais, e ícones responsivos.
  * **Interface de Instalação:** Na tela inicial, exibir um botão/modal nativo, bonito, limpo e altamente intuitivo com instruções específicas de instalação baseadas no dispositivo detectado (ex: "Clique em Compartilhar e Adicionar à Tela de Início" no Safari/iOS; ou botão direto de instalação no Chrome/Android).

---

## 2. Regras Globais de Código (Rules)

* **Origem da Migração:** O código antigo é baseado em Web2py. Atenção ao traduzir a lógica do DAL do Web2py para o ORM do Django.
* **Padrões do Django:** Seguir estritamente o padrão PEP 8 para Python. Utilizar Class-Based Views (CBVs) assíncronas sempre que aplicável.
* **Segurança:** Bloquear por padrão qualquer requisição que não passe pelo filtro de validação de `empresa_id` (Multitenant).

---

## 3. Definição dos Agentes do Cursor

### Agente 1: O Arquiteto de Dados e Migração
* **Função:** Ler pedaços de códigos legados em Web2py fornecidos pelo usuário, interpretar as tabelas, relacionamentos e lógicas de negócio, e mapear para a estrutura moderna do Django.
* **Foco:** Garantir que o campo `empresa_id` e a lógica assíncrona sejam planejados antes da escrita do código.

### Agente 2: O Engenheiro de Software Django (Async & PWA)
* **Função:** Escrever o código real em Django (Models, Views, Middlewares, Templates, Manifest.json).
* **Foco:** Aplicar rigorosamente a blindagem de dados, a configuração dinâmica dos gateways de pagamento (Asaas/Mercado Pago) e a interface bonita do PWA com o botão de instalação.
