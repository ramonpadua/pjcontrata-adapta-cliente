# AP-2026-09-25-1445 — Google Ads API v25: exigências na criação de campanha

- Status: candidato
- Escopo: projeto do cliente
- Task/SPEC: F2-T07 · SPEC-2-002 (Google Ads — publicação controlada)
- Sinal: na criação de campaign via REST v25, `containsEuPoliticalAdvertising` é OBRIGATÓRIO (FieldError.REQUIRED sem ele; UNDECLARED/UNSPECIFIED inválidos — usar `DOES_NOT_CONTAIN_EU_POLITICAL_ADVERTISING`); mutate combinado budget+campaign com resource names temporários (-1/-2) via `googleAds:mutate` é atômico — falha no request = nada criado, zero objeto órfão; consultas de estado são via GAQL searchStream (GET direto de resource name retorna 404 HTML); REST devolve camelCase (`resourceName`) — aceitar `resource_name` OU `resourceName`; BudgetStatus "NORMAL" não existe no enum v25 (usar default); conta de teste rejeita enhancedCpc (OPERATION_NOT_PERMITTED) e maximize* (CONVERSION_TRACKING_NOT_ENABLED) — `manualCpc {enhancedCpcEnabled:false}` é o caminho.
- Evidência: 05_entregas/fase-2/recibo-F2-T07.md (publicação real T-F2T07-REAL-FINAL 24289350920 PAUSED confirmada por GAQL; retry idempotente sem duplicar); tentativas de diagnóstico de 25/set na coleção publication_attempt com erros reais sanitizados (BudgetStatus, network_setting, invalid argument).
- Regra reutilizável: ao montar payload de campaign na Google Ads API v25+, incluir `containsEuPoliticalAdvertising=DOES_NOT_CONTAIN_EU_POLITICAL_ADVERTISING` e usar mutate combinado com resource names temporários; consultar estado por GAQL searchStream, nunca por GET de resource name.
- Quando aplicar: qualquer criação/consulta de recursos Google Ads via REST no Engaja PJ (e integrações futuras na mesma versão ou superior).
- Quando não aplicar: versões anteriores à v25 (campos/enum podem diferir); mutação de recurso já existente (usar update, não create).
- Confiança: alta — provado em publicação real com confirmação por consulta e retry idempotente.
- Privacidade: sem segredo, dado pessoal ou conteúdo bruto.