# SIO-EAP — Registro de Sessão de Desenvolvimento
**Data:** 23 de março de 2026 | **Horário:** 13h–15h (Fortaleza, CE — UTC-3)
**Responsável:** Vladimir Ferreira Silva
**Assistente:** Comet (Perplexity AI)
**Ferramenta de geração de código:** Gemini Pro (Agente de Modelagem e Orçamentação)

---

## Contexto do Projeto

O **SIO-EAP (Sistema Integrado de Orçamentação — Estrutura Analítica de Projeto)** é um sistema web desenvolvido para gestão de orçamentos e planejamento de obras da construção civil. O sistema é multiobra e multiusuário, com backend no Firebase (Firestore + Auth) e frontend em HTML/React single-file hospedado no GitHub Pages.

- **Repositório GitHub:** https://github.com/vladimirfersengkratus/eap-sistema-obras
- **Site ao vivo (GitHub Pages):** https://vladimirfersengkratus.github.io/eap-sistema-obras/
- **Projeto Firebase:** eap-sistema-obras
- **Gemini Gem (agente):** https://gemini.google.com/gem/4fc08162bab4

---

## Arquivos de Referência Utilizados na Sessão

| Arquivo | Conteúdo |
|---|---|
| `PLANEJAMENTO_CRF_R03.xlsm` / `.pdf` | Planejamento em Linha de Balanço da Casa RF — equipes, ritmos e fases |
| `ORC_RF_2025-R00.xlsx` | Orçamento da Obra RF com EAP completa |
| `EAP_AC.xlsx` | Estrutura Analítica de Projeto base da AC Incorporações |
| `Qto_ac_v1.0.xlsx` | Memórias de cálculo e quantitativos |

---

## Evolução do Sistema nesta Sessão

### 1. Módulo de Cronograma Gantt + Linha de Balanço (LOB)
Implementado com base no arquivo `PLANEJAMENTO_CRF_R03.xlsm`.

**Gantt:**
- Cabeçalho de 3 linhas: Meses (com borda espessa ao final) / Iniciais dos dias (Segundas em vermelho, feriados em preto) / Datas rotacionadas a 90°
- Por padrão exibe apenas dias úteis (seg–sex)
- Toggle "Incluir fins de semana" — quando ativo, sáb/dom aparecem em cinza
- Barras coloridas por pacote de trabalho
- Edição manual da semana de início

**Linha de Balanço (LOB):**
- Gráfico vetorial SVG com eixo Y = pavimentos/zonas, eixo X = semanas
- Ritmo calculado pela fórmula: `Qtd / (Produtividade × Nº de Equipes)`
- Slider para ajuste do número de equipes com recálculo automático
- Detector de conflitos de ritmo: círculo vermelho piscante no cruzamento de linhas
- Integração com quantitativos da EAP

### 2. Banco de Feriados
- Feriados nacionais fixos do Brasil
- Feriados municipais de Fortaleza-CE (Carnaval, N.S. Assunção, etc.)
- Campo para feriados customizados por obra
- Feriados bloqueiam dias automaticamente no Gantt

### 3. Cadastro de Fases da Obra (editáveis por obra)
Tela: `Engenharia > Cronograma > Fases da Obra`
- Campos: Nome, Ordem, Cor
- CRUD completo (criar, editar, reordenar, excluir)
- Fases padrão: ADMINISTRAÇÃO DA OBRA, IMPLANTAÇÃO, INFRAESTRUTURA E FUNDAÇÕES, SUPERESTRUTURA, VEDAÇÕES E EMBUTIDOS, REVESTIMENTOS, ACABAMENTOS, SERVIÇOS EXTERNOS

### 4. Cadastro de Pacotes de Trabalho
Tela: `Engenharia > Cronograma > Pacotes de Trabalho`
- Campos: Nome, Cor (color picker), Zona/Pavimento
- Vinculação de itens da EAP com rateio de % por fase
  - Ex: Tubo PVC Esgoto → 30% no pacote Infraestrutura / 70% no pacote Embutidos
- Quantitativos não alocados ficam no "banco de pendências"
- Conexão direta entre EAP de orçamento e cronograma

---

## Lógica do Planejamento em Linha de Balanço (Obra Casa RF)

**Zonas da obra (eixo Y):**
Fundação → Térreo → Pavimento Superior → Cobertura → Área Externa

**Pacotes identificados no arquivo:**

| Pacote | Serviços principais |
|---|---|
| Infraestrutura e Fundações | Escavação, Blocos/Sapatas, Tubulações enterradas (30% hidráulica/esgoto/eletrodutos) |
| Superestrutura | Pilares, Vigas, Lajes (concreto + fôrma + armação CA-50/CA-60) |
| Vedações e Embutidos | Alvenaria, Rasgos, Prumadas hidráulicas, Eletrodutos corrugados |
| Revestimentos | Chapisco, Reboco, Contrapiso, Impermeabilização |
| Acabamentos | Porcelanato, Forro de Gesso, Pintura, Esquadrias, Louças, Fiação |

**Tratamento de serviços em mais de um pacote:**
- Rateio (Split) de quantitativos por % na tela de montagem do pacote
- Na LOB: duas linhas distintas para o mesmo item orçamentário em momentos diferentes do eixo X

---

## Estado da Infraestrutura (verificado em 23/03/2026)

### GitHub
| Item | Status |
|---|---|
| Repositório | Público, 5 commits |
| Arquivo `index.html` | Atualizado (versão SIO-EAP Cronograma) |
| GitHub Pages | Ativo — https://vladimirfersengkratus.github.io/eap-sistema-obras/ |
| Domínio autorizado no Firebase Auth | Sim (`vladimirfersengkratus.github.io`) |

### Firebase (`eap-sistema-obras`)
| Serviço | Status |
|---|---|
| Authentication — Google | Ativo |
| Authentication — Anônimo | Ativo |
| Domínios autorizados | localhost, firebaseapp.com, web.app, github.io |
| Firestore — dados | Ativo — estrutura `users/{userId}/empresas` e `obras` |
| Firestore — Regras | **CORRIGIDAS nesta sessão (14:45)** — regra baseada em `request.auth.uid` sem expiração |
| Firebase Hosting | Sem deploy (requer Firebase CLI) — não necessário no momento |
| Plano | Spark (gratuito) |

**Regras Firestore atuais (publicadas 23/03/2026 14:45):**
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/{document=**} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

---

## Estrutura de Módulos do Sistema (versão atual)

```
SIO-EAP
├── Login (Firebase Auth — Google)
├── Seleção de Empresa / Obra
├── Engenharia
│   ├── Orçamento EAP (menu lateral com grupos)
│   │   ├── Lançamento de serviços
│   │   ├── Memória de cálculo automática (C × L × H × Repetições)
│   │   └── Custo unitário editável / % do total
│   ├── Cronograma
│   │   ├── Gráfico Gantt (cabeçalho 3 linhas, toggle fins de semana)
│   │   ├── Linha de Balanço LOB (SVG, detector de conflitos)
│   │   ├── Fases da Obra (CRUD editável por obra)
│   │   ├── Pacotes de Trabalho (rateio da EAP)
│   │   └── Feriados (nacionais + Fortaleza + customizados)
│   ├── Medições (a implementar)
│   └── Relatórios
│       └── Orçamento Completo (PDF / Excel)
├── Cadastros
│   ├── Fornecedores (a implementar)
│   ├── Catálogo EAP (a implementar)
│   ├── Usuários (a implementar)
│   └── Mão de Obra e Equipes (a implementar)
└── Admin
```

---

## Catálogo EAP — Grupos implementados

| Grupo | Nome |
|---|---|
| 01 | Administração da Obra |
| 02 | Implantação / Canteiro |
| 03 | Movimento de Terra |
| 04 | Fundações |
| 05 | Superestrutura (Concreto/Fôrma/Armação) |
| 06 | Alvenaria |
| 07 | Cobertura |
| 08 | Impermeabilização |
| 09 | Revestimento Interno |
| 10 | Revestimento Externo |
| 11 | Pisos e Contrapiso |
| 12 | Forro |
| 13 | Pintura |
| 14 | Esquadrias |
| 15 | Louças e Metais |
| 16 | Instalações Hidráulicas (AF/AQ — PVC Soldável, CPVC, PPR, PEX) |
| 17 | Instalações de Esgoto (PVC NBR 5688, série normal e reforçada) |
| 18 | Instalações Elétricas (eletrodutos, caixas, cabos) |
| 19 | Transportes e Elevadores |
| 20 | Ar Condicionado (BTUs por ambiente) |
| 21 | Instalação de Gás GLP |
| 22 | Reservatórios e Equipamentos de Lazer |
| 23 | Paisagismo e Área Externa |

---

## Pendências e Próximos Passos

### Módulos a implementar:
- [ ] **Cadastro de Equipes** (`Cadastros > Mão de Obra e Equipes`)
  - Composição: oficial, meio-oficial, servente + valor horário
  - Vinculação a pacote/serviço
  - Produtividade diária/semanal
  - Slider de nº de equipes integrado ao LOB
- [ ] **Módulo de Medições** (avanço físico-financeiro)
- [ ] **Módulo de Fornecedores**
- [ ] **Controle de Usuários por Empresa** (convite por e-mail)
- [ ] **Importação em lote de planilha** para a EAP
- [ ] **Cronograma Gantt com caminho crítico**
- [ ] **Domínio próprio** (ex: sio-eap.com.br) → necessário Firebase Hosting

### Quando migrar para Firebase Hosting:
- Ao adicionar Firebase Functions (e-mails, PDFs no servidor, APIs externas)
- Ao usar domínio próprio profissional
- Ao tornar o repositório GitHub privado
- Ao lançar o sistema como SaaS com cobrança

---

## Histórico de Commits desta Sessão

| Commit | Descrição |
|---|---|
| #3 | feat: SIO-EAP v2 - menu modular, memória de cálculo funcional, relatórios |
| #4 | Adicionar arquivos por meio de upload (versão com Cronograma Gantt + LOB) |
| #5 | Adicionar arquivos por meio de upload (versão SIO-EAP Cronograma — versão atual) |

---

## Notas Técnicas

- O sistema é um **single-file HTML** (`index.html`) com React (via CDN), Tailwind CSS e Firebase Compat v9
- Toda lógica roda no browser — sem servidor próprio
- Persistência no Firestore com estrutura: `users/{userId}/empresas/{id}` e `users/{userId}/obras/{id}`
- Autenticação: Google OAuth + fallback Anônimo
- O sistema carrega via GitHub Pages; o domínio `vladimirfersengkratus.github.io` está na lista de domínios autorizados do Firebase Auth

---

*Registro gerado automaticamente pelo assistente Comet (Perplexity AI) em 23/03/2026 às 15h.*
