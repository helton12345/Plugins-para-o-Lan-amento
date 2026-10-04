# Regras permanentes para o Claude neste repositório

## Auditorias e análises pesadas (regra do usuário — NUNCA descumprir)
- **Nunca** rodar auditoria pesada (workflow com vários agentes, fan-out de subagentes, verificação adversarial,
  varredura "exaustiva" do plugin inteiro) **sem o usuário aceitar antes**, depois de ele ver a estimativa de custo.
- Motivo: já aconteceu — uma auditoria de ≈ 2,1 milhões de tokens estourou o limite da sessão, deixou o trabalho
  parado e não rendeu nenhuma correção verificada.
- Padrão: ler só o necessário, reproduzir o problema com um teste curto, corrigir o mínimo. Para bugs já
  apontados, ir direto ao arquivo/função citado. Se uma análise ampla parecer útil, **propor e esperar o "sim"**
  (dizendo o custo aproximado), nunca iniciar por conta própria — mesmo que "ultracode" ou um skill sugira workflow.
- Antes de qualquer workflow/subagente: o usuário precisa ter pedido explicitamente esse método naquela conversa.
