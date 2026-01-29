# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Visão Geral

**Sistema de Controle de Processos - Gabinete Judicial**

Aplicação web standalone para gerenciamento de processos judiciais em um gabinete. Totalmente client-side, sem backend - todos os dados são armazenados no localStorage do navegador.

**Idioma**: Português (Brasil)

## Stack Tecnológico

- **HTML5, CSS3, JavaScript ES6** - Tudo em um único arquivo (`controle-processos-completo.html`)
- **jsPDF + AutoTable** (via CDN) - Geração de relatórios PDF
- **localStorage** - Persistência de dados
- **Google Fonts** - Montserrat + Open Sans

## Como Executar

```bash
# Sem build necessário - abrir diretamente no navegador
# Windows:
start controle-processos-completo.html

# Ou simplesmente abrir o arquivo em qualquer navegador moderno
```

## Debugging

- Abrir DevTools (F12)
- Console para erros JavaScript
- Application > Local Storage para inspecionar dados (chave: `processos-gabinete`)
- Ctrl+F5 para hard refresh (limpar cache)

## Arquitetura

### Estrutura do Arquivo Único

```
controle-processos-completo.html (~10.800 linhas)
├── <head>
│   └── <style> (CSS completo ~3500 linhas)
├── <body>
│   ├── <aside class="sidebar"> (navegação lateral recolhível)
│   ├── <div class="container">
│   │   ├── Painel de Alertas (#alerts-section)
│   │   ├── Tabela Em Andamento (.table-section-andamento)
│   │   ├── Tabela Despachados (.table-section-despachados)
│   │   └── Tabela Finalizados (.table-section-finalizados)
│   └── ~15 Modais (edição, relatório, produção, alertas, configurações, etc.)
└── <script> (JavaScript completo ~5900 linhas)
    └── class ProcessoManager { ... }
```

### Classe Principal: ProcessoManager

**Constantes de Prioridade:**
```javascript
static DIAS_ATENCAO = 70;   // Processos com 70+ dias = prioridade "urgente"
static DIAS_CRITICO = 89;   // Processos com 89+ dias = prioridade "muito-urgente"
```

**Métodos Principais:**

| Método | Descrição |
|--------|-----------|
| `inicializar()` | Ponto de entrada, configura eventos e renderiza |
| `renderizarProcessos()` | Separa processos por status e renderiza tabelas |
| `atualizarPrioridadesAutomaticas()` | Atualiza prioridades baseado nos dias decorridos |
| `atualizarAlertas()` | Atualiza contadores de alertas e badges |
| `cadastrarProcesso()` | Adiciona novo processo |
| `salvarEdicao()` | Salva edição de processo |
| `mudarStatusRapido()` | Altera status (despachar, finalizar, retornar) |
| `gerarRelatorioPDF()` | Gera PDF de processos finalizados |
| `exportarDados()` / `importarDados()` | Backup/restore JSON |
| `encontrarProcessoPorId()` | Busca robusta de ID (trata string/number) |
| `gerarIdUnico()` | Cria IDs únicos usando timestamp + random + sequência |

### Fluxo de Dados

```
localStorage (processos-gabinete)
       │
       ▼
ProcessoManager.carregarProcessos()
       │
       ▼
renderizarProcessos()
├─► atualizarPrioridadesAutomaticas()
├─► atualizarAlertas()
└─► Renderiza 3 tabelas
       │
       ▼
Interações do Usuário
       │
       ▼
salvarProcessos() ──► localStorage
```

### Fluxo de Status do Processo

```
Cadastro
   ↓
Em Andamento (aguardando-despacho ou concluso)
├── Prioridades auto-atualizadas por dias decorridos
└── Alertas rastreiam processos críticos/atenção
   ↓
Despachado (despachado)
└── Data registrada (dataDespacho)
   ↓
Finalizado (finalizado)
├── Data registrada (dataFinalizacao)
└── Pode ser exportado em relatório PDF
```

## Estrutura do Objeto Processo

```javascript
{
  id: Number,              // timestamp + random decimal (único)
  numeroProcesso: String,  // ex: "0000000-00.0000.0.00.0000"
  assunto: String,         // descrição do assunto
  classeProcessual: String, // "apelacao-civel", "agravo-instrumento", etc.
  dataConclusao: String,   // "YYYY-MM-DD"
  prioridade: String,      // "normal", "urgente", "muito-urgente", "prioridade-advogado"
  status: String,          // "aguardando-despacho", "concluso", "despachado", "finalizado"
  observacoes: String,     // notas livres
  dataCadastro: String,    // ISO timestamp
  dataDespacho: String,    // "YYYY-MM-DD" (quando despachado)
  dataFinalizacao: String, // "YYYY-MM-DD" (quando finalizado)
  dataModificacao: String, // ISO timestamp
  historico: Array         // Entradas do log de alterações
}
```

## Chaves do localStorage

| Chave | Conteúdo |
|-------|----------|
| `processos-gabinete` | Array JSON de todos os processos |
| `ordenacao-preferencias` | Preferências de ordenação por tabela |
| `sidebar-preferencias` | Estado do sidebar (colapsado, seção ativa) |
| `finalizados-preferencias` | Ano/mês selecionado na aba de finalizados |
| `configuracoes-sistema` | Classes processuais, assuntos, thresholds |
| `classes-processuais` | Lista de classes processuais customizadas |
| `assuntos` | Lista de assuntos customizados |
| `historico-processos` | Histórico de alterações nos processos |
| `feriados` | Lista de feriados cadastrados |
| `ferias` | Períodos de férias cadastrados |
| `calendario-posicoes-customizadas` | Posições customizadas no calendário |
| `migracao-ids-decimal` | Flag de migração executada |

## Tarefas Comuns

### Adicionar novo campo ao processo
1. Adicionar propriedade no objeto em `cadastrarProcesso()`
2. Adicionar input no modal de cadastro (HTML)
3. Atualizar `editarProcesso()` e `salvarEdicao()`
4. Atualizar renderização da tabela se necessário
5. Considerar migração para dados existentes

### Modificar thresholds de dias
Alterar constantes no início da classe `ProcessoManager`:
```javascript
static DIAS_ATENCAO = 70;  // >= 70 dias = Atenção (urgente)
static DIAS_CRITICO = 89;  // >= 89 dias = Crítico (muito-urgente)
```

### Modificar transições de status
Ajustar lógica em:
- `mudarStatusRapido()` - trata mudanças de status com confirmações
- Renderização de botões em `renderizarTabelaAndamento()`, etc.

### Modificar estilos
- CSS está no `<style>` dentro do `<head>`
- Variáveis CSS em `:root` (cores, sombras, etc.)

## Observações Importantes

1. **Auto-prioridade**: Prioridades são recalculadas automaticamente baseadas nos dias, EXCETO `prioridade-advogado` que nunca é alterada automaticamente

2. **IDs únicos**: Usam timestamp + decimal aleatório + sequência para evitar colisões entre sessões

3. **Migrações**: Existem funções de migração (`migrarIDsParaDecimal()`, `migrarIDsStringParaNumber()`) que rodam uma vez para corrigir dados antigos - cuidado ao modificar estrutura do objeto

4. **Performance**: Com >500 processos pode haver lag na renderização - considerar virtualização ou paginação se necessário

5. **Sistema de Modais**: Múltiplos modais com suporte a ESC - fecham na ordem inversa (último aberto primeiro)

6. **Sidebar**: Recolhível (280px expandido, 70px colapsado) - em mobile abre como overlay

7. **Busca com Debounce**: Funcionalidade de pesquisa usa debounce de 300ms para performance
