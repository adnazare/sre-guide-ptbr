<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/adnazare/sre-guide-ptbr/main/assets/banner-06.svg">
    <img src="https://raw.githubusercontent.com/adnazare/sre-guide-ptbr/main/assets/banner-06-light.svg" alt="Cabeçalho da página 06 de 07 do guia: Ferramentas de SRE: como comparar opções em 2026" width="880">
  </picture>
</p>

# Ferramentas de SRE: como comparar opções em 2026

*Para quem: quem avalia ferramentas de observabilidade, incidentes e IA · Leitura: cerca de 15 min; use o mapa para ir direto à área*

**O essencial**

- Regra do guia: instrumente com OpenTelemetry, use um backend e mantenha Prometheus para alerta de infraestrutura.
- Desempate por cardinalidade, retenção, exportação, auditoria, residência de dados e evidência nas sugestões de IA.
- Em Kubernetes, separe as decisões: entregar (Argo CD, Devtron), liberar com segurança (Argo Rollouts, Flagger), operar o cluster (Karpenter, KEDA, Kyverno, Velero) e declarar SLOs (Pyrra, Sloth).
- Agentes de SRE já estão disponíveis em vários fornecedores (Datadog, Azure, AWS, PagerDuty, Grafana). Trate como emergente: comece em modo somente leitura e exija evidência.
- Opsgenie: não escolha para adoção nova.

É uma comparação editorial, não um ranking nem uma recomendação universal. Preços e recursos mudam por contrato, região e versão: confirme na documentação do fornecedor e faça uma prova de conceito no seu contexto. Última revisão: 3 de novembro de 2026.

Siglas usadas: APM (monitoramento de desempenho de aplicações), OTLP (protocolo de telemetria do OpenTelemetry), AIOps (IA aplicada a operações), RBAC (controle de acesso por papéis), PII (dados pessoais identificáveis) e ITSM (gestão de serviços de TI).

## Mapa: o que uma pessoa SRE usa, por área

Use este mapa para achar a área; os detalhes e as cautelas estão nas seções abaixo.

| Área | Opções citadas neste guia |
| --- | --- |
| SLI, SLO e error budget | Pyrra, Sloth, OpenSLO |
| Métricas, logs e traces | Prometheus, Mimir, Loki, Tempo, Jaeger, SigNoz, Grafana, Datadog, New Relic, Honeycomb, Elastic |
| Coleta e instrumentação | OpenTelemetry (SDK e Collector), Grafana Alloy, OpenTelemetry eBPF Instrumentation (OBI) |
| Profiling contínuo | Grafana Pyroscope, Parca |
| Experiência externa | Catchpoint |
| Incidentes e on-call | PagerDuty, incident.io, Grafana IRM, Opsgenie (em descontinuação), Statuspage |
| Agentes de IA para SRE | Datadog Bits AI SRE, Azure SRE Agent, AWS DevOps Agent, PagerDuty Paige, Grafana Assistant, incident.io, HolmesGPT, K8sGPT, kagent, kubectl-ai |
| Entrega e GitOps | Argo CD, Flux, Argo Rollouts, Flagger, Devtron |
| Operação de clusters | Karpenter, KEDA, Karmada, Cluster API, Headlamp, Velero, Kyverno, Falco, Cilium, Istio |
| Custo | OpenCost, Kubecost |
| Infraestrutura como código e segredos | OpenTofu, Terraform, Crossplane, OpenBao, Vault, External Secrets |
| Chaos | Chaos Mesh, AWS Fault Injection Service |
| Portal de desenvolvedor | Backstage |

## Observabilidade

| Ferramenta | Melhor cenário | Custo e portabilidade | IA e cautelas |
| --- | --- | --- | --- |
| OpenTelemetry + OTLP | Padronizar instrumentação e manter portabilidade entre backends | O padrão é aberto; SDK e Collector exigem operação. O custo principal fica no backend | Sem IA própria; fornece telemetria para as ferramentas que a implementam |
| Prometheus | Métricas de infraestrutura, RED e alertas baseados em PromQL | Controle da cardinalidade; retenção local costuma ser curta; armazenamento remoto adiciona componentes | Não é uma ferramenta de IA |
| Grafana | Explorar e visualizar dados de várias fontes | OSS ou serviço pago; lock-in menor com fontes abertas | Recursos assistidos variam por produto e plano; valide versão e controles |
| Grafana Alloy | Coletar e encaminhar telemetria de diferentes fontes | Avalie CPU, configuração e operação do agente | Não é o motivo para escolher |
| Grafana Loki | Logs de alto volume com labels controlados | O modelo de labels e consultas LogQL influenciam lock-in e custo | Recursos dependem do produto Grafana usado |
| Grafana Tempo | Traces com entrada OTLP e integração com métricas | Volume de spans, retenção e armazenamento | Recursos dependem do produto Grafana usado |
| Grafana Mimir | Métricas Prometheus em escala | Object storage, queriers e cardinalidade elevam o custo operacional | Sem IA no núcleo |
| SigNoz | Plataforma OTel com logs, métricas e traces em ClickHouse | Self-host exige operar ClickHouse; compare custo total em carga equivalente | Recursos mudam rapidamente; valide release e política de dados |
| OpenTelemetry eBPF Instrumentation (OBI) | Obter métricas e traces de aplicações Linux sem alterar o código | Doado pela Grafana (Beyla) ao OpenTelemetry em 2025; versão 1.0.0 em 30 de outubro de 2026. Roda com privilégios elevados no nó | A propagação de contexto de traces varia por linguagem; para Java e .NET, planeje SDKs |
| Grafana Pyroscope e Parca | Profiling contínuo: qual código consumiu CPU ou memória em uma janela | Open source; Pyroscope tem servidor AGPL-3.0. Agentes eBPF exigem privilégios no nó | O sinal de profiles do OpenTelemetry está em alpha desde março de 2026 e não é indicado para produção crítica |
| ClickStack | Avaliar observabilidade sobre ClickHouse em cargas de alto volume | Compare cluster, retenção e operação com alternativas no volume real | Não o trate como AIOps |
| Datadog | APM e observabilidade gerenciada para equipes com orçamento de SaaS | Custo pode variar com hosts, métricas customizadas e ingestão; avalie dependência do agente | Recursos de IA e governança variam; valide tratamento de dados |
| New Relic | APM gerenciada já contratada quando telemetria e uso cabem no plano | Ingestão, usuários e dependência de produto | Valide recursos e limites do plano atual |
| Honeycomb | Investigar eventos e alta cardinalidade em sistemas distribuídos | Custo depende de eventos e retenção; OTLP ajuda na portabilidade | Valide quais análises assistidas estão disponíveis |
| Elastic Observability | Reaproveitar Elastic para logs, APM e métricas | Operação do cluster, shards, camadas de armazenamento e ingestão | Recursos assistidos dependem do stack contratado |
| Jaeger | Backend e interface de traces self-hosted com escopo focado | Custo e operação dependem do armazenamento escolhido | Não é uma plataforma de IA |
| Catchpoint | Medir experiência externa, DNS, rede e pontos de presença | Serviço especializado; avalie cobertura, frequência e lock-in dos dados | Não é o motivo principal para escolher |

**Regra de seleção do guia:** instrumente com OpenTelemetry, mantenha Prometheus/PromQL para alerta de infra, escolha um backend. Dois backends de trace sem um motivo de retenção ou de compliance duplicam custo e divergem número.

## Gestão de incidente e on-call

| Ferramenta | Melhor para | Limites e custo | IA e atenção |
| --- | --- | --- | --- |
| PagerDuty | Escalas, alertas, incidentes e automação de resposta | Licença depende de usuários, serviços e eventos; não define SLI nem corrige alertas ruidosos | Resumo e triagem assistidos exigem revisão humana, especialmente em SEV-1 |
| incident.io | Coordenar papéis, timeline, status e follow-up do incidente | Não substitui telemetria; custo depende do plano e do uso | Revise fatos e fontes antes de publicar atualizações ou postmortems |
| Opsgenie | Clientes existentes do ecossistema Atlassian | Fim de venda em 4 de junho de 2025; suporte até 5 de abril de 2027 | Não escolher para adoção nova; planeje migração e confirme prazos com a Atlassian |
| Grafana IRM (Grafana Cloud) | Integrar escalas, alertas e resposta a incidentes ao Grafana Cloud | Produto gerenciado; confirme plano, disponibilidade regional e integração necessária | Recursos assistidos dependem do produto e da configuração |
| Statuspage e páginas de status integradas | Comunicar impacto e atualizações a usuários | Não detectam nem coordenam, por si sós, o incidente | Automação de status precisa de fonte e aprovação |
| Jira / Linear | Acompanhar ações corretivas depois do incidente | Não substituem a sala, a timeline ou a ferramenta de resposta em tempo real | Não use um ticket como único registro operacional |
| Slack / Teams | Coordenar conversa durante o incidente | Histórico precisa ser convertido em timeline e decisões rastreáveis | Bot sem fonte verificável pode espalhar informação incorreta |

> [!WARNING]
> **Opsgenie:** fim de venda em 4 de junho de 2025 e fim de suporte em 5 de abril de 2027, conforme a página de licenciamento da Atlassian. Não é recomendado para adoção nova. Quem ainda opera no produto deve migrar para Jira Service Management antes dessa data. Fonte: [atlassian.com/licensing/opsgenie](https://www.atlassian.com/licensing/opsgenie).

## AIOps e assistentes

Esta categoria é emergente: trate cada recurso como um piloto delimitado, com critério de sucesso e plano de reversão.

| Classe | Bom ponto de partida | Risco a evitar | Controle mínimo |
| --- | --- | --- | --- |
| Correlação de eventos e anomalias | Recursos como Datadog Watchdog e equivalentes: apontar onde investigar | Resolver ou encerrar incidentes sem validação | Link para a série, janela e serviço que sustentam a sugestão |
| Resumo de incidente e rascunho de postmortem | Ferramentas como incident.io, PagerDuty ou fluxo interno: organizar a timeline | Publicar causa ou impacto sem revisão | Diff humano e referências a eventos e mensagens |
| Consulta de logs e traces em linguagem natural | Acelerar a busca por spans e eventos | Conclusões que a query não sustenta | Mostrar e preservar a query executada |
| Remediação automatizada | Ações idempotentes e limitadas, como rollback ou escala dentro de teto | Permissão ampla ou comando aberto em produção | Allowlist, kill switch, limite de impacto, auditoria e teste com agente desligado |
| Observabilidade de LLM | Evals, custo, latência e guardrails com ferramentas como Langfuse | Definir SLO de qualidade sem dataset de avaliação | Dataset versionado e SLI de qualidade separada de disponibilidade |

Dynatrace, Splunk e similares seguem relevantes em empresas reguladas. A escolha depende de contrato vigente e de residência de dado, não de uma funcionalidade de 2026. Exija OTLP na entrada antes de renovar.

## Entrega e plataforma no Kubernetes

| Ferramenta | Melhor cenário | Custo e portabilidade | IA e cautelas |
| --- | --- | --- | --- |
| Devtron | Painel e plataforma de entrega para vários clusters: CI/CD, Helm, GitOps (Argo CD e Flux), RBAC e visibilidade de custo | Tem edição open source (Apache 2.0) e edições comerciais; a documentação lista OSS, Freemium e Enterprise. Passa a ser uma camada entre o time e o cluster, então avalie a dependência | O fornecedor anuncia recursos de IA e um agente de SRE (Atlas). São afirmações do fornecedor: valide em prova de conceito e aplique os controles de remediação automatizada da seção de IA nas operações |
| Argo CD | GitOps declarativo para entrega contínua em Kubernetes | Open source; você opera o controller | Sem IA própria |
| Argo Rollouts | Canary e blue-green com análise de métricas e rollback | Open source; precisa de SLI por versão para decidir | Sem IA própria |
| Flagger | Progressive delivery acoplada a ingress ou service mesh | Open source; depende do ingress ou mesh escolhido | Sem IA própria |
| Backstage | Portal de desenvolvedor para catálogo de serviços e golden paths | Open source; exige uma equipe de plataforma para manter | Sem IA no núcleo |
| Port | Portal de desenvolvedor pronto para uso: catálogo, templates e autosserviço | Plataforma comercial: pese a dependência do fornecedor | Valide recursos de IA e o tratamento de dados |
| Apache APISIX | API gateway com gestão de tráfego, segurança e observabilidade, integrado à Gateway API do Kubernetes | Open source | Sem IA no núcleo |

Se você já usa Argo CD ou Flux, Devtron pode ser uma interface sobre o que existe, não uma troca. Se não tem equipe de plataforma, compare o custo de operar a ferramenta com o custo de manter os pipelines atuais.

## SLO e error budget

| Ferramenta | Melhor cenário | Custo e portabilidade | IA e cautelas |
| --- | --- | --- | --- |
| Pyrra | SLOs com Prometheus: gera recording rules e alertas de burn rate em várias janelas e oferece uma interface | Open source; só Prometheus. Os mantenedores informam que cuidam do projeto no tempo livre | Sem IA |
| Sloth | Gerar regras Prometheus de SLO e de burn rate a partir de uma definição simples | Open source; só Prometheus | Sem IA |
| OpenSLO | Descrever SLOs em um formato aberto e neutro de fornecedor | Especificação aberta; o suporte depende de cada ferramenta | Não é uma ferramenta de IA |

Qualquer uma gera o que o [exemplo de burn rate](03-exemplos-implementacao.md#alerta-por-burn-rate-página-e-ticket) mostra escrito à mão. Use-as quando houver muitos serviços; com um ou dois, o YAML manual basta.

## Chaos e testes de falha

| Ferramenta | Melhor cenário | Custo e portabilidade | IA e cautelas |
| --- | --- | --- | --- |
| Chaos Mesh | Experimentos de falha em Kubernetes | Open source; operação própria | Sem IA própria |
| AWS Fault Injection Service | Experimentos em recursos AWS, com condições de parada | Serviço gerenciado; específico da AWS | Sem IA própria |

Ferramenta não substitui hipótese, escopo e condição de abort. Veja [entrega segura e testes de falha](02-catalogo-praticas.md#entrega-segura-e-testes-de-falha).

## Agentes de SRE lançados em 2025 e 2026

Status em 3 de novembro de 2026, conforme anúncios dos fornecedores. Os ganhos divulgados (por exemplo, queda de MTTR) vêm dos próprios fornecedores e de períodos de prévia; meça no seu ambiente.

| Produto | Status | O que faz | Cautela |
| --- | --- | --- | --- |
| Datadog Bits AI SRE | Disponível desde 2 de dezembro de 2025 | Investiga alertas com a telemetria do Datadog e entrega uma conclusão de causa raiz | Raciocina sobre os dados que o Datadog tem; confira quais integrações externas estão em disponibilidade geral |
| Azure SRE Agent | Disponível desde 10 de março de 2026 | Investiga incidentes com recursos Azure, ferramentas de observabilidade e repositórios; sugere ou, se configurado, executa mitigações | Defina permissões, modo de execução e políticas antes de liberar ações |
| AWS DevOps Agent | Disponível desde 31 de março de 2026 | Investiga incidentes na AWS, em multicloud e on-premises, com CloudWatch e parceiros como Datadog, Grafana e PagerDuty | Revise permissões por conta e a trilha de auditoria |
| PagerDuty Paige e PD Reliability Platform | Disponível em 3 de novembro de 2026 | Detecta, investiga e remedia; pode operar com aprovação humana ou de forma autônoma em cenários aprovados | Lançamento recente: comece com aprovação humana |
| Grafana Assistant Investigations | Disponível no Grafana Cloud desde julho de 2026 | Investiga métricas, logs, traces e profiles e gera um relatório com evidências | Sem versão self-managed |
| incident.io Investigations | Disponível desde agosto de 2026, como complemento pago | Levanta hipótese de causa raiz com nível de confiança e evidências ligadas | Só começa a partir de um incidente declarado |
| Dynatrace SRE Agent | Em preview segundo a documentação | Analisa um problema depois da análise de causa raiz do Dynatrace e escreve recomendações | Confirme o que seu tenant já pode usar |
| HolmesGPT | Open source; sandbox da CNCF desde outubro de 2025 | Agente que consulta Kubernetes, métricas e logs para achar causa raiz; tem modo que roda em segundo plano | Pode executar consultas que você não previu: limite as permissões |
| K8sGPT | Open source; sandbox da CNCF desde dezembro de 2023 | Varre o cluster com regras fixas e, opcionalmente, usa um LLM para explicar os achados | Confira o que a anonimização cobre antes de enviar dados a um modelo externo |
| kagent | Open source; sandbox da CNCF desde maio de 2025 | Executa agentes de IA como recursos do Kubernetes | Mais pesado de operar; é um framework, não um agente pronto |
| kubectl-ai | Open source, do Google | CLI que traduz linguagem natural em comandos kubectl e pede confirmação antes de alterar | Assistente de terminal, não um agente de plantão |

Como avaliar qualquer um deles, na ordem recomendada:

1. Rode em modo somente leitura e compare as conclusões com as de quem atendeu o incidente.
2. Exija evidência em cada conclusão: query, janela e serviço.
3. Garanta auditoria do que o agente consultou e executou.
4. Só então libere mitigação, com allowlist, kill switch e teste com o agente desligado.

## Operação de clusters Kubernetes

Status de maturidade na CNCF em 3 de novembro de 2026.

| Ferramenta | Para que serve | Status e cautelas |
| --- | --- | --- |
| Karpenter | Provisiona e remove nós conforme os pods que não conseguem ser agendados, com consolidação para reduzir custo | Open source (Apache 2.0); criado pela AWS, com o núcleo neutro doado via Kubernetes SIG Autoscaling; v1.0 em 2024. Cada provedor tem sua implementação |
| KEDA | Autoscaling por evento, como filas e streams | Graduado. Defina teto para não repassar a carga a uma dependência saturada |
| Karmada | Orquestrar aplicações em vários clusters | Graduado em setembro de 2026 |
| Cluster API | Criar e gerenciar o ciclo de vida de clusters de forma declarativa | Projeto da comunidade Kubernetes |
| Headlamp | Interface web extensível para vários clusters | Sandbox |
| Velero | Backup e migração de recursos e volumes | Entrou na CNCF em março de 2026 (sandbox). Teste a restauração, não só o backup |
| Kyverno | Políticas como código na admissão: requests, probes, limites | Graduado em março de 2026 |
| Falco | Detecção de ameaças em tempo de execução | Graduado |
| Cilium e Hubble | Rede com eBPF e visibilidade do tráfego entre serviços | Cilium graduado; Hubble faz parte do Cilium |
| Istio | Service mesh: mTLS, controle de tráfego e telemetria | Graduado. Mesh tem custo de operação: adote com um motivo, não por padrão |
| OpenCost e Kubecost | Custo por namespace, workload e rótulo | OpenCost é incubating; Kubecost é a oferta comercial da IBM. O OpenCost tem servidor MCP para agentes consultarem custo |
| OptScale | FinOps: orçamento, anomalias de custo e otimização de recursos, incluindo Kubernetes e GPU | Emergente. Compare com OpenCost antes de somar outra ferramenta de custo |
| Keptn | Automação do ciclo de vida de aplicações em Kubernetes | Arquivado pela CNCF em setembro de 2025: não escolha para um projeto novo |

## Infraestrutura como código e segredos

| Ferramenta | Para que serve | Status e cautelas |
| --- | --- | --- |
| OpenTofu | Alternativa open source (MPL-2.0) ao Terraform, na Linux Foundation | Versão 1.13 em outubro de 2026. Pensado como substituto direto: teste seus módulos antes de migrar |
| Terraform | Infraestrutura como código muito usada | Licença BUSL desde 2023, que não é open source: confira os termos de uso |
| Crossplane | Provisionar infraestrutura pela API do Kubernetes | Graduado em outubro de 2025 |
| OpenBao | Fork open source (MPL-2.0) do Vault, no OpenSSF | Versão 2.7 em outubro de 2026; projeto em estágio sandbox |
| Vault | Gestão de segredos, chaves e certificados | Licença BUSL: confira os termos |
| External Secrets | Sincroniza segredos externos para dentro do cluster | Sandbox na CNCF |

## Operar sistemas de IA

Quando o serviço que você opera usa modelos, entram componentes novos no caminho crítico. A coluna “Status” é a classificação do guia, não um selo externo.

| Ferramenta ou técnica | Para que serve | Status | Cautela |
| --- | --- | --- | --- |
| Langfuse | Traces, avaliações e gestão de prompts de aplicações com LLM | Emergente | Hospedar a plataforma tem custo operacional |
| LiteLLM | Gateway de modelos: integração com vários provedores, controle de custo e de acesso | Emergente | É um novo ponto único no caminho: precisa de SLO, failover e limites próprios |
| DeepEval | Testes de comportamento de aplicações com LLM | Emergente | Sem dataset de avaliação versionado, o resultado não é comparável |
| Execução isolada (sandbox) para agentes de código | Restringir o que o agente consegue alcançar | Emergente | Combine com privilégio mínimo e credenciais temporárias |
| MITRE ATLAS | Modelo de ameaças específico para sistemas de IA | Emergente | É um catálogo de ameaças, não uma ferramenta de defesa |
| Execução durável (Temporal, Restate) | Retomar workflows de agentes depois de falhas e esperar aprovação humana | Emergente, mas ignorar o tema já custa caro | Comece pelo suporte do próprio framework do agente |

## Critérios para desempatar a escolha

| Critério | Pergunta objetiva | Sinal ruim |
| --- | --- | --- |
| Cardinalidade | Qual o teto de série ativa e o preço da série excedente? | “Ilimitado” sem sampling |
| Retenção | Métrica 15 meses, trace 7–30 dias, log quente 7–14 e frio 90? | Um único tier para tudo |
| Exportação | Consigo sair com OTLP ou com bloco Prometheus? | Só UI e API de leitura lenta |
| RBAC e auditoria | Quem viu PII no trace e quem mudou a regra de alerta? | Admin compartilhado |
| Residência | O span sai da região exigida por contrato? | Collector sem processador de atributo |
| Página | O alerta de SLO vive no mesmo produto que o plantão? | Webhook frágil entre três ferramentas |
| IA | A sugestão cita a evidência? | Texto sem query, sem janela, sem serviço |

## Combinações de referência por cenário

| Cenário | Stack coerente |
| --- | --- |
| Startup | OTel SDK, Prometheus, Grafana OSS, Alertmanager, plantão no PagerDuty ou Grafana IRM, status manual |
| Scale-up cloud-native | OTel Collector, Mimir ou SigNoz, Loki, Tempo, SLO burn rate (Pyrra ou Sloth), incident.io ou PagerDuty, Argo CD com Argo Rollouts ou Flagger para canary, Devtron se quiser uma camada única sobre CI/CD e GitOps |
| Enterprise | OTel na borda para não prender o vendor, SaaS ou SigNoz/Mimir conforme residência, ITSM só para follow-up, policy de error budget escrita |
| Regulado | O mesmo, mais retenção com trilha, amostragem sem perder o erro, atributo de PII removido no collector, chaos em produção só com change aprovado |

As combinações são exemplos coerentes, não receitas únicas. Veja o contexto de adoção no [caminho de adoção](05-adocao-caminho-cenarios.md) e as práticas que essas ferramentas implementam em [Práticas essenciais](02-catalogo-praticas.md).

---

[← Anterior: Caminho de adoção](05-adocao-caminho-cenarios.md) · [Índice do guia](../README.md#mapa-do-guia) · [Próxima: Referências →](07-referencias.md)
