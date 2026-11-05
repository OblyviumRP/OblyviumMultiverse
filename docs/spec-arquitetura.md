# Spec de arquitetura: OblyviumMultiverse

Proposta, sem código. Base: `oblyvium` (`3854b43`), Multiverse-Core 5.8.1 mais 3 commits, lida em 2026-11-05. Cada fase vira um PR separado.

## Objetivo

Deixar o fork rastreável e sem risco de thread. Nosso código são 3 commits em 2 arquivos, 45 linhas, sobre 40 mil de terceiro. Refatorar aqui só encarece a próxima versão.

Fora de escopo: refatorar o Multiverse-Core.

## Estado medido

- Mudanças nossas: `AsyncSafetyTeleporterAction` e `BlockSafety`, para `teleportAsync` e checagem de spawn na thread da região.
- O jar do Release informa versão `local`, sem como ligar a um commit. Sem release nossa.
- Nem o `plugin.yml` do repo nem o do jar instalado declaram `folia-supported`. O `shreddedpaper.yml` tem `run-unsupported-plugins-in-sync: true`, então ele roda em modo de compatibilidade sync. Não medi o efeito no tick.
- 7 workflows com `ubuntu-latest`, herdados, contra a regra do runner. 7 branches extras herdadas. Branch padrão `oblyvium`, fora da regra de `main`.

## SOLID

| Princípio | Onde | Mudança |
|---|---|---|
| S, O, L, I, D | Código do upstream | Não aplicar. |
| D | Mudança nossa em `BlockSafety` | Se crescer, a checagem por região sai para uma classe nossa, chamada de um ponto só. Reduz o conflito no merge. |

## Padrões pedidos

Chain of Responsibility, Método da fábrica e Decorador não têm caso **nosso** com 45 linhas. No upstream, o caminho de teleporte já tem etapas e ações próprias.

Se o patch de thread crescer, o certo é **Decorador**: embrulhar a ação de teleporte seguro com a verificação de região, em vez de editar os métodos dela. Fica registrado para quando houver a próxima correção de thread, sem antecipar.

## Fases

1. Versão rastreável: `5.8.1-oblyvium.N` no build e tag por entrega. Aceite: jar diz a versão e a tag aponta para o commit.
2. Workflow próprio em `runs-on: [self-hosted, ptero]`, desligando os herdados. Aceite: jar construído pelo CI.
3. Definir `folia-supported`. Testar com `true` no servidor de teste, com teleporte e troca de mundo com bots. Aceite: decisão registrada, sem erro de thread no log.
4. Decidir autoupdate com o dev. A regra pede perguntar antes.
5. Limpar branches herdadas e registrar a exceção de branch, ou passar para `main`. Aceite: catálogo e GitHub dizem o mesmo.
6. Procedimento de atualização: como trazer 5.9 do upstream. Aceite: o README descreve e foi usado uma vez.

## Riscos

- **Thread:** com `folia-supported: true`, qualquer caminho sem correção vira erro em jogo. Por isso a fase 3 é só no servidor de teste.
- **Mundos:** Multiverse carrega mundos de jogadores. Teste nunca no Release.
- **Upstream:** cada linha nossa no herdado custa no merge.

## Não fazer

- Refatorar, migrar para `.mb` ou renomear pacote.
- Decorador antes de haver segunda correção de thread.
