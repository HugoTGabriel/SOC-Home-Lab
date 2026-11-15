# 🕵️ Relatório de Caso: Scoping & Prontidão em Windows IR (Cenário OpenDoor LTD)

**Data:** 15 de Novembro de 2026  
**Autor:** Hugo Gabriel Cunha Alves  
**Função:** Analista de Suporte de TI / Foco em SOC N1  
**Plataforma / Sala:** TryHackMe — *Introduction to Windows IR* (*Path: Windows Incident Response & Forensics*)  
**Escopo do Exercício:** Triagem inicial, entrevista de scoping e planejamento de coleta forense (cenário interativo/briefing — execução prática das ferramentas prevista nas salas seguintes do módulo).

---

## 🎯 1. Objetivo e Contexto do Incidente
Documentar os primeiros passos de resposta a incidentes (IR) ao assumir uma investigação externa na empresa fictícia **OpenDoor LTD**, um ambiente corporativo Windows com **Active Directory** (~2.500 usuários / ~2.000 endpoints) apresentando anomalias e picos de tráfego suspeitos, mas **sem alertas registrados na solução de segurança de endpoint**.

---

## 🔍 2. Descobertas de Triagem e Postura (Scoping)
Durante o levantamento técnico inicial com a equipe interna de TI, duas falhas críticas explicaram a falta de visibilidade e guiaram a contenção inicial:

1. **EDR Coverage Gap (Falha de Cobertura de Agentes):**
   - A licença da solução principal citada pela equipe (`BitDefender`) estava **expirada**, deixando a maior parte do parque sem proteção/telemetria ativa.
   - Havia apenas uma implementação piloto do **TrendMicro** (EDR/EPP) rodando em **15 hosts** (14 estações não críticas e o controlador de domínio `DC-06`), o que explicava a ausência de alertas durante a movimentação do adversário nos demais endpoints.

2. **Comunicação Out-of-Band — OOB (NIST SP 800-61):**
   - Diante da suspeita de comprometimento do Active Directory e possível uso de credenciais internas vazadas, **canais corporativos integrados ao AD/tenant** (como **Microsoft Teams** e e-mail interno) foram evitados para discussão da resposta.
   - O motivo operacional é impedir que o invasor acompanhe em tempo real o plano de contenção dentro do próprio ambiente comprometido, adotando comunicação externa isolada (*Out-of-Band*) com registro de auditoria.

---

## 🧠 3. Fundamentos Forenses Mapeados (Pré-Coleta)

### A. Ordem de Volatilidade (RFC 3227 / ISO/IEC 27037)
Antes de desligar ou reiniciar qualquer endpoint suspeito, a coleta deve seguir do dado mais efêmero para o mais persistente:

1. **Registradores e Cache da CPU** *(camada mais volátil)*
2. **Estado em Memória e Rede:** tabelas de roteamento, cache ARP, tabela de processos, conexões ativas e **Memória RAM**
3. **Arquivos Temporários de Sistema** (`temp` / `tmpfs`)
4. **Armazenamento Persistente (Disco/NTFS):** registros `.evtx`, hives do Registro, `$MFT`, *Prefetch*, *Amcache*
5. **Logs Remotos (SIEM/Syslog), Topologia Física e Backups**

> **Regra prática:** Desligar um host na pressa apaga processos ativos em memória, sockets de rede e eventuais credenciais/artefatos *fileless* antes da captura.

### B. Papel das Ferramentas Mapeadas na Sala
| Categoria | Ferramentas Citadas | Função no Fluxo de DFIR |
| :--- | :--- | :--- |
| **Aquisição de Memória RAM** | `DumpIt`, `winpmem` | Realizam o **dump (captura)** da memória RAM física para preservação e análise posterior (ex.: via Volatility). Não analisam sozinhas — preservam o estado vivo. |
| **Coleta Direcionada de Disco (*Triage*)** | `KAPE`, `CyLR` | Extraem artefatos críticos do sistema de arquivos NTFS (`.evtx`, `SAM`/`SYSTEM`/`SOFTWARE`/`NTUSER.DAT`, `$MFT`, *Prefetch*, *Amcache*) com agilidade, sem precisar clonar discos inteiros logo na triagem inicial. |
| **Correlação Centralizada** | **SIEM** | Consolida e correlaciona os logs coletados de múltiplos servidores e estações para reconstruir a linha do tempo (*timeline*). |

---

## 🏁 4. Lições Operacionais & Próximos Passos
- **Nunca confundir ausência de alertas com ambiente limpo:** validar primeiro status de licenciamento, saúde do serviço e percentual real de cobertura (*Deployment Coverage*) do EDR.
- **Assumir comprometimento do canal interno em incidentes de AD:** toda coordenação de crise começa fora do diretório afetado (*Out-of-Band*).
- **Próximo passo prático na trilha:** avançar para as salas operacionais do módulo (*Accessing a Compromised Network* e *Key Artifacts for DFIR*) para praticar em VM a coleta com ferramentas de triagem e análise dos artefatos extraídos.

---
*Relatório documentado como parte da trilha contínua de estudos em DFIR, Windows IR e SOC N1.*
