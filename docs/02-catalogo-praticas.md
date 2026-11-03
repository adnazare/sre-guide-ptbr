<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/adnazare/sre-guide-ptbr/main/assets/banner-02.svg">
    <img src="https://raw.githubusercontent.com/adnazare/sre-guide-ptbr/main/assets/banner-02-light.svg" alt="Cabeçalho da página 02 de 07 do guia: Práticas essenciais de SRE" width="880">
  </picture>
</p>

# Práticas essenciais de SRE

*Para quem: SRE, desenvolvimento e plataforma · Leitura: cerca de 8 min*

**O essencial**

- Comece por um serviço e uma jornada do usuário. Não é preciso adotar tudo de uma vez.
- Alerte por impacto no usuário e por burn rate, não por CPU.
- Prepare incidente e plantão antes da falha e teste falhas com hipótese.

Comece com um serviço importante para o usuário. Defina uma meta, meça o que acontece, prepare-se para falhas e melhore com o que aprender. Não é preciso adotar todas as práticas de uma vez.

## Mapa das práticas

Status e esforço são avaliação editorial do guia, não métrica de mercado. “Sinal de que funcionou” vem dos critérios de “pronto” do [caminho de adoção](05-adocao-caminho-cenarios.md).

| Prática | Categoria | Status | Esforço | Sinal de que funcionou |
| --- | --- | --- | --- | --- |
| [SLI, SLO e error budget](#sli-slo-e-error-budget) | SLI/SLO | Consolidado | Médio | Uma jornada com alerta de budget e uma política que alguém já aplicou |
| [Monitoramento e observabilidade](#monitoramento-e-observabilidade) | Monitoring, observability | Consolidado | Médio a alto | Trace de um 5xx aberto a partir do exemplar |
| [Incidentes e plantão](#incidentes-e-plantão) | Incident management, on-call | Consolidado | Baixo a médio | SEV-1 com postmortem em até cinco dias úteis |
| [Automação e capacidade](#automação-e-capacidade) | Toil, capacity planning | Consolidado; remediação por IA é emergente | Médio | Custo de telemetria visível por serviço e série ativa estável após um release |
| [Entrega segura e testes de falha](#entrega-segura-e-testes-de-falha) | Release engineering, chaos | Canary consolidado; chaos em produção exige mais maturidade | Médio a alto | Um rollback automático registrado e um game day em produção com hipótese |
| [Resiliência e recuperação](#resiliência-e-recuperação) | Reliability, DR | Consolidado | Médio a alto | Um teste de restauração dentro do RTO e um failover ensaiado |
| [IA em produção como serviço](#ia-em-produção-como-serviço) | SLO, custo, governança | Emergente | Alto | Um SLO por caminho, custo por tenant visível e agente testado desligado |
| [Plataforma e Kubernetes](#plataforma-e-kubernetes) | Platform SRE | Consolidado | Médio a alto | Serviço novo chega ao primeiro deploy já com burn rate |

## SLI, SLO e error budget

Escolha uma jornada do usuário, como login ou checkout.

- **SLI** (indicador de nível de serviço) é a medida do serviço percebida pelo usuário, como proporção de pedidos bem-sucedidos ou atendidos dentro do tempo esperado.
- **SLO** (objetivo de nível de serviço) é a meta acordada para essa medida, numa janela definida. Disponibilidade e latência podem ter metas separadas.
- **Error budget** é a margem de falha permitida pelo SLO. Use-a para decidir quando seguir com mudanças ou priorizar confiabilidade.
- Alerte pelo **burn rate**: a velocidade com que o serviço consome o error budget. Página para queima rápida; use ticket para tendências lentas.

Exemplo: um SLO de 99,9% significa que até 0,1% dos eventos válidos podem falhar na janela escolhida. A meta não deve ser copiada sem observar o serviço e combinar a expectativa com produto.

<details>
<summary>Ver exemplo de SLO em YAML (formato ilustrativo)</summary>

```yaml
service: checkout
window: 30d
indicators:
  - name: availability
    good: successful_valid_requests
    total: valid_requests
    objective: 0.999
```

O formato é ilustrativo. Veja as [regras e exemplos completos](03-exemplos-implementacao.md).

</details>

## Monitoramento e observabilidade

Meça o serviço antes de escolher dashboards ou alertas. Comece por tráfego, erros, latência e saturação; complemente com traces quando precisar localizar uma dependência lenta ou com falha.

- Use **RED** (taxa, erros e duração) para serviços que atendem requisições.
- Use **USE** (utilização, saturação e erros) para investigar recursos como CPU, disco e pools.
- Página deve indicar impacto atual ou iminente para o usuário e uma ação humana. O restante pode virar ticket ou informação de dashboard.
- Combine telemetria interna com um teste externo da jornada crítica quando isso ajudar a detectar falhas que o serviço não enxerga. Health checks ajudam a operar o processo, mas não provam que a jornada do usuário funciona.
- Teste o próprio alerta: simule a falha e confirme que a métrica muda, o alerta dispara e o dashboard mostra a condição. Simuladores que mantêm estado ao longo do tempo ajudam, mas, se não forem fiéis ao sistema real, geram falsa confiança.
- Controle cardinalidade, retenção e dados sensíveis. Evite colocar identificadores pessoais em métricas, logs ou traces: telemetria com dado pessoal também entra no escopo da LGPD (Lei Geral de Proteção de Dados), e o ponto mais barato de remoção é o collector.
- OpenTelemetry ajuda a instrumentar serviços e encaminhar sinais para backends. Avalie operação e custo do backend separadamente.
- Para jornadas digitais, complemente métricas da API com uma medida da experiência real; um nome como XLO não substitui a definição clara de SLI e SLO.

Para serviços com IA, HTTP 200 não mede se a resposta é útil. Acompanhe qualidade avaliada, latência, custo e falhas do modelo; veja [SRE e IA no panorama](01-panorama-sre-2026.md#como-ler-esses-números).

## Incidentes e plantão

Prepare o processo antes de uma falha:

1. Defina níveis de severidade pelo impacto no usuário.
2. Em incidentes maiores, separe quem coordena, quem atua no sistema e quem comunica.
3. Registre horário, impacto, decisões e ações numa timeline.
4. Após o incidente, documente fatores contribuintes e ações com responsável e prazo.
5. Revise alertas que não levaram a uma ação e ajuste-os.

Declare o incidente cedo quando houver impacto ou risco relevante. Mantenha o plantão sustentável: escala clara, handoff, turnos-sombra para quem entra, acesso temporário e revisão de páginas repetidas. O SRE Book usa como teto no máximo dois incidentes por turno de 12 horas; se a média passa disso, o problema é o sistema, não a escala. Postmortem sem culpa busca melhorar o sistema, não procurar um culpado.

<details>
<summary>Ver checklist de incidente: declarar, medir, organizar, mitigar, comunicar, registrar, encerrar e aprender</summary>

- [ ] **Declarar:** qual serviço e jornada foram afetados? Quando começou? Qual a severidade e quem está liderando?
- [ ] **Medir:** qual sinal confirma o impacto? Está acontecendo agora? Quais regiões, versões ou clientes estão envolvidos?
- [ ] **Organizar:** quem coordena, quem investiga e quem comunica? Todos sabem seus papéis e o canal de trabalho?
- [ ] **Mitigar:** qual ação reduz o impacto mais rápido? É reversível? Como confirmar que funcionou e como voltar atrás?
- [ ] **Comunicar:** quem precisa ser informado, por qual canal e quando será a próxima atualização? Diga o que se sabe e o que ainda está sendo investigado.
- [ ] **Registrar:** anote horários, sinais consultados, mudanças, decisões, responsáveis e resultado de cada ação. Preserve links para alertas, deploys e dashboards.
- [ ] **Encerrar:** o serviço voltou ao nível esperado? Os sinais confirmam recuperação? Há riscos ou ações imediatas sem responsável?
- [ ] **Aprender:** marque revisão sem culpa, atribua responsáveis e prazos às ações e atualize runbook ou alertas quando necessário.

</details>

### O que revisar periodicamente

- **Mudanças recentes:** deploys, configuração, dependências, provedor de cloud ou modelo de IA antes e durante o incidente.
- **Sinais do usuário:** disponibilidade e latência da jornada; CPU ou saúde do pod ajudam no diagnóstico, mas não provam impacto por si só.
- **Qualidade dos alertas:** páginas sem ação, incidentes sem detecção e sinais que chegaram tarde.
- **Preparação do plantão:** runbooks que ainda funcionam, contatos e acessos disponíveis, handoff e cobertura da escala.
- **Ações recorrentes:** causas que voltaram, correções atrasadas e toil repetido que pode ser removido.
- **Mudanças de ferramentas e dependências:** avisos oficiais de suporte, limites e alterações que afetem detecção, coordenação ou recuperação.

Revise esses pontos periodicamente e pratique a resposta em exercícios curtos.

## Automação e capacidade

Toil (trabalho manual e repetitivo que cresce com o serviço) só diminui se alguém o mede. Registre tarefas recorrentes e automatize primeiro as que têm passos conhecidos e reversíveis. Mantenha revisão humana para decisões que dependem de contexto.

Para capacidade, acompanhe o gargalo real: filas, pools, latência de dependências e limites de recursos. Teste picos de forma isolada e combine limites para autoscaling. Evite usar CPU média como único sinal.

## Entrega segura e testes de falha

- Faça mudanças em etapas pequenas; use canary (liberar a mudança primeiro para uma fatia pequena do tráfego) quando houver tráfego e sinais suficientes para comparar com a versão estável.
- Defina previamente condições para interromper a promoção e como reverter.
- Teste falhas com hipótese, escopo e condição de abortar. Comece em ambiente controlado e avance com autorização.
- Não trate staging como prova de resiliência em produção; ele pode não reproduzir tráfego, dados e dependências reais.
- Não habilite agente de remediação antes de testar o processo com o agente desligado.
- Dê aos pipelines e agentes privilégio mínimo e credenciais temporárias (confiança zero), em vez de chaves fixas com acesso amplo.
- Meça entrega e estabilidade juntas (métricas DORA). Acelerar a entrega de código sem olhar para falhas e recuperação não prova melhora.

Veja modelos de [canary, rollback e postmortem](03-exemplos-implementacao.md).

## Resiliência e recuperação

Disponibilidade no dia bom não prova recuperação no dia ruim.

- Defina **RTO** (tempo máximo aceitável para voltar) e **RPO** (perda máxima aceitável de dados) por serviço, a partir do impacto no usuário, e não um valor único para a empresa.
- Backup que nunca foi restaurado não é backup. Agende um teste de restauração e registre o tempo real contra o RTO.
- Ensaie failover e a perda de uma zona antes de precisar dela. Multirregião só entra quando o SLO ou o requisito regulatório exigem: custa dinheiro e complexidade.
- Registre dependências críticas (provedor, DNS, identidade, modelo de IA) e o que o serviço faz quando uma delas cai.

## IA em produção como serviço

Quando há modelo ou agente no caminho do usuário, ele é um serviço como outro qualquer, com três diferenças que pedem atenção desde o primeiro dia:

- **Um SLO por caminho, não um por sistema.** Resposta de cache e resposta gerada pelo modelo têm latências muito diferentes. Um número único esconde a piora do caminho caro. Veja o [exemplo](03-exemplos-implementacao.md#slo-de-um-agente-de-ia-latência-com-e-sem-cache).
- **Qualidade ao lado da disponibilidade.** Um agente pode responder rápido e errado. Meça a fração de respostas com fonte citada ou aprovadas numa avaliação por amostra, e mantenha esse número separado do 5xx.
- **Custo e desempenho por tenant e por modelo.** É o jeito de ver quem consome o orçamento e qual modelo piorou. Alerte sobre custo como se alerta sobre erro.

E o básico de sistemas distribuídos continua valendo: retry com limite, fila de mensagens mortas (DLQ) para o que não processou, degradação com mensagem clara, gateway com controle de acesso, isolamento por tenant, política como código e auditoria central. Antes de introduzir agentes, registre a linha de base de golden signals e de DORA. Sem ela, você não sabe se melhorou.

## Plataforma e Kubernetes

Se muitos times repetem os mesmos passos, crie um caminho padrão que já inclua métricas, alertas, deploy e runbook. A plataforma deve facilitar o caminho seguro, não virar um gate manual para cada mudança.

Em Kubernetes, confira:

- `requests` e `limits` adequados ao serviço;
- probes de startup, readiness e liveness com finalidades diferentes;
- PodDisruptionBudget (limite de pods que podem ficar indisponíveis durante manutenção) e distribuição entre zonas quando necessários ao SLO;
- autoscaling ligado à métrica que representa o gargalo.

Esses recursos ajudam a operar workloads, mas não substituem SLI e SLO do usuário.

---

[← Anterior: Panorama](01-panorama-sre-2026.md) · [Índice do guia](../README.md#mapa-do-guia) · [Próxima: Exemplos →](03-exemplos-implementacao.md)
