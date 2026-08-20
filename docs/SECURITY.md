# Política de segurança experimental

O objetivo é estudar hardening sem confundir segurança com alteração arbitrária de comportamento.

## Regras

- Não remover validações para obter desempenho.
- Toda alteração em parsing, protocolo, autenticação ou privilégios exige teste negativo.
- Preferir mudanças pequenas e auditáveis.
- Tratar crashes, corrupção de memória e comportamento indefinido como falhas críticas do experimento.
- Nunca considerar um sanitizer limpo como prova de ausência de vulnerabilidades.

## Builds instrumentados sugeridos

Quando suportado pelo ambiente de compilação, manter builds separados para:

- AddressSanitizer (ASan);
- UndefinedBehaviorSanitizer (UBSan);
- stack protector e hardening do compilador;
- símbolos de debug para análise com debugger/profiler.

## Registro

Cada achado deve conter:

- componente afetado;
- forma de reprodução;
- impacto observado;
- alteração proposta;
- testes adicionados;
- resultado após correção.
