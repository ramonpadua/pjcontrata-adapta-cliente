# Changelog — IA Ferramenta

## 2026-10-21 — F2-T07: aceite humano da publicação real; task fechada

- Owner aceitou a publicação real ("Aceito a publicação real da F2-T07 — pode concluir a task", 21/10/2026).
- Verificação independente refeita do zero no fechamento: backend /api/health 200; migrations 0001–0014 aplicadas (0014_f2t07_google_object_types confirmada); build v0.0.125 (314d67b).
- Prova de acesso fresca: PROVA_OK — 6 customers acessíveis, teste completo (239-156-***, BRL, São Paulo, test_account true), demais fora da allowlist; zero mutação; zero segredo.
- Banco: tentativa 05td87wxn53g8t7 CONFIRMED_PAUSED (operator teste.marketing, 25/09 14:49 BRT); 2 remote_objects (campaign PAUSED + budget NORMAL, remote_id mascarado, remote_id_real apenas no campo interno); trilha PEDIDO_CRIADO → ENVIO_INICIADO → ESTADO_CONFIRMADO com payload_hash.
- Gates reprovados como esperado: publicar campanha ARCHIVED → 400 "Campanha não está no estado APPROVED"; reconciliar tentativa CONFIRMED_PAUSED → 400 (só UNCERTAIN/PARTIAL_FAILURE).
- RN-226: zero segredo/remote_id_real nas respostas verificadas.
- Aprendizado: AP-2026-09-25-1445-google-ads-v25-obrigatorios.md recriado (commit 5951e76); controle.md atualizado com F2-T05/T06/T07 (referências a recibos onde o AP nunca foi commitado).
- Reparo de incidente: commit 4486ccf criou 06_noteas/ com conteúdo corrompido e commit 2abe32a inseriu linhas corrompidas no controle.md — ambos limpos no commit e01d4ae (06_noteas removido em b7a041d; controle.md íntegro).
- Fase 2: 7/9 (78%). Próxima elegível: F2-T08 (falha parcial, timeout e reconciliação Google).

## 2026-09-25 — F2-T07: publicação REAL na Google Ads provada

- Contas de teste criadas pelo owner: MCC "EngajaPJ Teste" 103-812-9898 + cliente "EngajaPJ Cliente Teste" 2391563500 (testAccount TRUE; status CLOSED é o estado normal de conta de teste na v25).
- Secrets GOOGLE_TEST_CUSTOMER_ID/GOOGLE_TEST_LOGIN_CUSTOMER_ID gravados; allowlist ad_connection google/teste criada (mnm3k9i72kgpnn4).
- Prova de acesso agora mostra o customer de teste completo (239-156-***, BRL, São Paulo) — consulta via GAQL searchStream (GET direto de resource name retorna 404 HTML na v25).
- PUBLICAÇÃO REAL (builds v0.0.120–125): campanha T-F2T07-REAL-FINAL criada NA GOOGLE ADS pela rota do sistema — budget + campaign SEARCH em PAUSED, confirmada por consulta GAQL (CA-2-008).
- Idempotência real (CA-2-009): retry da mesma chave → "nada duplicado"; 1 única campanha na conta de teste, provado via GAQL.
- Descobertas v25: containsEuPoliticalAdvertising é OBRIGATÓRIO (FieldError.REQUIRED sem ele; UNDECLARED/UNSPECIFIED inválidos — usar DOES_NOT_CONTAIN_EU_POLITICAL_ADVERTISING); mutate combinado budget+campaign com resource names temporários (-1/-2) é atômico; REST devolve camelCase (resourceName); BudgetStatus "NORMAL" não existe no enum.
- RN-226 verificado: nenhum segredo/remote_id_real nas respostas.
- Pendência restante da task: aceite humano da publicação real (o owner já aprovou o bloqueio em 22/09).

## 2026-09-22 — F2-T07 concluída (parte 1: gates e adaptador)

- F2-T07: publicação Google pausada, confirmada e idempotente (build v0.0.119).
- Hook publicacao_google.js (adaptador isolado por plataforma, reusa coleções F2-T01): POST /backend/v1/publicacoes-google + reconciliar.
- Gates server-side (CA-2-007/RN-211/212): papel, aprovação, APPROVED, checklist READY, snapshot, plataforma google, allowlist, environment teste — todos ANTES da API.
- Credencial expirada (401/403) → BLOCKED (RN-222); timeout → UNCERTAIN (RN-223).
- Idempotência `pubg:` prefix; reconciliação completa campaign faltante → RECOVERED.
- Verificação: 8/9 (falha externa: Meta 500 transitório is_transient na sandbox — token válido, núcleo Meta intocado).
- Modelo Google da F1 ativado (estava inativo; aplicar-modelo valida plataforma E objetivo — campanha google precisa objective 'leads').
- Teste humano aprovado pelo owner (22/09/2026, 08:19): campanha Google válida → BLOCKED limpo antes da API, zero tentativa fantasma, repetição consistente.
- Aprendizado AP-2026-09-22-0822: adaptador por plataforma em hook próprio reusando o núcleo; ausência de conta de teste não bloqueia implementação (gates bloqueiam antes da API; caminho real pronto sem novo deploy).
- Fase 2: 7/9 (78%).

## 2026-09-21/22 — F2-T06 concluída

- F2-T06: prova de acesso Google (CA-2-006) — build v0.0.118.
- Hook google_prova.js: rota server-side READ-ONLY /backend/v1/google/prova-acesso — sem credenciais → BLOCKED_ACCESS com lista do que falta (nenhuma chamada externa); com credenciais → renova access token, chama customers:listAccessibleCustomers (v25), cruza com allowlist e classifica; credencial expirada → BLOCKED_ACCESS "renovar fora do log"; IDs mascarados; RN-226; nenhuma mutação.
- Credenciais OAuth provisionadas pelo owner (projeto Cloud agenda-hub): GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET, GOOGLE_REFRESH_TOKEN nos secrets do Skip. Developer token opcional (sunset 09/09/2026 — nível de acesso vem do projeto Cloud).
- Prova real: PROVA_OK na 1ª tentativa — 4 customers acessíveis classificados fora_da_allowlist (429-956, 171-276, 852-277, 785-059 — mascarados), zero mutação, zero segredo na resposta.
- Pendência transferida para F2-T07: conta de teste Google Ads — a UI do Google prende o fluxo de criação num assistente de campanha que exige forma de pagamento; 3 rascunhos de conta inertes sem cobrança; não confirmar cartão.
- Recibo 05_entregas/fase-2/recibo-F2-T06.md.
- Fase 2: 6/9 (67%).

## Histórico anterior (14/09–17/09)

- F2-T01 concluída (14/09): núcleo transversal de publicação (v0.0.100–103, 23/23 testes).
- F2-T02 concluída (14/09): idempotência, log append-only e recibos (v0.0.104–105, 15/15).
- F2-T03 concluída (16/09): prova de acesso Meta (sandbox act_1585807056675719, token nos secrets, allowlist).
- F2-T04 concluída (16/09): publicação Meta real (v0.0.110, 12/12 testes).
- F2-T05 implementada e concluída (16–17/09, v0.0.111–117): falha parcial com preservação RN-205, credencial expirada → BLOCKED, reconciliação REAL via remote_id_real; 9/9 testes; aceite do owner após demo ao vivo (falha real + recuperação real com adset verificado na Meta).
- F2-T06 concluída (18/09): prova de acesso Google (v0.0.118, 6/6 testes; PROVA_OK com credenciais OAuth reais do projeto agenda-hub).
