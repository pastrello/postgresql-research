# SEC-002 — Allowlist opcional de bibliotecas dinâmicas

Status: `research`
Prioridade: P2 hardening
Risco esperado: médio

## Comportamento atual

`load_external_function()` resolve nomes usando `dynamic_library_path` e carrega a biblioteca após verificar o bloco de compatibilidade/ABI.

`load_file(..., restricted=true)` possui regra adicional: somente nomes sob `$libdir/plugins/` sem separadores adicionais são aceitos.

Isso reflete o modelo de privilégios atual do PostgreSQL e não é uma vulnerabilidade identificada nesta auditoria.

## Objetivo experimental

Para instalações muito fechadas, oferecer uma política opcional que limite bibliotecas carregáveis a diretórios e/ou nomes explicitamente autorizados, mesmo em caminhos que hoje são permitidos a atores privilegiados.

## Possível design

- GUC de startup para diretórios permitidos;
- default vazio/desabilitado = comportamento upstream;
- canonicalizar caminho antes da comparação;
- nunca permitir caminho relativo;
- preservar `$libdir` e o mecanismo de plugins trusted/restricted;
- logar negação sem executar `dlopen()`;
- considerar modo de auditoria (`log only`) antes de modo enforcement.

## Riscos

- extensões fora de `$libdir` podem deixar de funcionar quando enforcement estiver ativo;
- symlinks e mounts exigem política clara sobre caminho lógico versus caminho real;
- não deve ser apresentado como substituto para permissões de filesystem, SELinux/AppArmor ou controle de superuser.
