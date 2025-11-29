# 💸 AppChat de Finanças Pessoais com Vibe Coding

PRD Refinado no M365 Copilot

```
APP: Finanças Conversacionais (MVP)
Autor: Matheus
Versão: 0.1 (MVP)

============================================================
1) VISÃO GERAL
============================================================
Contexto:
App de finanças pessoais controlado por conversas em linguagem natural, com registro simples de gastos, categorização automática, resumo de saldo e dicas de economia.

Problema:
Apps comuns exigem entradas complexas (formulários, telas rígidas), pouca personalização, gerando desistência.

Solução:
Experiência conversacional + IA para entender frases como “Gastei R$50 no mercado” e sugerir ações.

Objetivo do Produto:
Facilitar o registro de gastos e acompanhamento financeiro via chat, com recomendações básicas.

Público-Alvo:
Iniciantes em organização financeira que querem praticidade e linguagem acessível.

Diferencial:
Conversas naturais + recomendações simples, focado em quem está começando.

============================================================
2) FUNCIONALIDADES-CHAVE (MVP)
============================================================
1. Registro de gastos via chat (PT-BR):
   - Ex.: “Gastei R$50 no mercado”, “Paguei R$120 de energia”.

2. Classificação automática das transações:
   - “Mercado” → Alimentação; “Uber” → Transporte; “Energia” → Moradia; etc.

3. Visualização simples do saldo e categorias (no chat):
   - Resumo semanal/mensal; “Como está meu gasto esta semana?”

4. Sugestões básicas de economia:
   - Ex.: “Você gastou 35% acima da média em Alimentação; que tal definir um limite de R$300/semana?”

(Futuro: metas financeiras, relatórios avançados, agente financeiro com dicas personalizadas.)

============================================================
3) FLUXO DO USUÁRIO (MVP)
============================================================
Tela inicial → Chat → Usuário digita gasto → App responde com:
   (a) categoria detectada,
   (b) saldo atualizado/resumo rápido,
   (c) sugestão simples de economia (se aplicável).

============================================================
4) DIRETRIZES DE DESIGN
============================================================
Estilo: Minimalista, limpo, universal.
Paleta: Branco/cinza claro + azul ou verde para ações positivas.
Tipografia: Inter, Roboto ou SF Pro.
Componentes: Chat com balões arredondados, ícones Material Icons.
Acessibilidade: Contraste adequado, botões grandes, foco visível, feedback textual.
Tom visual: Educativo e amigável.

============================================================
5) MÉTRICAS DE SUCESSO
============================================================
- % de usuários que registram ≥5 gastos na primeira semana.
- Retenção em 30 dias.
- Engajamento no chat (interações/semana por usuário).
- (Opcional) % de mensagens corretamente classificadas pelo NLP.

============================================================
6) PLANO DE VALIDAÇÃO
============================================================
Hipótese: Usuários preferem registrar gastos via chat do que via formulário.
Teste: Landing page + protótipo de chat → medir conversão, tempo até 1º registro, e feedback qualitativo (NPS curto e entrevistas).

============================================================
7) REQUISITOS TÉCNICOS
============================================================
NLP: Azure Language Services ou OpenAI API (PT-BR, intents, entity extraction).
Backend: Node.js (Express) ou Python (FastAPI).
Banco de Dados: PostgreSQL (prod) ou SQLite (dev).
Frontend: Web (React/Vite) ou mobile híbrido (React Native/Expo).
Autenticação: Email + senha (MVP) ou OAuth.
Infra: Docker (dev), vercel/railway/render para deploy rápido.
Observabilidade: Logs estruturados, métricas, armazenamento de eventos de chat.

============================================================
8) ARQUITETURA (MVP)
============================================================
Camadas:
- UI (Chat) → API Gateway → Serviço NLP → Serviço de Transações → DB
- Serviço de Sugestões (heurísticas simples, executa pós-transação)
- Serviço de Resumo (consulta agregada semanal/mensal)

ASCII (alto nível):
[Web/Mobile Chat]
      |
   HTTPS
      |
[API Backend] --(NLP)--> [NLP Provider]
      |                     ^
      v                     |
   [DB - Transações, Categorias, Usuários]
      |
   [Sugestões/Resumo]

============================================================
9) MODELOS DE DADOS (SQL)
============================================================
Tabela users:
- id (pk, uuid)
- name (text)
- email (text, unique)
- created_at (timestamp)

Tabela categories:
- id (pk)
- name (text, unique)  # Ex.: Alimentação, Transporte, Moradia, Lazer, Saúde, Educação, Outros
- keywords (text[])    # ex.: ["mercado","supermercado","carrefour","pão de açúcar"]

Tabela transactions:
- id (pk, uuid)
- user_id (fk users.id)
- amount (numeric(12,2))    # positivo para despesa; (MVP: só despesas)
- currency (text)           # "BRL"
- description (text)
- category_id (fk categories.id, null se indefinida)
- occurred_at (date/time)
- created_at (timestamp)
- source (text)             # "chat"
- nlp_confidence (numeric)  # 0-1

Tabela budgets (futuro):
- id (pk)
- user_id
- category_id
- period (text)             # "weekly"|"monthly"
- limit_amount (numeric)

============================================================
10) ENDPOINTS (MVP)
============================================================
POST /chat/message
Body: { "userId": "...", "message": "Gastei R$50 no mercado" }
Resp: {
  "parsed": { "amount": 50.00, "currency": "BRL", "category": "Alimentação", "description": "mercado" },
  "transaction": { ... },
  "summary": { "week_spent": 320.75, "top_categories": [{name:"Alimentação",spent:180.20}] },
  "suggestion": "Defina um limite semanal para Alimentação."
}

POST /transactions
Body: { "userId":"...", "amount": 50.00, "currency":"BRL", "description":"mercado", "occurredAt":"2025-11-29T14:05:00-03:00" }

GET /summary?userId=...&period=week
Resp: { "periodStart":"2025-11-24", "periodEnd":"2025-11-30", "totalSpent": 540.00, "byCategory":[...] }

GET /categories
Resp: [ {id:1,name:"Alimentação"}, ... ]

============================================================
11) NLP (INTENTS E ENTIDADES)
============================================================
Intents:
- add_expense: “gastei R$50 no mercado”, “paguei 120 de luz”, “comprei uber de 25”
- ask_summary: “como está meu gasto?”, “resumo da semana”
- ask_balance (MVP simples): “quanto gastei hoje?”, “saldo do mês”
- misc_help: “como funciona?”, “o que posso perguntar?”

Entidades:
- amount: “R$50”, “50 reais”, “50,00”
- category candidate: “mercado”, “uber”, “energia”
- date/time: “hoje”, “ontem”, “terça”, “29/11”
- merchant (futuro): nomes específicos (ex.: “Carrefour”, “Uber”)

Regras de parsing (MVP):
- Normalizar moeda BRL: aceitar “R$”, “reais”, separador decimal “,”.
- Regex exemplos:
  - Valor: r"(?:R\$|\b)\s*(\d{1,3}(?:\.\d{3})*(?:,\d{2})|\d+(?:,\d{2})?)"
  - Data opcional: r"\b(hoje|ontem|amanhã|segunda|terça|quarta|quinta|sexta|sábado|domingo|\d{1,2}/\d{1,2}(?:/\d{2,4})?)\b"
- Normalização: converter "50,00" → 50.00 (ponto como decimal internamente).

Mapeamento de categorias (heurístico inicial):
- Alimentação: ["mercado","supermercado","carrefour","pão de açúcar","ifood","restaurante"]
- Transporte: ["uber","99","ônibus","metrô","gasolina","ipiranga"]
- Moradia: ["aluguel","condomínio","energia","luz","água","internet","vivo","claro"]
- Lazer: ["cinema","spotify","netflix","show","bar"]
- Saúde: ["farmácia","consulta","plano","academia"]
- Educação: ["curso","livro","faculdade","ufabc"]
- Outros: fallback quando não há match.

============================================================
12) LÓGICA DE SUGESTÕES (MVP)
============================================================
Regras simples:
- Se gasto semanal em uma categoria > média das últimas 4 semanas + 20% → sugerir limite semanal.
- Se “Outros” > 25% do total → sugerir detalhar descrições para melhorar categorização.
- Se gasto recorrente semelhante todo mês (ex.: “energia”, “internet”) → sugerir marcar como recorrente e acompanhar.
Mensagens:
- Tom amigável, objetivo, com ação clara (definir limite, revisar gastos, ajustar categoria).

============================================================
13) UI/UX (CHAT)
============================================================
- Balões arredondados; cores neutras; verde/azul para confirmações.
- Mensagens estruturadas: valor + categoria + data + resumo.
- Acessibilidade: contraste AA, teclas de atalho, labels claros, aria-live para mensagens.

============================================================
14) EXEMPLOS DE CONVERSA
============================================================
User: "Gastei R$50 no mercado"
App: "Registrado: R$50 em Alimentação (mercado) hoje. Total da semana: R$180. Dica: quer definir limite semanal para Alimentação?"

User: "Paguei R$120 de energia ontem"
App: "Registrado: R$120 em Moradia (energia) ontem. Semana: R$300. Você costuma pagar energia todo mês; quer marcar como recorrente?"

User: "Como está meu gasto esta semana?"
App: "Resumo (24–30/11): Total R$540. Top categorias: Alimentação R$220, Moradia R$180, Transporte R$90. Sugestão: Alimentação +35% vs. média; definir limite?"

============================================================
15) PSEUDOCÓDIGO (PROCESSAMENTO DE MENSAGEM)
============================================================
function handleChatMessage(userId, message):
  parsed = nlp.parse(message)   # amount, date, keywords
  amount = normalizeBRL(parsed.amount)
  date = resolveDate(parsed.date) or now()
  description = extractDescription(parsed.keywords)
  category = classify(description)  # regras + keywords
  confidence = score(category, description)

  tx = saveTransaction(userId, amount, "BRL", description, category, date, confidence)
  summary = getWeeklySummary(userId, date.week)
  suggestion = suggest(summary, category)

  return { parsed, transaction: tx, summary, suggestion }

============================================================
16) ESTRUTURA DE PASTAS (REPO)
============================================================
/app
  /frontend            # React/React Native (chat UI)
  /backend             # API (Node/Express ou Python/FastAPI)
    /src
      controllers/
      services/
      nlp/
      db/
      routes/
      tests/
    .env.example
  /infra               # Docker, compose, CI
/docs                  # PRD, fluxos, decisões
README.md
LICENSE

============================================================
17) INSTRUÇÕES DE EXECUÇÃO (DEV, EXEMPLO FASTAPI)
============================================================
- Requisitos: Python 3.11, pip, Docker (opcional).
- Passos:
  1) cp backend/.env.example backend/.env  # configure DB_URL e keys de NLP
  2) make dev (opcional) ou:
     - pip install -r backend/requirements.txt
     - uvicorn src.main:app --reload
  3) Frontend:
     - npm i && npm run dev
- Testes:
  - pytest (backend) / vitest (frontend)

============================================================
18) PRIVACIDADE E SEGURANÇA
============================================================
- Criptografar senhas (bcrypt).
- Armazenar apenas o necessário (minimização de dados).
- Logs sem PII sensível (mas com IDs).
- LGPD: consentimento para processamento de dados, deleção sob solicitação.
- Backups e controle de acesso por usuário.

============================================================
19) LOCALIZAÇÃO (PT-BR)
============================================================
- Formatação monetária: "R$ 1.234,56".
- Datas: DD/MM/YYYY.
- Mensagens em português simples; evitar jargões.

============================================================
20) ROADMAP FUTURO
============================================================
- Metas financeiras por categoria (semana/mês).
- Relatórios avançados (tendências, comparativos).
- Agente financeiro proativo (alertas, recomendações contextuais).
- Importação de extratos (OFX/CSV) e reconciliação.
- Análise de recorrência e assinaturas.
- Multi-conta (cartão, conta corrente, carteira).
- Notificações (push/email) com resumo.

============================================================
21) MÉTRICAS (IMPLEMENTAÇÃO)
============================================================
- event: transaction_created (userId, amount, category, timestamp)
- event: summary_viewed
- event: suggestion_shown / suggestion_accepted
- Dashboards: conversões 1º dia, 1ª semana, retenção D30.

============================================================
22) VALIDACÃO (EXPERIMENTO)
============================================================
- Landing page com valor e demo do chat.
- Métricas: CTR para experimentar, taxa de 1º registro, 5 registros/semana, NPS.
- Entrevistas com 8–12 usuários iniciantes (qualitativo).
- Iterar mensagens e heurísticas de sugestão.

============================================================
23) NOTAS FINAIS
============================================================
- Foco em simplicidade, rapidez de registro, e linguagem acessível.
- Sempre responder com confirmação, resumo e possível ação (definir limite, revisar categoria).
- Medir, aprender e iterar com feedback contínuo.
```
Resultado final com o Lovable: https://coin-convos.lovable.app

<img width="1875" height="930" alt="image" src="https://github.com/user-attachments/assets/eacc3458-c963-4602-aa3c-37bb7dd006f1" />

<img width="1868" height="922" alt="image" src="https://github.com/user-attachments/assets/4994c42b-1c54-416f-b3f9-8c212bddeac5" />

<img width="1870" height="922" alt="image" src="https://github.com/user-attachments/assets/7f60f764-9dd3-4109-bd54-f5e85e0ac915" />


Resumo:

Um app de finanças pessoais que transforma o controle financeiro em uma experiência conversacional. Em vez de formulários e planilhas, você registra seus gastos falando com o app, que classifica automaticamente as transações, mostra resumos claros e sugere formas simples de economizar. Ideal para quem quer começar a organizar suas finanças sem complicação.

✅ Funcionalidades
1. Registro de gastos via chat
   Ex.: “Gastei R$50 no mercado”.
     
2. Classificação automática
    Identifica a categoria (Alimentação, Transporte, Moradia, etc.).
      
3. Resumo financeiro no chat
    Mostra total gasto e categorias (ex.: resumo semanal).
      
4. Sugestões simples de economia
    Ex.: “Você gastou muito em alimentação esta semana, que tal definir um limite?”.

### O que funcionou bem?
  A classificação automática das transações por categoria torna o processo rápido e sem esforço manual.
  
  O tom educativo e amigável ajuda a engajar usuários sem parecer técnico demais.
  
  O PRD está bem detalhado, com fluxo claro e métricas definidas, o que facilita transformar em produto real.

### O que não funcionou como esperado?
  Falta integração com fontes externas
  
  A dependência de NLP pode gerar erros em frases
  
  Erros de design que posteriormente foram corrigidos

### O que aprendeu sobre conversar com IAs?
  É essencial dar contexto claro e estruturado
  
  Quanto mais específico e detalhado o pedido, melhor a qualidade do resultado
  
  A IA é ótima para organizar ideias, gerar documentação e protótipos, mas precisa de validação humana para nuances e experiência do usuário.
  
  Conversar com IA é como trabalhar com um parceiro criativo


