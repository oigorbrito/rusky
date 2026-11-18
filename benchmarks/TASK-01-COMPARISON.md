# Comparação — TASK 01 — Pi Original vs. Pi-Rust

- **Data:** 2026-11-18
- **Projeto:** `github.com/go-chi/chi/v5`
- **Tarefa:** corrigir admissão de requisição com contexto já cancelado em `ThrottleWithOpts`
- **Status:** **EMPATE FUNCIONAL PROVISÓRIO — auditoria de diff pendente**
- **Relatórios individuais:** [Pi-Rust](./PI-RUST-TASK-01.md) · [Pi Original](./PI-ORIGINAL-TASK-01.md)

## 1. Resultado técnico

Os dois agentes identificaram o mesmo problema: no select de admissão, `ctx.Done()` e a disponibilidade de um token de backlog podem estar prontos simultaneamente. A seleção não determinística pode permitir que uma requisição já cancelada avance até o handler.

Ambos reportaram a mesma estratégia de correção: verificar `ctx.Err()` antes do select, rejeitar a requisição e não consumir tokens.

Ambos modificaram:
- `middleware/throttle.go`
- `middleware/throttle_test.go`

## 2. Comparação de validação

| Critério | Pi-Rust | Pi Original |
|---|---|---|
| Reprodução antes da correção | 100/100 falhas | 100/100 falhas |
| Regressão depois da correção | 100/100 PASS | 100/100 PASS |
| Suíte completa `go test ./...` | PASS | PASS |
| `go vet ./...` | PASS | PASS |
| Formatação | `gofmt -l middleware`: limpo | `gofmt -l .`: limpo |
| Testes de throttle repetidos | PASS, conforme relatório | PASS, `-count=3` |
| Race Detector | Não executado | Não executado |
| Git/diff no workspace | Não disponível | Não disponível |
| Commit-base confirmado | Não | Não |

**Leitura:** empate na evidência funcional reportada. Ainda não existe base para diferenciar a qualidade interna das implementações, pois os diffs completos não foram comparados.

## 3. Telemetria visual das sessões

Captura fornecida pelo usuário em 2026-11-18. Valores transcritos literalmente:

| Indicador exibido | Pi Original | Pi-Rust |
|---|---:|---:|
| `↑` | 50 | 42 |
| `↓` | 25k | 22k |
| `R` | 505k | 349k |
| `W` | 71k | 42k |
| `CH` | 94.7% | 98.8% |
| Custo | US$ 0.027 | US$ 0.020 |
| Modelo exibido | claude-haiku-5-5 | claude-haiku-5-5 |
| Indicador adicional | `(sub)` | `medi...` |

O custo visual do Pi-Rust foi aproximadamente 25,9% menor (US$ 0.007 de diferença). R e W também aparecem menores no Pi-Rust. Os símbolos `↑` e `↓` foram preservados sem interpretação porque a semântica dos contadores não foi documentada.

**Ressalva de comparabilidade:** a captura sugere indicadores de configuração diferentes (`(sub)` no Pi Original e `medi...` no Pi-Rust). Portanto, a diferença de custo/consumo é apenas uma vantagem preliminar observada, não uma conclusão causal sobre o harness. É necessário confirmar que as configurações de raciocínio e demais parâmetros eram equivalentes.

## 4. Eficiência reportada

| Métrica | Pi-Rust | Pi Original |
|---|---|---|
| Tempo | Não disponível no relatório | ~22 min, inferido |
| Chamadas de ferramentas | ~19 Bash (relatório individual anterior) | ~19 chamadas manuais |
| Operações Edit | 2 | 2 |
| Tokens de entrada/saída | Não disponíveis no relatório | Não disponíveis no relatório |
| Custo | US$ 0.020 na captura | US$ 0.027 na captura |
| Intervenção humana | Nenhuma reportada | Nenhuma reportada |

Os números de chamadas são estimativas manuais, não métricas instrumentadas. Não se deve inferir produtividade apenas deles.

## 5. Auditoria pendente e resolução

Os dois workspaces não possuem `.git`, mas isso não bloqueia a recuperação do diff. O procedimento de auditoria é:

1. Confirmar que o clone original limpo de `go-chi/chi` está exatamente no commit `c64ed7d6635f33a8b2feaff7dcfa55677023ac7d`.
2. Comparar o clone limpo contra `go-chichipioriginal/chi-master`.
3. Comparar o mesmo clone limpo contra `go-chichipirust/chi-master`.
4. Registrar diffs completos, arquivos alterados, adições/remoções e alterações colaterais.
5. Comparar implementação e testes lado a lado.
6. Executar `go test -race ./middleware -run '^TestThrottleAlreadyCanceled$' -count=100 -timeout=60s` em ambiente com CGO e compilador C, se disponível.

Não é necessário repetir a implementação para recuperar os diffs.

## 6. Veredito da TASK 01

- **Correção funcional:** empate, conforme testes reportados.
- **Eficiência:** vantagem preliminar do Pi-Rust na captura (US$ 0.020 vs. US$ 0.027), sujeita à confirmação de equivalência de configuração.
- **Qualidade do diff:** não avaliada ainda.
- **Confiabilidade concorrente:** incompleta; `-race` ausente nos dois.
- **Vencedor geral:** não declarado.

A TASK 01 fica registrada como **empate funcional provisório, auditoria de diff pendente**. A próxima tarefa deve usar workspaces independentes derivados do mesmo commit, com baseline Git preservado e parâmetros idênticos.
