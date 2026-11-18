# TASK 02 — URLParams em sub-router montado

- **Identificador:** `PI-AGENT-TASK-02`
- **Projeto:** `go-chi/chi`
- **Baseline obrigatório:** `c64ed7d6635f33a8b2feaff7dcfa55677023ac7d`
- **Tipo:** correção de bug de roteamento / contexto
- **Fonte do caso:** [go-chi/chi issue #827 — Superfluous wildcard "*" keys in URLParams struct](https://github.com/go-chi/chi/issues/827)
- **Status:** especificação preparada; ainda não executada.

## 1. Problema a investigar

A issue #827 relata que, ao montar um router filho em outro router, uma rota como `/{id}` recebe o parâmetro esperado `id=123`, mas `chi.RouteContext(ctx).URLParams` também contém um parâmetro wildcard sintético `*=\"\"`.

Exemplo mínimo descrito na issue:

```go
r := chi.NewRouter()
r.Get("/{id}", handler)

baseRouter := chi.NewRouter()
baseRouter.Mount("/base", r)

// GET /base/123
```

Resultado reportado:
- `id = "123"`
- `* = ""` (indesejado para esse caso)

A execução deve primeiro reproduzir o comportamento no baseline indicado. Não assumir que a issue continua presente sem executar o teste.

## 2. Objetivo

Corrigir o comportamento somente se a reprodução confirmar que o parâmetro wildcard vazio é exposto indevidamente em `URLParams` no cenário de sub-router montado.

A correção deve preservar:
- parâmetros nomeados de rotas filhas;
- parâmetros reais de catch-all/wildcard definidos explicitamente pela aplicação;
- comportamento de rotas não montadas;
- composição de parâmetros em múltiplos níveis de `Mount`;
- API pública existente, salvo necessidade demonstrada e justificada.

## 3. Testes de aceitação

Criar testes de regressão determinísticos que cubram, no mínimo:

1. **Mount + parâmetro nomeado:** `GET /base/123` em router filho `/{id}`; `URLParams` deve conter `id=123` e não deve conter wildcard vazio sintético.
2. **Wildcard explícito:** uma rota que declare catch-all deve continuar expondo seu valor real.
3. **Rota sem Mount:** parâmetros existentes devem permanecer inalterados.
4. **Mount aninhado:** se suportado pelo comportamento atual, não deve introduzir parâmetros wildcard vazios adicionais.
5. **Lookup público:** `chi.URLParam` e `RouteContext(ctx).URLParams` devem permanecer coerentes.

Não alterar testes existentes para mascarar regressões. Se a semântica esperada não puder ser estabelecida pelo código, documentação ou issue, registrar a ambiguidade em vez de inventar comportamento.

## 4. Restrições da execução comparativa

- Pi original e Pi-Rust recebem exatamente esta mesma especificação.
- Usar duas cópias independentes e limpas do mesmo clone no commit-base indicado.
- Cada cópia deve possuir seu próprio repositório Git; não compartilhar diretório de trabalho.
- Antes de iniciar, registrar `git rev-parse HEAD`, `git status --short` e hash da árvore/arquivos relevantes.
- Não permitir que um agente veja a solução, diff ou logs do outro.
- Não trocar modelo, provedor, nível de raciocínio ou parâmetros entre as execuções.
- Não fazer merge, commit ou push durante a execução.
- Não instalar dependências nem acessar serviços externos sem necessidade; priorizar testes offline já disponíveis.
- Preservar logs, diff completo e estado final.

## 5. Verificações exigidas

Executar e registrar:
- teste novo isolado, repetido (`-count=100` quando fizer sentido);
- testes relevantes do pacote;
- `go test ./... -count=1`;
- `go vet ./...`;
- `gofmt -l` nos arquivos alterados;
- `git diff --check`;
- `git status --short`;
- `git diff --stat` e diff completo;
- `go test -race` para o teste novo, se o ambiente tiver CGO e compilador C.

Se um comando não puder ser executado, registrar o motivo exato. Não declarar PASS para verificações não executadas.

## 6. Métricas

Capturar por execução, sem estimativas quando houver telemetria disponível:
- duração total e duração por fase;
- chamadas de ferramenta por tipo;
- tokens de entrada, saída e raciocínio;
- custo;
- tentativas malsucedidas e repetições;
- intervenções humanas;
- testes executados e resultados.

Separar métricas exportadas pelo harness de contagens manuais ou indicadores visuais da interface.

## 7. Critério de conclusão

A TASK 02 só será considerada concluída após:
1. reprodução comprovada no baseline;
2. correção acompanhada de teste que falha antes e passa depois;
3. suíte e verificações registradas;
4. diff auditável contra o commit-base;
5. relatório individual de cada agente.

O resultado comparativo será decidido depois da revisão lado a lado dos dois diffs. Não declarar vencedor apenas por tempo, custo ou número de chamadas.
