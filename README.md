# Meu Balanço 💰

PWA de planejamento financeiro pessoal. Leve, offline-first, sem cadastro.

Acesso: [krentz.github.io/meubalanco](https://krentz.github.io/meubalanco)

---

## O que é

Uma ferramenta simples pra quem quer controlar entradas e saídas mensais sem a complexidade de planilhas ou apps pesados. O usuário abre no celular, adiciona um lançamento em segundos e vê o saldo em tempo real.

A proposta é entregar valor rápido — MVP validável — com possibilidade de evoluir para um produto com backend, autenticação e assinatura.

---

## Decisões técnicas

### PWA em vez de app nativo
Optamos por web app instalável (PWA) em vez de Flutter/React Native por uma razão central: **velocidade de validação**. Um arquivo `.html` hospedado no GitHub Pages elimina App Store, review, SDK updates e complexidade de distribuição. Se o produto validar, o backend e o app nativo entram numa segunda fase com dados de mercado na mão.

### localStorage como banco de dados
Dados salvos no `localStorage` do browser, por mês (`mb_YYYY_M`). Recorrentes ficam em chave separada (`mb_recorrentes`). Funciona offline, zero custo, zero servidor. Limitação conhecida: dados ficam no dispositivo. Solução futura: Firebase/Supabase com autenticação.

### GitHub Pages para hospedagem
Gratuito, HTTPS nativo (requisito para PWA e Service Worker), deploy via push. Repo pode ser privado — o Pages continua público normalmente.

### Sem framework
HTML/CSS/JS puro. Sem dependências de build, sem node_modules, sem pipeline. O produto inteiro é um arquivo `index.html` + assets. Fácil de manter, fácil de evoluir.

### Bibliotecas externas (CDN)
- `SheetJS (xlsx)` — exportação para Excel no browser
- `Tabler Icons` — ícones outline consistentes
- `DM Sans + DM Mono` — tipografia (Google Fonts)

---

## Estrutura do projeto

```
meubalanco/
├── index.html          ← app completo
├── manifest.json       ← configuração PWA (nome, ícone, cor)
├── service-worker.js   ← cache offline
├── icon-192.png        ← ícone Android/PWA
├── icon-512.png        ← ícone splash screen
└── README.md
```

---

## Modelo de dados

Lançamentos salvos por mês em `localStorage`:

```json
{
  "periodo": "Junho 2026",
  "mes": 5,
  "ano": 2026,
  "lancamentos": [
    {
      "tipo": "entrada",
      "descricao": "Salário junho",
      "valor": 5000,
      "categoria": "Salário",
      "recorrente_id": "abc123"
    },
    {
      "tipo": "saida",
      "descricao": "",
      "valor": 1200,
      "categoria": "Casa / moradia",
      "recorrente_id": "xyz456"
    }
  ]
}
```

Recorrentes salvos separadamente em `mb_recorrentes`:

```json
[
  {
    "id": "abc123",
    "tipo": "entrada",
    "descricao": "Salário",
    "valor": 5000,
    "categoria": "Salário",
    "inicio_ano": 2026,
    "inicio_mes": 5
  }
]
```

Regras dos recorrentes:
- Propagam apenas para o mês atual e futuros (nunca retroativos)
- Editar atualiza todos os meses futuros e o template
- Apagar remove o template e todos os meses onde aparece

---

## Funcionalidades atuais

- Adicionar entradas e saídas com descrição, valor e categoria
- Descrição opcional — usa a categoria como fallback
- Toggle de lançamento recorrente (propaga automaticamente mês a mês)
- Editar e apagar lançamentos (com propagação para recorrentes)
- Resumo: total entradas, total saídas, saldo do mês
- Barra de renda comprometida com alerta visual (verde → âmbar → vermelho)
- Breakdown de saídas por categoria
- Navegação entre meses
- Exportar dados: JSON (backup), Excel (.xlsx), PDF (via print nativo)
- Importar backup JSON (com migração automática de versões antigas)
- Instalável como PWA no iOS e Android
- 100% offline após primeira carga

---

## Evolução planejada

### Curto prazo (próxima sessão)
- [ ] Tela de visão anual — comparar meses lado a lado
- [ ] Meta de poupança configurável pelo usuário
- [ ] Semáforo visual por categoria (verde/amarelo/vermelho)
- [ ] Categorias customizáveis pelo usuário
- [ ] Confirmação antes de apagar lançamento recorrente

### Médio prazo (pós-validação)
- [ ] Backend com Firebase ou Supabase
- [ ] Autenticação (e-mail/senha ou Google)
- [ ] Sincronização entre dispositivos
- [ ] Controle de acesso por token para distribuição paga
- [ ] Domínio próprio (`meubalanco.com.br` ou `meubalanco.app`)

### Longo prazo (produto escalado)
- [ ] Modelo de assinatura mensal (SaaS)
- [ ] Versão Flutter para iOS e App Store (base de código já validada)
- [ ] Histórico e gráficos de evolução patrimonial
- [ ] Integração com Open Finance (leitura automática de extratos)
- [ ] Multi-usuário / modo casal

---

## Monetização

Estratégia atual: infoproduto — distribuição do link mediante pagamento único. Sem backend necessário nesta fase.

Evolução natural: SaaS com plano gratuito limitado e plano pago com sincronização em nuvem, histórico longo e exportações avançadas.

---

## Instalação (desenvolvimento local)

Por ser um arquivo estático, qualquer servidor HTTP funciona:

```bash
# Python
python3 -m http.server 8080

# Node
npx serve .
```

Abrir `http://localhost:8080` no browser.

> Service Worker só registra em HTTPS ou localhost — desenvolvimento local funciona normalmente.
