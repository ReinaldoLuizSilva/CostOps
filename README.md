# CostOps

Inventário e análise de custos multi-cloud (Azure, GCP e OCI) com coleta automatizada, relatórios agendados e alertas de anomalia.

> 🚧 **Projeto em planejamento.** Desenvolvimento previsto para começar após outubro de 2026.

## Problema

Ambientes multi-cloud e multi-cliente dificultam responder perguntas simples: quais recursos existem, quanto cada um custa, o que cresceu de um mês para o outro e o que está ocioso. Cada provedor tem seu próprio console e formato de faturamento, e a consolidação costuma ser manual.

O CostOps automatiza essa coleta e centraliza o resultado.

## Ideia

- Coletar inventário de recursos e dados de custo dos três provedores via API/SDK
- Normalizar em um modelo comum (provedor, conta, serviço, recurso, tags, custo, período)
- Persistir o histórico para permitir comparação entre períodos
- Gerar relatório consolidado e alertar quando houver variação fora do esperado
- Rodar de forma agendada e automatizada, sem intervenção manual

## Stack pretendida

| Camada | Tecnologia |
|---|---|
| Coleta | Python (SDKs oficiais Azure, GCP e OCI) |
| Fontes | Azure Resource Graph + Cost Management · GCP Asset Inventory + Billing · OCI Usage API |
| Persistência | a definir (Oracle ADB Always Free ou armazenamento de objetos) |
| Infraestrutura | Terraform |
| Automação | GitHub Actions (execução agendada) |
| Relatório | a definir (Markdown/CSV, dashboard ou notificação) |

## Roadmap (rascunho)

- [ ] **M0** — Definir escopo, modelo de dados comum e estratégia de autenticação
- [ ] **M1** — Coletor de um provedor rodando localmente
- [ ] **M2** — Coletores dos demais provedores e normalização
- [ ] **M3** — Persistência e histórico
- [ ] **M4** — Relatório consolidado
- [ ] **M5** — Execução agendada e infraestrutura em Terraform
- [ ] **M6** — Alertas de anomalia de custo
- [ ] **M7** — *(possível)* Execução como CronJob em Kubernetes

## Decisões técnicas

*(a preencher a cada milestone)*

## Autor

Reinaldo Luiz da Silva — [LinkedIn](https://www.linkedin.com/in/reinaldoluizdasilva) · [GitHub](https://github.com/ReinaldoLuizSilva)