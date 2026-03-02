# Sistema de Controle de Processos — Gabinete Judicial

Aplicação web **standalone** para gerenciamento de processos judiciais em gabinete. Totalmente client-side, sem backend ou instalação — basta abrir o arquivo no navegador.

---

## Como executar

```
Abrir o arquivo controle-processos-completo.html em qualquer navegador moderno.
```

Nenhum servidor, build ou instalação são necessários. Todos os dados ficam salvos no `localStorage` do navegador.

---

## Stack tecnológico

| Tecnologia | Uso |
|---|---|
| HTML5 + CSS3 + JavaScript ES6 | Toda a aplicação |
| jsPDF 2.5.1 | Geração de relatórios PDF |
| jsPDF AutoTable 3.5.31 | Tabelas nos PDFs |
| SheetJS (XLSX) 0.18.5 | Importação de planilhas Excel |
| Google Fonts | Tipografia (Montserrat + Open Sans) |

Todas as dependências externas são carregadas via CDN.

---

## Estrutura do arquivo

```
controle-processos-completo.html  (~13.300 linhas)
├── <style>          CSS completo (~3.500 linhas)
├── <body>
│   ├── Sidebar      Navegação lateral recolhível
│   ├── Painel       Alertas e gráfico de evolução
│   ├── Tabelas      Em Andamento · Despachados · Finalizados
│   └── 21 Modais    Cadastro, edição, relatórios, etc.
└── <script>         Classe ProcessoManager (~6.000 linhas)
```

---

## Funcionalidades

### Gerenciamento de processos

- **Cadastro individual** de processos com número, assunto, classe processual, data de conclusão, prioridade, status e observações
- **Cadastro em lote** via texto ou importação de planilha Excel (.xlsx / .xls)
- **Edição** de todos os campos de um processo existente
- **Rascunho / Minuta** — editor de texto livre por processo, acessível clicando no número do processo; processos com minuta exibem badge verde distintivo
- **Exclusão** individual com confirmação
- **Cópia** do número do processo para a área de transferência

### Fluxo de status

```
Cadastro
    ↓
Em Andamento  (aguardando-despacho  /  concluso)
    ↓
Despachado    ← data de despacho registrada
    ↓
Finalizado    ← data de finalização registrada
    ↑
Reabrir       (qualquer status pode retornar ao andamento)
```

### Prioridades automáticas

As prioridades são recalculadas automaticamente com base nos dias decorridos desde a data de conclusão:

| Prioridade | Condição |
|---|---|
| Normal | Menos de 70 dias |
| Urgente | 70 a 88 dias |
| Muito Urgente (Crítico) | 89 dias ou mais |
| Prioridade Advogado | Manual — nunca é alterada automaticamente |

Os thresholds são configuráveis em **Configurações**.

### Alertas e monitoramento

- **Painel de alertas** com contadores em tempo real de processos críticos, em atenção e finalizados no mês
- **Sidebar** exibe badges com os contadores atualizados
- **Modal de alertas** lista processos filtrados por categoria (críticos, atenção, próximos do prazo)
- **Gráfico de evolução** dos processos críticos nos últimos 30 dias, com tooltip interativo
- **Modal de gráfico expandido** com filtros de período: 30 dias, 60 dias, 90 dias, 6 meses, 1 ano ou todo o período

### Relatórios e análises

| Relatório | Descrição |
|---|---|
| **Relatório PDF** | Processos finalizados em um período, gerado com jsPDF + AutoTable |
| **Produção Mensal** | Cards com estatísticas de despachados e finalizados por mês/ano |
| **Histórico do Processo** | Timeline completo de todas as alterações, exportável em PDF |
| **Planejamento de Prazos** | Calendário interativo (heatmap) com processos críticos, feriados e férias |

### Calendário e planejamento

- Calendário mensal com processos críticos distribuídos por dia
- Destaque visual para feriados, férias e fins de semana
- Drag-and-drop para reorganizar a visualização
- Cadastro de feriados e períodos de férias

### Busca, filtros e ordenação

- **Busca** por número de processo ou assunto (com debounce de 300ms)
- **Ordenação** por qualquer coluna, crescente ou decrescente, com preferência salva por tabela
- **Filtro por mês/ano** na tabela de Finalizados
- **Filtros visuais**: badges coloridos de status e prioridade; indicador de minuta

### Exportação e importação de dados

- **Exportar JSON** — salva todos os dados em arquivo para backup
- **Importar JSON** — restaura dados a partir de um arquivo exportado anteriormente
- **Importar Excel** — importação de processos em lote a partir de planilha .xlsx/.xls
- **Limpar dados** — apaga tudo do localStorage após confirmação

### Configurações

- Threshold de dias críticos (padrão: 89 dias) ajustável pelo usuário
- Classes processuais customizáveis (CRUD completo)
- Assuntos customizáveis (CRUD completo)
- Tema escuro / claro com toggle manual e respeito à preferência do sistema

---

## Objeto de processo

```javascript
{
  id:               Number,   // timestamp + decimal aleatório (único)
  numeroProcesso:   String,   // ex: "0000000-00.0000.8.20.0000"
  assunto:          String,
  classeProcessual: String,   // "apelacao-civel" | "agravo-instrumento" | ...
  dataConclusao:    String,   // "YYYY-MM-DD"
  prioridade:       String,   // "normal" | "urgente" | "muito-urgente" | "prioridade-advogado"
  status:           String,   // "aguardando-despacho" | "concluso" | "despachado" | "finalizado"
  observacoes:      String,
  minuta:           String,   // rascunho/minuta do processo
  dataCadastro:     String,   // ISO timestamp
  dataDespacho:     String,   // "YYYY-MM-DD"
  dataFinalizacao:  String,   // "YYYY-MM-DD"
  dataModificacao:  String,   // ISO timestamp
  historico:        Array     // log de alterações
}
```

---

## Chaves do localStorage

| Chave | Conteúdo |
|---|---|
| `processos-gabinete` | Array de todos os processos |
| `ordenacao-preferencias` | Preferência de ordenação por tabela |
| `sidebar-preferencias` | Estado do sidebar (colapsado, seção ativa) |
| `finalizados-preferencias` | Mês/ano selecionado na aba Finalizados |
| `configuracoes-sistema` | Thresholds e configurações gerais |
| `classes-processuais` | Classes processuais customizadas |
| `assuntos` | Assuntos customizados |
| `historico-processos` | Log de alterações dos processos |
| `feriados` | Feriados cadastrados |
| `ferias` | Períodos de férias cadastrados |
| `tema-preferencia` | Tema escuro ou claro |
| `calendario-posicoes-customizadas` | Posições customizadas no calendário |

---

## Arquitetura

Toda a lógica está encapsulada na classe `ProcessoManager`:

```
localStorage
     │
     ▼
carregarProcessos()
     │
     ▼
renderizarProcessos()
├── atualizarPrioridadesAutomaticas()
├── atualizarAlertas()
└── renderizarTabela[Andamento | Despachados | Finalizados]()
     │
     ▼
Interação do usuário
     │
     ▼
salvarProcessos() ──► localStorage
```

### Métodos principais

| Método | Descrição |
|---|---|
| `inicializar()` | Ponto de entrada — configura eventos e renderiza |
| `renderizarProcessos()` | Separa processos por status e renderiza as tabelas |
| `atualizarPrioridadesAutomaticas()` | Recalcula prioridades por dias decorridos |
| `atualizarAlertas()` | Atualiza contadores e badges de alerta |
| `cadastrarProcesso()` | Adiciona novo processo |
| `salvarEdicao()` | Persiste edição de processo |
| `mudarStatusRapido()` | Transições de status com confirmação |
| `abrirMinuta()` / `salvarMinuta()` | Gerencia rascunho/minuta do processo |
| `gerarRelatorioPDF()` | Gera PDF dos processos finalizados |
| `exportarDados()` / `importarDados()` | Backup e restore em JSON |
| `encontrarProcessoPorId()` | Busca robusta de ID (trata string e number) |
| `gerarIdUnico()` | Cria IDs via timestamp + random + sequência |

---

## Debugging

1. Abrir **DevTools** (F12) no navegador
2. **Console** → erros JavaScript
3. **Application > Local Storage** → inspecionar dados (chave: `processos-gabinete`)
4. **Ctrl + F5** para hard refresh (limpar cache)

---

## Observações importantes

- **Auto-prioridade**: recalculada a cada renderização, exceto `prioridade-advogado`
- **IDs únicos**: gerados com timestamp + decimal aleatório para evitar colisões entre sessões
- **Migrações**: o sistema inclui funções de migração automática para atualizar dados de versões anteriores
- **Performance**: com mais de 500 processos pode haver lentidão na renderização; considerar paginação se necessário
- **Modais**: fecham em ordem inversa de abertura; a tecla ESC fecha o modal mais recente
- **Offline**: funciona completamente sem conexão com a internet
